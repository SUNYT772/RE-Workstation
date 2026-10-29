# 逆向工程参考资源汇总

来源：skills/reverse-engineering/awesome-re-resources.md

本文保留原资料的技术主题与判断依据。历史名称和版本信息用于解释概念，不规定工具依赖；下文只描述知识、输入输出及可观察证据。


> 原资源目录的知识分工：格式规范、架构语义、运行时、行为证据、反混淆与学习材料。项目规模和时间区间保留为原资料背景，未联网核验，不构成依赖或访问要求。

---

## 综合资源库

| 项目 | 覆盖 | 历史标识或知识范围 |
| ------ | ------ | ------ |
| **awesome-reversing** (tylerha97) | 逆向工具/书籍/课程/练习 | `github.com/tylerha97/awesome-reversing` |
| **awesome-reverse-engineering** (alphaSeclab) | 3500+ 工具 + 2300 文章，全平台 | `github.com/alphaSeclab/awesome-reverse-engineering` |
| **Reverse-Engineering** (mytechnotalent) | 免费教程：x86/x64/ARM/AVR/RISC-V | `github.com/mytechnotalent/Reverse-Engineering` |
| **awesome-malware-analysis** (rshipp) | 恶意软件分析工具/资源 | `github.com/rshipp/awesome-malware-analysis` |
| **reversingBits** | 逆向/二进制分析速查表合集 | `github.com/mohitmishra786/reversingBits` |
| **awesome-arm-exploitation** | ARM 利用资源（视频/文章/书籍） | `github.com/HenryHoggard/awesome-arm-exploitation` |
| **Binary-Analysis-Automation** | 自动化二进制分析（ML/脚本/静态/动态） | `github.com/user1342/Awesome-Binary-Analysis-Automation` |

---

## ELF / Linux 逆向专项

| 资源 | 说明 | 历史标识或知识范围 |
|------|------|------|
| **libelfmaster** | 安全 ELF 解析库（取证/恶意软件重建） | `github.com/elfmaster/libelfmaster` |
| **ELF 规范** | 官方 ELF 格式文档 | `refspecs.linuxfoundation.org/elf/elf.pdf` |
| **Linux Internals** | /proc 文件系统、内存布局、syscall | `0xax.gitbooks.io/linux-insides/` |
| **Compiler Explorer** | 在线看 C/C++/Rust/Go 编译成什么汇编 | `godbolt.org/` |

---

## ARM / AArch64 专项

| 资源 | 说明 | 历史标识或知识范围 |
|------|------|------|
| **ARM 官方架构手册** | 完整指令集参考 | `developer.arm.com/documentation` |
| **Azeria Labs** | ARM 汇编/利用教程（最佳入门） | `azeria-labs.com/writing-arm-assembly-part-1/` |
| **ARM64 syscall 表** | Linux AArch64 系统调用号 | `arm64.syscall.sh/` |
| **QEMU 用户态模拟** | 不需要真实设备分析 ARM 二进制 | 跨架构系统调用语义与轨迹 |

---

## 恶意软件分析

| 资源 | 说明 | 历史标识或知识范围 |
|------|------|------|
| **YARA** | 恶意软件特征匹配规则 | `github.com/VirusTotal/yara` |
| **Volatility 3** | 内存取证框架 | `github.com/volatilityfoundation/volatility3` |
| **FLOSS** | 自动提取混淆字符串 | `github.com/mandiant/flare-floss` |
| **Detect It Easy (DiE)** | 文件类型/壳/编译器识别 | `github.com/horsicq/Detect-It-Easy` |
| **PE-bear** | PE 文件分析器 | `github.com/hasherezade/pe-bear` |
| **Capa** | 自动识别二进制能力（网络/文件/加密等） | `github.com/mandiant/capa` |
| **Unpacker** | 通用脱壳框架 | `github.com/malwaretech/UnpackerFramework` |

---

## 动态分析 / 沙箱

| 资源 | 说明 | 历史标识或知识范围 |
|------|------|------|
| **Frida** | 跨平台动态插桩 | `frida.re/` |
| **strace** | Linux 系统调用跟踪 | 系统自带 |
| **ltrace** | 库函数调用跟踪 | 系统自带 |
| **QEMU** | 用户态/系统态模拟 | `www.qemu.org/` |
| **Unicorn** | CPU 模拟框架（可编程） | `www.unicorn-engine.org/` |
| **Qiling** | 高级二进制模拟框架 | `qiling.io/` |
| **angr** | 符号执行 + 二进制分析 | `angr.io/` |
| **Triton** | 动态二进制分析框架 | `triton-library.github.io/` |

---

## 反混淆 / 脱壳

| 资源 | 说明 | 历史标识或知识范围 |
|------|------|------|
| **UPX** | 常见压缩壳，解压前后布局对照 | `upx.github.io/` |
| **unipacker** | 通用 PE 脱壳器 | `github.com/unipacker/unipacker` |
| **de4dot** | .NET 反混淆 | `github.com/de4dot/de4dot` |
| **JADX** | Android DEX 反混淆 | `github.com/skylot/jadx` |
| **JEB** | 商业 Android/ARM 反编译器 | `www.pnfsoftware.com/` |
| **Miasm** | 逆向工程框架（IR/符号执行/反混淆） | `github.com/cea-sec/miasm` |
| **OLLVM 反混淆** | 控制流平坦化/虚假控制流对抗 | 以路径条件与位向量语义恢复 |

---

## 历史在线平台的证据类型

| 平台 | 说明 | 历史标识或知识范围 |
|------|------|------|
| **VirusTotal** | 多引擎扫描 + 行为分析 | `www.virustotal.com/` |
| **Joe Sandbox** | 自动化恶意软件分析 | `www.joesandbox.com/` |
| **ANY.RUN** | 交互式在线沙箱 | `any.run/` |
| **Hybrid Analysis** | 免费恶意软件分析 | `www.hybrid-analysis.com/` |
| **Compiler Explorer** | 看编译器输出 | `godbolt.org/` |
| **Dogbolt** | 多反编译器对比（IDA/Ghidra/Binary Ninja） | `dogbolt.org/` |

---

## 学习主题的层次

### 入门主题（原资料建议 0–3 个月）

1. Reverse Engineering for Beginners（历史标识：`beginners.re/`） — 免费电子书
2. Azeria Labs ARM 教程（历史标识：`azeria-labs.com/`） — ARM 汇编基础
3. Nightmare（历史标识：`guyinatuxedo.github.io/`） — CTF 逆向/Pwn 教程
4. crackmes.one（历史标识：`crackmes.one/`） — 逆向练习题

### 进阶主题（原资料建议 3–12 个月）

1. Practical Binary Analysis（历史标识：`practicalbinaryanalysis.com/`） — 实战二进制分析
2. The IDA Pro Book（历史标识：`nostarch.com/idapro2.htm`） — IDA 深度使用
3. Malware Unicorn RE101（历史标识：`malwareunicorn.org/workshops/re101.html`） — 恶意软件逆向
4. pwnable.kr（历史标识：`pwnable.kr/`） / pwnable.tw（历史标识：`pwnable.tw/`） — Pwn 练习

### 高级

1. Modern Binary Exploitation（历史标识：`github.com/RPISEC/MBE`） — RPI 课程
2. How to Hack Like a Ghost（历史标识：`nostarch.com/how-hack-ghost`） — 高级渗透
3. Windows Internals（历史标识：`docs.microsoft.com/en-us/sysinternals/`） — Windows 内核
4. 真实样本的静态结构、行为语义与证据记录（MalwareBazaar 为原资料资源名）

---

## 速查表

| 速查表 | 历史标识或知识范围 |
|--------|------|
| x86/x64 指令速查 | `www.felixcloutier.com/x86/` |
| ARM64 指令速查 | `developer.arm.com/documentation/ddi0602/latest` |
| Linux syscall 表 (x64) | `blog.rchapman.org/posts/Linux_System_Call_Table_for_x86_64/` |
| Linux syscall 表 (ARM64) | `arm64.syscall.sh/` |
| GDB 速查 | `darkdust.net/files/GDB%20Cheat%20Sheet.pdf` |
| radare2 速查 | radare2 相关速查主题（其他模块，未建立链接） |
| IDA 快捷键 | `hex-rays.com/products/ida/support/freefiles/IDA_Pro_Shortcuts.pdf` |
| Ghidra 快捷键 | Ghidra 界面操作参考（历史资料名） |
