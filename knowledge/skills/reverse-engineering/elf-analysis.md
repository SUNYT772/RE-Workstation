# ELF 二进制深度分析参考

来源：skills/reverse-engineering/elf-analysis.md

本文保留原资料的技术主题与判断依据。历史名称和版本信息用于解释概念，不规定工具依赖；下文只描述知识、输入输出及可观察证据。


> 逆向 Linux/Android ELF 文件时的结构解析、反分析对抗识别和分析技巧。

---

## ELF 结构速查

### 文件头 (ELF Header)

```text
偏移  大小  字段              说明
0x00  4    e_ident[EI_MAG]   Magic: 7f 45 4c 46 ("\x7fELF")
0x04  1    e_ident[EI_CLASS] 1=32bit, 2=64bit
0x05  1    e_ident[EI_DATA]  1=LE, 2=BE
0x10  2    e_type            2=EXEC, 3=DYN(PIE/SO), 4=CORE
0x12  2    e_machine         0x03=x86, 0x3E=x86_64, 0xB7=AArch64, 0x28=ARM
0x18  8    e_entry           入口点虚拟地址（本表后半为 ELF64 布局）
0x20  8    e_phoff           程序头表偏移
0x28  8    e_shoff           节头表偏移（strip 后可能为 0）
0x38  2    e_phnum           程序头数量
0x3C  2    e_shnum           节头数量
```
### 程序头 (Program Header)

```text
类型值  名称       说明
0x01   PT_LOAD    可加载段（代码/数据）
0x02   PT_DYNAMIC 动态链接信息
0x03   PT_INTERP  解释器路径（/lib/ld-linux.so）
0x04   PT_NOTE    辅助信息
0x06   PT_PHDR    程序头表自身
0x6474e550 PT_GNU_EH_FRAME  异常处理
0x6474e551 PT_GNU_STACK     栈可执行标记
0x6474e552 PT_GNU_RELRO     只读重定位
```
### 常见节 (Sections)

| 节名 | 说明 |
|------|------|
| `.text` | 代码段 |
| `.rodata` | 只读数据（字符串常量） |
| `.data` | 已初始化全局变量 |
| `.bss` | 未初始化全局变量 |
| `.plt` / `.got` | 动态链接跳转表 |
| `.init_array` | 构造函数指针数组 |
| `.fini_array` | 析构函数指针数组 |
| `.dynamic` | 动态链接信息 |
| `.symtab` / `.dynsym` | 符号表 |
| `.strtab` / `.dynstr` | 字符串表 |

---

## 反分析手法识别

### 常见 ELF 反分析技术

| 手法 | 特征 | 对抗方式 |
|------|------|---------|
| 损坏程序头 | PHDR 填充垃圾数据（如 0x0a） | 比较真实加载段与损坏 PHDR，记录修复依据 |
| 无 section header | `e_shoff = 0`, `e_shnum = 0` | 只依赖程序头分析，不依赖 section |
| 去符号 (strip) | 无 `.symtab`，函数名全丢 | GoReSym(Go) / 签名匹配 / FLIRT |
| 静态链接 | 无 `.dynamic`，体积巨大 | 用 FLIRT/Lumina 识别库函数 |
| 伪装文件类型 | 后缀 .sh/.txt/.jpg | 以 magic bytes 与头字段判断 |
| UPX 加壳 | 包含 `UPX!` 标记 | UPX 解压后布局与原始入口 |
| 自定义壳 | 入口点跳转到解压代码 | OEP 附近的解压后内存状态 |
| 反调试 | ptrace(TRACEME) | 跟踪检测返回值及依赖分支 |
| 反虚拟机 | 检查 /proc/cpuinfo | 核对 cpuinfo 读取内容与判断条件 |
| 代码加密 | 运行时解密 .text | 解密后 .text 与原文件字节对照 |

### 识别自解压/自修改代码

入口附近出现可写映射、数据复制或解码、权限变化以及跳转到新区域，构成自解压或自修改线索。观察项包括映射返回地址、大小、权限、写入来源、mprotect 前后状态、最终跳转地址，以及解密后内存与原文件的差异。解密后区域是新的分析对象，但原始映射和解码关系仍需保留。
---

## ARM64 (AArch64) 逆向速查

### 寄存器

| 寄存器 | 用途 |
|--------|------|
| x0-x7 | 参数/返回值 |
| x8 | 间接结果（syscall 号） |
| x9-x15 | 临时寄存器 |
| x16-x17 | IP0/IP1（PLT 跳转） |
| x18 | 平台寄存器（Android: shadow call stack） |
| x19-x28 | 被调用者保存 |
| x29 (FP) | 帧指针 |
| x30 (LR) | 链接寄存器（返回地址） |
| SP | 栈指针 |
| PC | 程序计数器 |

### 常见指令模式

```text
函数序言：
  stp x29, x30, [sp, #-N]!    # 保存 FP 和 LR
  mov x29, sp                  # 设置帧指针

函数尾声：
  ldp x29, x30, [sp], #N      # 恢复 FP 和 LR
  ret                          # 返回（br x30）

系统调用：
  mov x8, #NR                  # syscall 号
  svc #0                       # 触发 syscall

条件分支：
  cmp x0, #0
  b.eq label                   # 等于跳转
  b.ne label                   # 不等于跳转
  cbz x0, label                # x0 == 0 跳转
  cbnz x0, label               # x0 != 0 跳转

地址加载：
  adrp x0, page                # 加载页地址高位
  add x0, x0, #offset          # 加低 12 位偏移
  ldr x0, [x1, #offset]        # 从内存加载
```
### Linux ARM64 系统调用号

| 号码 | 名称 | 说明 |
|------|------|------|
| 56 | openat | 打开文件 |
| 63 | read | 读取 |
| 64 | write | 写入 |
| 57 | close | 关闭 |
| 222 | mmap | 内存映射 |
| 226 | mprotect | 修改内存权限 |
| 117 | ptrace | 进程跟踪 |
| 220 | clone | 创建进程/线程 |
| 221 | execve | 执行程序 |
| 93 | exit | 退出 |
| 94 | exit_group | 退出进程组 |

---

## 常见压缩/打包算法识别

| 算法 | 识别特征 | 解压方式 |
|------|---------|---------|
| **LZSS** | 位流 + 字面量/匹配标记 | 自定义解压器（如本报告） |
| **ZLIB/Deflate** | Magic: `78 01`/`78 9C`/`78 DA` | ZLIB 数据流 |
| **GZIP** | Magic: `1F 8B` | GZIP 数据流 |
| **LZ4** | Magic: `04 22 4D 18` | LZ4 帧 |
| **LZMA/XZ** | Magic: `FD 37 7A 58 5A 00` (XZ) | XZ 容器与 LZMA 编码 |
| **Brotli** | 无固定 magic，看上下文 | Brotli 码流 |
| **Zstandard** | Magic: `28 B5 2F FD` | Zstandard 帧 |
| **UPX** | 字符串 `UPX!` | UPX 格式 |
| **自定义** | 入口点有解压循环 | 恢复码流语义与输出缓冲区关系 |

### 识别自定义压缩的线索

```text
1. 入口点附近有循环 + 位操作（移位、AND、OR）
2. 有"滑动窗口"回拷（从输出缓冲区往回读）→ LZ 系列
3. 有频率表/霍夫曼树构建 → Deflate/Huffman
4. 有固定大小块处理 → 块压缩（LZ4/Snappy）
5. 有算术编码特征（区间缩小）→ LZMA/ANS
```
---

## Linux 进程注入与自修改的行为识别

### 映射、权限与代码转移

行为模型是匿名私有映射接收代码字节，随后权限变为可执行，控制流最终进入该区域。证据包括保存的映射返回值、连续复制或循环写入、权限变化和 br/blr/jmp 的目标来源。它描述已有样本的加载机制，不提供注入调用序列或载荷。
### ptrace 相关目标进程变化

ptrace 相关注入的已有行为线索是进程附着与等待、目标寄存器读取和改写、目标地址空间写入、PC 改变与恢复执行。/proc/<pid>/mem 访问也可能参与。记录目标进程身份、写入区域和寄存器变化，能够区分调试行为、修改行为与实际控制流转移。
### /proc/self/mem 自修改

对 /proc/self/mem 的读写配合文件偏移，可表现为对自身地址空间的修改。源文用于解释只读 .text 或 W^X 语境下的运行时指令变化；其有效条件依内核和权限而异。关键证据是文件句柄、偏移对应的虚拟地址、修改前后字节和实际执行路径，不保留写入命令。
---

## 分析大型 ELF 的策略

对于 5MB+ 的大型二进制：

大型文件（原资料以 5MB 以上为例）的材料可按四个维度组织：

- 初始画像：架构、类型、保护属性，error/fail/http、/proc、/dev 字符串，导入和导出。
- 加载结构：PT_LOAD 布局、入口附近解压或解密、.init_array 构造函数与反调试。
- 功能锚点：字符串交叉引用，mmap/ptrace/open 以及 connect/send/recv 的调用关系。
- 模块差异：自解压前后区域、多模块功能边界、不同版本之间的结构和行为变化。

这些材料的组合由当前问题决定，不保留原文 5 分钟/10 分钟的固定阶段。
---

## 文件与运行时输出的核对项

需要相互核对的输出有 ELF 头、程序头、存在时的节头、数据区与全文件字符串、架构匹配的反汇编、系统调用与库调用轨迹、运行时内存区域。程序头修复的原例修改 ELF64 的 e_phnum（偏移 0x38，2 字节小端值）；正确数量应由真实程序头范围和加载关系支持，不能把例中的 2 当作通用答案。
