# 遥测致盲：ETW / AMSI / 反取证

来源：skills/edr-bypass-re/references/telemetry-blinding.md

本页保留源材料的技术表格、术语与判断；产品覆盖和有效性为历史描述，代码与固定操作已转为观察依据。


EDR 的检测能力很大程度依赖 ETW（Event Tracing for Windows）与 AMSI（Antimalware Scan Interface）这两条遥测管道。
本文档保留这两条管道的内部机制，并保留源材料中 Sysmon / PowerShell logging / 时间戳 spoof 等概念及证据关系。

对照 MITRE ATT&CK：T1562.001 / T1562.002 / T1562.006 / T1070 / T1027。

## 1. ETW 内部结构

ETW 是 Windows 内置的高性能事件追踪框架，EDR 用它做"轻量内核遥测"。
源材料列出的 provider：

| Provider GUID | 名称 | 谁在用 |
|--------------|------|--------|
| `{F4E1897C-BB5D-5668-F1D8-040F4D8DD344}` | Microsoft-Windows-Threat-Intelligence (ETW-TI) | Defender、MDE、第三方 EDR |
| `{A0C1853B-5C40-4B15-8766-3CF1C58F985A}` | Microsoft-Antimalware-Scan-Interface | Defender AMSI 上报 |
| `{22FB2CD6-0E7B-422B-A0C7-2FAD1FD0E716}` | Microsoft-Windows-Kernel-Process | 进程 / 线程基础事件 |
| `{2839FF94-8F12-4E1B-82E3-AF7AF77A450F}` | Microsoft-Windows-DotNETRuntime | .NET 加载、JIT |
| `{E13C0D23-CCBC-4E12-931B-D9CC2EEE27E4}` | .NET CLR | CLR 启动 |

### 关键用户态 API

| API | DLL | 作用 |
|-----|-----|------|
| `EtwEventWrite` | `ntdll.dll` | 写事件（最常用） |
| `EtwEventWriteFull` | `ntdll.dll` | 带 activity ID 的事件 |
| `EtwEventWriteEx` | `ntdll.dll` | 扩展版本 |
| `NtTraceEvent` | `ntdll.dll` | EtwEventWrite 底层 |
| `NtTraceControl` | `ntdll.dll` | 控制 trace session（启/停/查询 provider） |
| `EtwEventEnabled` | `ntdll.dll` | provider 是否启用 |
| `EtwEventRegister` | `ntdll.dll` | 注册 provider |

### 调用链


应用 EventWrite 语义经 TraceLogging 封装、ntdll 的 EtwEventWrite / Full / Ex、NtTraceEvent 系统调用进入内核 ETW core，再由订阅 session 的消费端接收。每层的事件生成、转发和消费是不同观察位置。


## 2. ETW Patch 的四种作用位置

### 方法 A：EtwEventWrite head patch

源例讨论 ntdll!EtwEventWrite 入口状态与返回结果不一致的情形。


入口被改成提前返回成功时，上游可能认为事件已提交，但实际写入链未完成。证据是入口完整性、返回语义与下游事件的差异；原补丁字节和汇编已移除。


对应输入输出：


原 C 代码的知识点为模块/导出定位、页面权限变化、入口内容变化及权限恢复。写入动作本身可能进入其他遥测链，不能由返回成功断言无告警；不复制定位和修改代码。


**可观察的时间关系**：写 ntdll 与权限变化可能先被其他通道观察，后续入口变化不能撤回已提交事件。原 ALPC_MODIFY_PROCESS / PROTECTVM 名称保留为源文线索，具体来源需结合环境记录。

### 方法 B：EtwEventEnabled always-false

另一作用位置是 provider 可用状态的判断：源例将固定 FALSE 与应用是否产生 EventWrite 关联。EventWrite 字节未变化只影响这一完整性观察，不代表其他状态或行为不可见。


EtwEventEnabled 返回 provider 可用状态。状态被错误固定为 FALSE 时，部分应用可能不再生成事件；它与修改 EventWrite 的影响位置不同，接口返回、实际 session 和事件产出需要对照。


### 方法 C：NtTraceControl 关 provider

trace session 状态与 ntdll 字节完整性是两个不同观察对象。


NtTraceControl 关联 trace session 控制，停止操作通常需要较高性能分析权限，例如原文列出的 SeSystemProfilePrivilege。状态变化及其他管道对停止事件的观察构成失效风险，不保留控制调用。


源材料列出的限制：

- 关 session 本身会触发"ETW provider stopped"事件被另一条管道感知
- 需要高权限

### 方法 D：内核态 ETW patch（仅在已有 BYOVD/内核读写时）


原文涉及内核 ETW provider 的启用状态与注册句柄两个内部对象。内核版本、读写权限与可信符号是前提；此处保留内核态状态与事件丢失的关系，不保留修改位置和写入方法。


此处只保留内核权限与遥测状态关系。

## 3. AMSI Bypass

AMSI 是 Windows 提供给 PowerShell / .NET / WMI / VBA 在执行脚本前做反病毒扫描的接口。
原材料聚焦 PowerShell 宿主与 AMSI 接口的关系。

### 经典 AmsiScanBuffer Patch


AmsiScanBuffer 的返回状态、扫描结果和宿主对错误的处理相互关联。源例使用 E_INVALIDARG 作为提前返回情形，且 x86/x64 返回约定不同；本页保留错误语义与 ABI 差异，移除补丁字节和 C 实现。


宿主初始化状态与签名检测：


PowerShell 反射示例改变 AMSI 初始化状态。初始化标记、宿主版本、接口结果和终端签名检测共同影响表现，不保留反射表达式、混淆或修改代码。


### 进阶方案 1：Hardware Breakpoint AMSI Bypass

原方案不以修改 amsi.dll 内容为前提，但调试寄存器与异常处理仍有独立证据：

原方案涉及每线程硬件断点、VEH 与扫描接口的异常上下文；返回状态、控制流与宿主处理之间的关系决定结果，不保留寄存器改写序列。

与同模块 unhook-techniques.md 的 HWBP Blindside 具有异常上下文和 VEH 的共同概念，不规定组合实现。

### 进阶方案 2：AmsiContext / AmsiSession 损坏

源材料将 AmsiContext 校验失败与 AmsiScanBuffer 的错误/结果语义关联，不能仅从“校验失败”推断宿主接受。


AmsiContext 的结构标识和会话有效性会影响校验路径。原例声称特定畸形上下文出现成功状态及 CLEAN 结果；这是需要结合版本/宿主验证的源描述，不能推广为通用行为。


### 进阶方案 3：Reflective 加载副本 amsi.dll

源方案区分系统 amsi.dll 与另一个映像副本，讨论 PowerShell 引擎的接口绑定。可观察项包括副本身份、载入方式、绑定目标、扫描结果与宿主版本；原“适用于高级 EDR”的说法缺少范围证据，不作通用有效性结论。

## 4. 反取证：清除痕迹

### PowerShell ScriptBlock Logging 关闭


ScriptBlockLogging、ModuleLogging、Transcription 三种策略分别影响脚本块、模块与转录记录；组策略和注册表状态决定配置来源，管理权限和实际采集结果需对照。不保留关闭日志的写入命令。


### 清 PowerShell history


当前会话历史与 PSReadLine 持久历史属于不同存储层，清除一层不证明另一层不存在。知识页保留两种证据位置，不保留删除操作。


### 清 Prefetch


Prefetch 文件反映执行痕迹，其访问权限及文件缺失也是证据；按名称删除与整体缺失会产生不同痕迹范围，不保留清理命令。


### 清 ETL log


Trace session、ETL 与 Event Log 的 EVTX 是不同状态/文件概念。原例混合停止 session 与删除 Security 日志；日志服务可能重建文件并生成清除记录（原文引用 Event ID 1102）。wevtsvc 接口完整性也是原文列出的观察点，不提供停止、删除或修改方式。


### 时间戳 spoof (T1070.006)


CreationTime、LastWriteTime、LastAccessTime 与参照文件一致，可能来自元数据复制而非真实活动时间。时间戳应与其他执行和文件系统证据交叉解释，不保留修改代码。


## 5. Sysmon 监控规避

Sysmon 在原材料中承担事件遥测，olaf 是源提及的配置历史名；实际配置决定哪些事件可见。
关键事件：

| Event ID | 含义 |
|----------|------|
| 1 | ProcessCreate（含 PPID、CommandLine、Hash） |
| 7 | ImageLoad（DLL 加载） |
| 8 | CreateRemoteThread |
| 10 | ProcessAccess（OpenProcess） |
| 11 | FileCreate |
| 12/13/14 | 注册表 |
| 22 | DNS Query |
| 25 | ProcessTampering（image hollowing） |

### 事件覆盖与可见性边界

既有进程内的活动不一定产生新 ProcessCreate，但仍可能产生内存、线程和句柄证据。父进程显示值与实际创建因果可分离；Process Hollowing 与 Event ID 25 的关系、模块踩踏/dirty vanity 的映像一致性、远程线程与 APC/Early Bird 的线程时序、DoH/HTTPS 与 DNS 可见性的差异，是原文列出的观察维度。任何一项的变化都不等于所有事件不可见。

## 6. 调用栈、时间戳与软件表象

命令行格式、services.exe 等父进程角色、签名 DLL 的内存内容与磁盘摘要、调用栈与实际控制流之间可能存在表象差异。原 CallStackSpoofer、EnableCallTracing 等术语保留为历史概念；证据需要关联这些层，而不以外观合法直接判断。

## 7. 时序、失败路径与证据到达

动作和遥测链具有先后关系，较早形成的事件可能已经被消费或关联。

原顺序的技术含义：


原执行顺序体现脚本扫描、事件写入、页面权限、用户态 Hook、调用栈和实际行为之间的时序关系：较早动作可以被尚未改变的其他层观察。各节的前提并不完全一致，本页保留因果关系而不规定组合次序。


源材料的失败观察：


原失败例一：ntdll 内容/权限改变先产生 PROTECTVM 与模块修改事件，随后遥测状态改变也无法抹除已提交的证据。原失败例二：敏感进程访问在扫描与遥测仍可见时形成 T1003.001 告警。可迁移经验是独立记录动作时刻、采集时刻和消费端到达时刻。
