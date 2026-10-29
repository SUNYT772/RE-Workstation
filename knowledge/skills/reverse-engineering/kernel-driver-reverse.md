# 内核驱动逆向：分发、结构与行为证据

来源：skills/reverse-engineering/kernel-driver-reverse.md

## 范围与阅读线索

本页保留 Windows/Linux 内核驱动、Rootkit 行为、C/C++ 对象恢复、编译器特征与内核状态观察知识。原工具操作转为结构关系和观察目标，调试连接配置与命令不属于正文。

## Windows 驱动逆向

### 驱动类型

| 类型 | 特征 | 分析重心 |
|---|---|---|
| WDM | 传统 Windows Driver Model，手动管理 IRP | DriverEntry、设备创建、Dispatch 例程 |
| KMDF | Kernel Mode Driver Framework，事件驱动 | EvtDriverDeviceAdd、队列和 I/O 回调 |
| WDF | KMDF 与 UMDF 的统称 | WdfDriverCreate 与框架对象关系 |
| Minifilter | 文件系统过滤驱动 | FltRegisterFilter、Pre/Post 回调 |

### WDM 的入口与分发关系

DriverEntry 是入口证据之一；IoCreateDevice 和 IoCreateSymbolicLink 的参数能够揭示设备对象与符号链接，例如 `\Device\MyDriver` 与 `\DosDevices\MyDriver` 的关系。设备名、链接名、调用者和创建条件共同决定用户态接口的可达性。

`DriverObject->MajorFunction[IRP_MJ_DEVICE_CONTROL]` 对应设备控制分发例程，用户态 DeviceIoControl 请求在此进入驱动。处理函数常按 IoControlCode 分支，形成“请求号—缓冲区语义—具体功能”的映射。

源文列出的审查线索包括用户缓冲区长度未验证、METHOD_NEITHER 下直接使用用户指针、IOCTL 权限检查缺失。它们是待证实的风险线索：证据需包含请求的实际可达性、指针来源、检查位置和最终内存访问，不能仅由函数名断言漏洞。

### IOCTL 编码解析

CTL_CODE 包含 DeviceType、Access、Function、Method。纯字段分解伪代码如下：

```text
device_type = (code >> 16) AND 0xFFFF
access      = (code >> 14) AND 0x3
function    = (code >> 2)  AND 0xFFF
method      = code AND 0x3
method: 0=BUFFERED, 1=IN_DIRECT, 2=OUT_DIRECT, 3=NEITHER
access: 0=ANY, 1=READ, 2=WRITE, 3=READ|WRITE
```

原例 `0x80002034` 对应 DeviceType `0x8000`、Function `0x80D`、Method BUFFERED、Access ANY。方法字段决定缓冲区传递形式，访问字段描述编码中的权限要求；实际访问控制还需结合设备对象与调用路径。

### 结构识别与历史资源

Driver Buddy Reloaded 的历史能力是识别 IOCTL、Dispatch 与设备名；WinDbg/IDA 代表内核状态与静态结构对照；FLIRT/Lumina 代表 WDK 库函数识别。这些名称用于说明已有资料的能力分工，不规定依赖。

原文参考主题包括 VoidSec 的 Windows Drivers RE Methodology、Driver Reversing 101 中 WDM/KMDF 的差异，以及 Methodology of Reversing Vulnerable Killer Drivers 的漏洞驱动方法。

## Linux 内核模块逆向

### LKM 结构

`.ko` 常为 ELF relocatable，而非普通 executable。init_module/module_init 与 cleanup_module/module_exit 对应初始化和清理入口；`.modinfo` 可包含模块信息。关键操作表包括字符设备的 file_operations、网络设备的 net_device_ops 和块设备的 block_device_operations。

### 入口、操作表与请求分发

register_chrdev、cdev_add、misc_register 等注册调用可关联到操作表。表中的 open/read/write、unlocked_ioctl、compat_ioctl 指针连接用户接口与内核实现；ioctl 通常按 cmd 分支。重定位信息、符号、函数指针和注册调用的交叉引用比孤立字符串更能支持归属判断。

### Rootkit 行为与检测证据

| 类别 | 行为特征 | 观察依据 |
|---|---|---|
| 系统调用表改写 | sys_call_table 条目偏离预期 | 内存表与对应磁盘 vmlinux 的差异 |
| VFS 修改 | file_operations 指针变化 | 指针是否落在预期内核代码或已知模块区间 |
| Netfilter | nf_register_net_hook 与回调 | hook 链表、注册者和处理路径 |
| kprobe/ftrace | 探针或跟踪回调注册 | 注册列表、回调地址和所属模块 |
| eBPF | BPF 程序与映射 | 程序身份、附着点、指令和 map 读写 |
| DKOM | 内核对象或进程链表直接改变 | task_struct 链表与 /proc 等视图的不一致 |

修改 /proc、VFS 或网络过滤路径可能导致进程、文件、连接隐藏。行为结论需要多个视图对照；注册某类回调本身也可能是正常功能。

### 状态材料

内核转储、内存取证、内核日志、已加载模块列表、模块元信息与用户态系统调用轨迹分别提供崩溃上下文、对象状态、事件时间、驻留范围、声明信息和接口调用。原文中的 crash、volatility3、dmesg/journalctl、lsmod、modinfo、strace 是这些材料的历史名称。

## C/C++ 二进制模式

### C 模式

| 结构 | 常见指令表现 |
|---|---|
| if/else | cmp 与 jcc 条件跳转 |
| switch | 连续比较或按索引访问跳转表 |
| for | 边界比较、循环体、增量和回跳 |
| while / do-while | 条件分别偏向循环顶部或底部 |
| 函数指针 | call reg 或 call [reg+offset] |
| 结构体 | 同一基址加固定字段偏移 |
| 动态分配 | 分配返回指针随后被用于字段访问 |
| 字符串比较 | strcmp 或重复字符串比较指令 |

### C++ 模式

对象首部或子对象中可包含 vtable 指针，间接调用使用该表中某个偏移。构造函数常写入 vtable 并初始化字段；析构函数清理成员并可能关联 operator delete。this 在不同 ABI 下可能位于 rcx、rdi 或其他位置。

继承可以表现为虚函数覆盖，多重继承可有多个子对象 vtable 指针。RTTI 可揭示类名或层次；异常路径常涉及 __cxa_throw 或 _CxxThrowException。std::vector 常表现为 begin/end/capacity 三个边界指针；std::string 的 SSO 使短串内联而长串堆分配。

### vtable 与类层次恢复

连续函数指针数组、只读数据区引用、构造函数的指针写入是定位依据。共享条目、析构函数、RTTI 和对象偏移共同支持继承关系。源中“表前 -8 常见 RTTI、首项常见析构函数”属于特定 64 位 ABI 的经验，具体位置应由目标 ABI 和表的地址点确认。

对每个间接调用，知识记录可保留对象基址、子对象偏移、表地址、槽位偏移、候选目标和归属依据。仅给函数改名不足以证明类层次。

### 结构体恢复

- 固定偏移的读写宽度反映字段大小；如 `[rdi+0]` 的 32 位读、`[rdi+8]` 的字段访问，以及 `movss` 对偏移 `0x10` 的标量浮点读。
- 分配大小提供对象总体积，如 `0x30` 即 48 字节，但可能还包含头部、对齐或尾随数据。
- 构造函数的初始化顺序、字段默认值和后续用途帮助判断类型。
- 同一偏移可能被不同视图访问，宽度或语义冲突应留为证据，不能为整齐排版而消除。

## 编译器与优化线索

MSVC 可见 security cookie、Rich Header 与特定调用约定；GCC 常见 __stack_chk_fail、栈保护及 `.note.GNU-stack`；Clang/LLVM 可能有相似保护与不同优化形态，sanitizer 构建可出现 __asan_*；MinGW 同时具有 GCC 与 Windows API 线索；Android Clang 资料可涉及 __android_log_print 与 PGO。

低优化常留下栈变量和冗余移动；基本优化增加寄存器使用；较高优化可能内联、展开循环、尾调用或向量化。PGO 可把冷路径分离到 `.text.cold`，LTO 可跨模块内联并消除死代码。优化等级与单个特征不是一一对应关系。

## 内核状态观察语境

### Windows

原文的内核调试内容承载这些观察目标：崩溃原因和栈、已加载模块及地址范围、驱动对象、DRIVER_OBJECT 布局、指定函数处的寄存器与内存。网络或串口只是状态取得的历史通道；连接配置、端口与密钥不保留。

### Linux

观察目标包括 vmlinux 与运行内核符号的一致性、内核线程、init_task、内核日志，以及可重定位模块的加载地址。QEMU、GDB、kgdb 是源中的历史环境名称；本页不包含启动或连接命令。

## 证据组织与参考主题

报告可按驱动类型、初始化、设备命名、分发表、IOCTL 字段、缓冲区与检查、对象布局、系统修改、日志和转储证据排列。源参考还包括 Elastic Linux Rootkit 系列、LOLDrivers、Windows Driver Samples、Linux Kernel Module Programming 和 Trail of Bits 的 C++ 去虚拟化文章。文献名称保留为进一步查阅线索，不要求联网。
