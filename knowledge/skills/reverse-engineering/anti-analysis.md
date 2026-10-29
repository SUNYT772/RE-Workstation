# 反分析机制：检测、隐式控制流与证据

来源：skills/reverse-engineering/anti-analysis.md

## 范围

保留 Linux/Windows 反调试、反虚拟机、反插桩、完整性与反反汇编主题。原干预代码转为检测输入、状态依赖与可观察结果；历史名称只用于概念对应。

## Linux Anti-Debug：Linux 反调试

### ptrace-Based：自跟踪与双进程

PTRACE_TRACEME 的返回值常被用于判断是否已被跟踪；原例以 -1 进入退出路径。但失败还可能源于策略或权限，需要返回值、errno 与分支的共同证据。fork 子进程附着父进程并长期等待，可能形成监视关系，占用跟踪者位置；父子 PID、wait 状态与谁执行真实逻辑应分开记录。

原文涉及库函数观察、指令变化、系统调用层与内核策略四个层次，它们作用范围不同：libc 入口与直接 syscall 不相同，改变策略也不是修改算法。这里保留差异，不保留预加载、补丁、结束进程或内核配置命令。

### /proc Filesystem Checks：文件视图

/proc/self/status 中 TracerPid、/proc/self/exe 链接和 /proc/self/maps 中库名是不同证据源。TracerPid 非零通常表达跟踪关系；链接和映射名变化可被用作环境指纹。读取时点、命名空间、读到的字节与判断条件决定实际结果；伪造文件内容的原例不转为操作说明。

### Timing-Based Detection：计时

rdtsc、clock_gettime(CLOCK_MONOTONIC)、gettimeofday 可比较两个时点的差。阈值、单位、时间源、线程调度和额外观测开销影响结论；延迟不独属于调试器。timespec/tv_sec/tv_nsec 与 timeval 的单位也不能混同。固定时间或移除计时指令会改变环境，证据需区分原测量与干预后结果。

### Signal-Based Anti-Debug：信号

SIGTRAP/int3 是否进入程序处理器、SIGALRM 的超时，以及 SIGSEGV 故意异常中的真实计算可产生隐式控制流。程序收到信号、观察端截获信号和处理器完成工作是不同事件。原 alarm(5) 是案例时限；不能把异常一律当作失败或垃圾路径。

### Syscall-Level Evasion：直接系统调用

x86-64 Linux 原例 syscall 101 对应 ptrace。直接系统调用避开 libc 符号，因此仅观察库函数会漏掉检测。架构、调用号、参数寄存器、返回值和错误码共同构成系统调用事件；其他架构编号不同。

## Windows Anti-Debug：Windows 反调试

### PEB Checks：进程环境块

BeingDebugged 位于 PEB 偏移 0x2；原文列 NtGlobalFlag 的 32/64 位示例偏移 0x68/0xBC，掩码 0x70 对应 heap tail/free/parameter 检查。布局取决于位宽与系统版本。字段读取、位测试与被选目标之间的关系，比“存在某 API”更能说明行为。

### NtQueryInformationProcess

原文三类查询分别为 ProcessDebugPort=0x7、ProcessDebugObjectHandle=0x1E、ProcessDebugFlags=0x1F。DebugPort 非零、DebugObject 查询的状态码和输出句柄、DebugFlags 的反向语义（0 可表示调试）不能混为同一布尔值。返回 NTSTATUS 与输出缓冲区都属于证据。

### Heap Flags

原 64 位示例把 Heap.Flags/ForceFlags 放在 0x70/0x74，预期值分别为 HEAP_GROWABLE=0x2 与 0。这是具体布局经验；堆实现、版本和创建标志可能改变这些条件。字段不符与真实检测分支需要分开判断。

### TLS Callbacks

PE TLS Directory 的 AddressOfCallBacks 指向回调数组；回调可在常规入口或 main 前执行，DLL_PROCESS_ATTACH 等原因值决定行为。只从 main 建模可能遗漏早期检测或解密。登记结构、回调地址、触发原因和随后状态是必要知识，原链接器与菜单操作不保留。

### Hardware Breakpoint Detection：硬件寄存器

GetThreadContext 的 CONTEXT_DEBUG_REGISTERS 及异常上下文可暴露 DR0–DR3。某寄存器非零不等于所有情况下都代表当前有效断点，还需控制位与线程上下文；源资料重点是线程可见调试状态。

### Software Breakpoint Detection：INT3 与校验

代码区域中的 0xCC 与累加和、CRC/哈希可以揭示字节变化。但合法 INT3、填充字节和误选范围也会出现该字节。软件断点改代码，硬件位置使用调试寄存器；两种检测分别观察不同状态。

### Exception-Based Anti-Debug：异常派发

UnhandledExceptionFilter、RaiseException 和 INT 2D 的行为取决于异常首次/再次处理与观察端策略。异常是否到达程序 handler、指令指针如何变化与后续路径，比简单“继续即有调试器”的表述更可靠。

### NtSetInformationThread：线程隐藏

原例信息类 0x11 为 ThreadHideFromDebugger。其意图是改变线程调试事件的可见性；返回状态、线程身份和实际收到的异常/位置事件共同支持结论。缺少事件也可能来自未到达路径。

## Anti-VM / Anti-Sandbox：环境指纹

### CPUID Hypervisor Bit

CPUID leaf 1 的 ECX bit31 表达 hypervisor-present，leaf 0x40000000 可提供品牌字段。原文列 VMwareVMware、Microsoft Hv、KVMKVMKVM、XenVMMXenVMM 为指纹；品牌字段需按实际寄存器组合解释，不能将单个寄存器地址当作完整字符串。

### MAC 与硬件标识

原资料 OUI 线索：VMware 00:0C:29/00:50:56，VirtualBox 08:00:27，Hyper-V 00:15:5D，Parallels 00:1C:42，QEMU 52:54:00。它们是弱指纹，可以被配置或与其他设备重合。

### Timing-Based VM Detection

某些指令在虚拟化条件下涉及额外开销，原例观察 rdtsc 包围 CPUID 的差并举阈值 500。单位、硬件、调度与时钟虚拟化决定分布，不能视为通用阈值。

### File / Registry Artifacts

已有检测样本关注 vm*.sys、vbox*.dll、VBoxService.exe，VMware Tools 注册表，VMTools/VBoxService 服务，vmtoolsd.exe、VBoxTray.exe、qemu-ga.exe，以及 Linux DMI product_name 与内核 hypervisor 日志。证据应包含实际读取项、字节、匹配方式与分支，而非仅存在相似文件名。

### Resource Checks

处理器数、物理内存和磁盘总量可被用于筛选环境。原检测例为少于 2 CPU、少于 2GB RAM、磁盘少于约 60GB；原资料还列 4 CPU/8GB/100GB 作为环境背景。这里保留阈值与判断语义，不规定环境配置；低资源也可能是真实设备。

## Anti-DBI：插桩指纹

### Frida Detection

已有检测维度包括映射中 frida/gadget 名称、本机端口 27042、libc 函数入口是否出现跳转字节、/proc/self/task 的 gmain/gdbus/frida-* 线程名，以及 Windows 的 frida-* 命名管道。0xE9/0xFF 等入口字节不是可靠单项判据，合法跳板也可能存在。

原反检测示例作用于 strstr 返回值和初始化时点；知识含义是“观察机制本身可以成为被检测输入”。需要区分原始环境信息与经过修改的信息，不保留截获代码。

### Pin / DynamoRIO Detection

pin-/pinbin/pinatrace、dynamorio/drcov/drrun 相关映射和已知指令序列的耗时都可能作为指纹。库名和性能开销属于不同证据源，缺少其中之一不能排除插桩。

## Code Integrity / Self-Hashing：完整性

.text 的 CRC32、函数体 MD5/SHA-256 和预期摘要比较可检测修改。知识记录包含覆盖区间、算法、预期值来源、计算时点和后果。独立 watchdog 可能约每 100000 微秒检查一次并清零结果后退出，因此最终内存为空不代表结果未曾出现。

原资料列字节变化、返回值变化、硬件状态、模拟和快照对照等不同观察路径；它们对代码和环境的影响不同，不能被视为普遍等价的“绕过”。

## Anti-Disassembly：反反汇编

### Opaque Predicates

恒真/恒假条件应按目标位宽和符号性证明。原汇编把 x² AND 1 说成恒零，这个说法有误：其奇偶性与 x 相同。原资料另一个 x(x+1) 的偶数性质才是对应例子；未知溢出与表达式类型仍需明确。保留原主题并标记错误，避免把示例注释当成证明。

### Junk Bytes / Overlapping Instructions

真实跳转后的无关字节可能被线性反汇编误当 call 开头，造成后续错位。真实入口、可达边、原始字节与重新定义的指令边界构成判断依据。

### Jump-in-the-Middle

变长 x86 指令中间也可能是另一路径的合法起点。不能只保留线性指令清单；需要每个可达入口分别解释其字节边界。

### Function Chunking

非连续代码块通过无条件跳转连接，同一函数不一定在单一地址区间。共享栈状态、寄存器契约和返回关系支持函数归属；函数创建菜单不保留。

### Control Flow Flattening

分发状态、虚假分支、指令替换、运行时字符串解密和用后清零可能叠加。Hikari、Polaris/Pluto（Trap Angr）、O-MVLL、Arkari/goron、amice 是原文变体名称；amice 的 VM Flatten 与普通 fla 需要不同语义。详细类别与历史项目覆盖见[OLLVM 专题](references/ollvm-deobfuscation.md)，这里不规定软件优先级。

### MBA 等价

```text
(x AND y) + (x OR y) = x + y
(x XOR y) + 2*(x AND y) = x + y
(x OR y) - (x AND NOT y) = y
NOT(NOT x AND NOT y) = x OR y
(x OR y) AND NOT(x AND y) = x XOR y
```

上述关系需在一致位宽下解释。原文把 `(a|b)+(a&b)-(~a&b)` 标为 a 并不普遍成立，例如 a=b=1 时为 2；故保留其表达式作为待证等价的反例，不照抄工具结果。

## SIGILL 与执行模式切换（Hack.lu 2015）

早期注册的 SIGILL/SIGSEGV/SIGTRAP 处理器可能读取 ucontext 中保存的 PC、解码“非法”字节并更新 PC 或段状态，从而在 32/64 位间切换或实现自定义 opcode。信号编号、fault PC、操作数字节、opcode 长度和返回上下文共同构成隐式边。

## SIGFPE 计数侧信道（PlaidCTF 2017）

SIGFPE 的发生次数可与逐字符校验的进展相关；正确前缀可能触发更多 handler。成立条件是相同初态、相同输入长度以及计数和接受条件之间的实际依赖。计数增大只是证据，不能脱离样本结构概括为“最多一定正确”；不保留进程运行循环。

## 算术轨迹逆序（MeePwn CTF 2017）

在没有内存副作用、依赖明确且各步可逆的算术链中，逆序组合并交换 add/sub、rol/ror，xor 自逆。寄存器位宽、旋转计数掩码、溢出与已知输出都是逆运算条件。

```text
原链：T1 → T2 → ... → Tn
逆链：inverse(Tn) → ... → inverse(T2) → inverse(T1)
ADD(k) ↔ SUB(k)；ROL(r) ↔ ROR(r)；XOR(k) ↔ XOR(k)
```

PEB.BeingDebugged 可能选择不同比较目标，轨迹中的输出必须标明来自哪个分支。否则逆链正确也会恢复错误目标。不保留汇编器和模拟器调用代码。

### 无 CALL 的栈帧链（THC CTF 2018）

小块通过保存的 RBP、返回地址和交错栈数据连接，leave 恢复栈位置，ret 进入下一函数，因而没有对应 CALL。栈不平衡、巨大栈帧错误、许多 leave/ret 小块和栈中函数指针是线索。

原字节变换包含 OR 0x80、XOR 0xCA、多次加 66 取低 8 位，最后 XOR 0xFE；还涉及逆序链和字符序反转。OR 不是整个字节域上的双射，所以下面的案例表达式仍需输入域证据，不能推广为通用可逆变换。

```text
r = byte OR 0x80
重复三次：r = ((r XOR 0xCA) + 66) mod 256
r = r XOR 0xFE
```

## 综合覆盖与层次关系

### 检测覆盖面

可按 ptrace、IsDebuggerPresent、NtQuery、rdtsc、GetTickCount、CheckRemoteDebuggerPresent、CPUID、/proc、自定义信号、alarm 和完整性函数组织证据。这个覆盖表描述可能的检测源，不是 agent 必须执行的阶段。

### Layered Anti-Debug：叠加案例

原例包含 TLS 早期检测、main 的自跟踪、watchdog 的计时和 /proc 扫描、代码 CRC32，以及 SIGSEGV 中的真实逻辑。某一检查通过不说明其他层也通过；多个线程、早期入口与信号边之间的依赖应保留。

### 检测与观察维度速查

| 检测 | 状态层 | 关键证据 |
|---|---|---|
| ptrace | Linux 跟踪关系 | 请求、返回值、errno、父子关系 |
| IsDebuggerPresent / NtQuery | Windows 进程状态 | PEB、信息类、状态与输出 |
| rdtsc / 时间函数 | 计时 | 单位、阈值、时间源与差分 |
| /proc/status | 文件视图 | TracerPid、读取时点与命名空间 |
| alarm / SIGTRAP / SIGFPE | 信号派发 | 信号顺序、handler、次数与路径 |
| TLS | 入口前回调 | AddressOfCallBacks、原因值与副作用 |
| DR 扫描 | 调试寄存器 | 线程与 DR 状态 |
| INT3 / CRC | 代码字节 | 原字节、覆盖区间、预期摘要 |
| DBI 检测 | 映射、线程、入口、管道 | 指纹匹配与误报可能 |
| CPUID | CPU 与虚拟化接口 | leaf、寄存器、品牌与时间 |
| ThreadHideFromDebugger | 线程事件 | 信息类 0x11 与实际事件可见性 |

## 证据组织

按检测源、时点、原始状态、判断分支、数据后果和隐式边组织报告。明确修改过的状态、未覆盖的线程、环境策略与例子中的错误；不以缺少报错或一个检测返回正常作为完整行为证明。
