# Unhook / 直接 / 间接 syscall 技术清单

来源：skills/edr-bypass-re/references/unhook-techniques.md

本页保留源材料的技术表格、术语与判断；产品覆盖和有效性为历史描述，代码与固定操作已转为观察依据。


本页按源材料的历史分类，解释用户态 Hook、系统调用、硬件断点与调用栈的关系；分类不代表当前产品有效性评估。
所有技术都对照 MITRE ATT&CK T1562.001 / T1027 / T1055，便于报告输出。

## 1. Peruns Fart / Fresh Ntdll from disk

### 原理

本节讨论当前进程 ntdll 内的用户态 Hook；磁盘映像是对照来源，其可信性与版本需单独确认。
映像恢复概念针对内存 `.text` 与基线的差异，不涵盖所有内核或遥测监控。


原图表达当前进程 ntdll `.text` 与磁盘映像 `.text` 的对应关系：一个是当前执行内容，一个是对照来源。二者版本和可信来源需要匹配，映像映射与权限变化属于独立观察面。


### 实现要点


映像文件、SEC_IMAGE section、映射地址、`.text` 范围、内存权限和内容一致性构成原代码的数据流。恢复前后的页面属性、映像字节和其他遥测可以相互校验；原映射、改写与恢复调用不保留。


### 注意

- 页面权限接口本身也可能存在 Hook，说明入口与后续状态变化之间具有递归依赖。
- ntdll 写入与页面权限改变可能被其他层独立观察。
- 源材料把 KERNEL_MODULE_LOAD、PROTECTVM 与映像/权限变化关联；事件名称及实际产生条件需以环境记录为据。

## 2. 直接 syscall (Direct Syscall)

### 原理

直接系统调用概念区分导出函数入口与系统调用边界。


直接系统调用由用户代码携带系统服务号（SSN）进入内核。x64 的参数约定和服务号具有版本依赖，原文中的固定十六进制值只属于示例，不应当作当前系统事实。


源材料以 syscall 到内核服务的关系解释用户态入口差异；未经过某个导出入口不等于所有用户态或内核监控均不可见。

### 系统调用描述与生成结构


原生成器承担把函数描述转成声明、胶水代码与体系结构 stub 的能力。此处仅保留这种输入输出关系，安装、克隆和生成命令均删除。


输出：


历史输出分为函数声明文件、C 胶水、MASM 汇编和标准 x64 stub。它们分别承载类型、解析和调用边界，不是本知识页的依赖文件。


构建一致性的知识目标：


声明、ABI、汇编与链接需要一致才能维持参数和栈约定；原 Visual Studio 项目步骤改写为这一一致性条件。


### NtCreateFile 的对象与 ABI


NtCreateFile 例子中的知识对象是句柄、OBJECT_ATTRIBUTES、UNICODE_STRING、IO_STATUS_BLOCK 与 NTSTATUS。Unicode 长度与最大长度使用字节计量，原生路径与对象属性确定目标，状态结果与句柄生命周期反映调用是否成功。寄存器/栈保存、SSN 解析和调用约定属于 ABI 问题；文件写入与直接 syscall 骨架不复制。


### 缺点

- syscall 指令位于 implant 自己的 `.text` 段（非 ntdll 内）→ kernel-mode telemetry 容易看出 "syscall from non-ntdll address"
- 这就是 indirect syscall 出现的原因

## 3. 间接 syscall (Indirect Syscall)

### 原理

间接系统调用讨论服务号、指令位置与返回链的来源差异。


间接系统调用把服务号解析与实际 syscall 指令位置分开，控制流落在 ntdll 内现有系统调用边界。入口 RIP 的映像归属与完整调用栈是不同证据，合法映像地址不证明整个行为合法。


源材料以 Nt* 导出附近的 syscall / ret 指令边界说明 RIP 的映像归属；具体指令字节和地址需以对应映像核对。RIP 属于 ntdll 与完整行为可信是不同判断。

### 间接系统调用的模式差异


原 jumper 与 randomized-jumper 分类表达固定或变化的系统调用边界位置；保留概念，不保留生成参数。


生成的 stub：


原 stub 包含服务号解析、边界地址解析和调用约定恢复三类工作；观察结果是 SSN、实际 RIP、返回链是否一致，不复制跳转汇编。


## 4. Hell's Gate / Halo's Gate / Tartarus Gate

三者解决"SSN 动态解析"的演进。

### Hell's Gate

- 假设 ntdll 未被 hook
- 知识对象为 Nt* 导出入口、指令中的服务号与标准 prologue；源“前4字节”的简写需要与具体指令布局核对，不能当固定跨版本偏移
- 原优势描述是减少固定 SSN 的版本依赖；是否可解析仍受版本和入口内容影响
- 缺点：如果 ntdll 已经被 hook（第一字节变成 jmp），提取失败

### Halo's Gate

- 修复 Hell's Gate 的 hook 问题
- 标准 prologue 不可用时，原算法把上下邻近函数作为候选信息；邻居数量、排列和入口完整性是推断条件
- 原推断假设邻近 Nt* 服务号存在连续关系，候选 SSN 的可信度取决于这个假设是否适用于对应映像


邻近导出的服务号关系是 Halo’s Gate 的推断前提。目标入口不可读时，邻居可提供候选关系；原数值仅示例，导出排列、版本与 Hook 改写都会影响推断可靠性。


### Tartarus Gate

- 进一步处理 **Hook 改了 SSN 但保留了 syscall 指令** 的高级 hook
- 同时校验 SSN 与 syscall;ret gadget 地址
- 源材料把三类校验互补视为稳定性来源，未给跨版本对照数据，不能据此保证组合效果

### 算法与历史实现概念


Hell’s Gate、Halo’s Gate、Tartarus Gate 与 SysWhispers3 是原文参考概念；保留算法演进关系，移除仓库获取与自举要求。


## 5. Hardware Breakpoint Blindside

### 原理

源机制连接 DR0–DR3、Hook trampoline、断点异常、VEH 和 RIP 上下文。其观察问题是异常前后控制流是否经过同一检测路径，以及实际系统调用边界和线程状态是否一致；不保留设置或改写次序。

### 优势

- 原方案描述不以写 ntdll 或移除 Hook 为前提，因此可没有对应页面改写事件；这不证明没有其他告警
- Hook 字节仍在与实际控制流经过 Hook 是不同证据
- 内存改写、调试寄存器、异常和上下文属于不同观察面，缺少一种事件不能证明整体不可见

### 上下文与异常结构


硬件断点机制涉及每线程 DR0–DR3、异常分派、VEH 与异常上下文。原骨架展示控制流在断点异常中改变；可观察项是线程调试寄存器、异常代码、上下文和返回路径。每线程至多四个地址以及多线程状态是限制，不保留上下文改写代码。


### 限制

- DRx 是每线程状态，多线程覆盖不能由单线程观察外推
- NtSetContextThread / NtGetContextThread 本身可能被监控，说明上下文访问也是独立证据面，不形成前置绕过要求
- Win11 22H2+ 引入 HVCI / 一些反调试缓解可能干扰

## 6. Call Stack Spoofing

### 问题

源材料将 NtAllocateVirtualMemory / NtCreateThreadEx 等调用、RtlCaptureStackBackTrace 与栈遥测关联，具体采集位置和完整度需要环境证据。non-image-backed memory 帧是源列出的关联信号，不能脱离调用上下文直接认定告警或恶意。

### 方案 A：CallStackSpoofer（William Burgess）

源结构区分真实线程栈、呈现给采集端的返回链与恢复后的线程状态。BaseThreadInitThunk / RtlUserThreadStart 是原返回链中的历史例子；帧所属模块、调用因果和实际控制流要相互对应，不保留切换/填充步骤。

### 方案 B：SilentMoonwalk

更激进，使用 desynchronized stack：


SilentMoonwalk 描述实际控制流与栈展开视图不同步。RSP/RBP、trampoline、RUNTIME_FUNCTION 与 UNWIND_INFO 共同决定 RtlVirtualUnwind 的解释；恢复一致性和映像支持是分析依据，原执行链改为结构关系。


栈展开结果由 RtlVirtualUnwind 与 RUNTIME_FUNCTION / UNWIND_INFO 的解释关系决定，展开视图与真实控制流的差异是分析依据。

### 活动与休眠期观察

原文比较执行期和休眠期调用栈，指出周期采样使休眠状态也有证据价值；具体产品的组合有效性说法不作为通用结论。

## 7. 机制、复杂度与源有效性描述

| 技术 | 关联监控面 | 源复杂度 | 源有效性描述（未复核） | ATT&CK |
|------|------|--------|------------|--------|
| Peruns Fart | 用户态 hook | 低 | 中（易被 ETW 抓） | T1562.001 |
| Direct syscall (SysWhispers) | 用户态 hook | 低 | 低-中（kernel 看 RIP 在 implant） | T1106 / T1562.001 |
| Indirect syscall (jumper) | 用户态 hook + kernel RIP 检测 | 中 | 中-高 | T1106 |
| Hell's / Halo's / Tartarus | SSN 解析 | 中 | 高（基础设施） | T1027 |
| HWBP Blindside | hook + 无写操作 | 高 | 高 | T1562.001 |
| CallStackSpoofer / SilentMoonwalk | call stack telemetry | 高 | 高 | T1564 |

原表保留机制、复杂度和 ATT&CK 的历史分类；有效性栏仅代表源材料经验，未做当前产品核验。
