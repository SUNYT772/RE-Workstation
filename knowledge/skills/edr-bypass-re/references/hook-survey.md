# EDR Hook 调研速查

来源：skills/edr-bypass-re/references/hook-survey.md

本页保留源材料的技术表格、术语与判断；产品覆盖和有效性为历史描述，代码与固定操作已转为观察依据。


本文档汇总主流 EDR / AV 在用户态与内核态的监控点，用于解释组件、监控点与证据关系。

## 1. 主流 EDR 指纹与 hook 模式速查

| 厂商 / 产品 | 用户态组件 | 内核驱动 | 主要监控面 |
|------------|-----------|---------|-----------|
| CrowdStrike Falcon | `CSFalconService.exe`, `CSAgent.sys` 注入到目标进程 | `CSAgent.sys`, `CSBoot.sys` | 重内核 callback + ETW-TI；用户态 hook 较少（云查) |
| Microsoft Defender for Endpoint (MDE) | `MsMpEng.exe`, `MpClient.dll` | `WdFilter.sys`, `WdBoot.sys`, `WdNisDrv.sys` | AMSI + ETW-TI + ntdll inline hook + kernel callback 全面 |
| SentinelOne | `SentinelAgent.exe`, `SentinelHelperService.exe` | `SentinelMonitor.sys`, `SentinelDeviceControl.sys` | ntdll 用户态 hook 重 + 内核 callback + 自有 ETW provider |
| Elastic Defend (原 Endpoint Security) | `elastic-endpoint.exe` | `elastic-endpoint-driver.sys` | 主要 ETW + 少量 ntdll hook，配合 Elastic Agent 上传 |
| ESET | `ekrn.exe`, `eamsi.dll` | `eamonm.sys`, `epfwwfp.sys` | 用户态 hook 非常多（NtCreateFile / NtOpenProcess 等） |
| Sophos Intercept X | `SophosFileScanner.exe`, `SophosNtpService.exe` | `SophosED.sys`, `hmpalert.sys` | ntdll hook + HMPA 内存防护 + 内核 callback |
| Kaspersky | `avp.exe`, `klif.sys` | `klif.sys`, `klhk.sys` | 重用户态 hook + KLIF 自有微过滤 + 网络过滤驱动 |
| Trend Micro Apex One | `TmListen.exe`, `TmCCSF.dll` | `tmcomm.sys`, `tmactmon.sys` | 用户态 hook + 行为监控驱动 |
| Carbon Black | `RepMgr.exe`, `RepWAV.exe` | `ParityDriver.sys` | 偏内核 callback + ETW |

### 快速指纹输入与输出


进程名、驱动文件名与版本提供指纹输入；输出为组件、进程/驱动身份及版本的关联清单。上表中的字符串是历史识别线索，不规定枚举方式或依赖。


## 2. 用户态 ntdll hook 重点函数

原文列出的重点观察函数（具体覆盖受产品/版本影响）： `ntdll.dll` 导出（按 ATT&CK 行为分组）：

| 函数 | 监控的行为 | ATT&CK |
|------|-----------|--------|
| `NtCreateThreadEx` | 远程线程注入、QueueUserAPC 注入 | T1055.002 / T1055.004 |
| `NtAllocateVirtualMemory` | shellcode 申请 RWX 内存 | T1055 |
| `NtAllocateVirtualMemoryEx` | 跨进程内存申请（Win10+ 新 API） | T1055 |
| `NtProtectVirtualMemory` | 改页面权限 RW→RX | T1055 |
| `NtWriteVirtualMemory` | 跨进程写 shellcode | T1055.012 |
| `NtMapViewOfSection` | section-based 注入（Process Doppelganging / Ghosting） | T1055.013 |
| `NtCreateSection` | 配合 MapViewOfSection | T1055.013 |
| `NtOpenProcess` | 打开目标进程拿 handle | T1057 |
| `NtQueueApcThread` / `NtQueueApcThreadEx` | APC 注入 | T1055.004 |
| `NtCreateProcess` / `NtCreateProcessEx` / `NtCreateUserProcess` | 创建子进程（含 PPID spoof） | T1106 |
| `NtSetContextThread` | 改线程上下文（线程劫持注入） | T1055.003 |
| `NtResumeThread` | 注入完后恢复线程 | T1055 |
| `NtQuerySystemInformation` | 枚举进程 / 驱动 / handle | T1057 / T1082 |
| `NtAdjustPrivilegesToken` | 提权获取 SeDebugPrivilege 等 | T1134 |
| `NtLoadDriver` | 加载内核驱动（BYOVD） | T1543.003 |

### 验证 hook 是否存在


同版本磁盘 ntdll 与进程内存的 `.text` 可以按导出和 RVA 对照。正常入口包含参数寄存器处理、SSN 与系统调用边界；异常跳转可能指向 trampoline。证据保留原始字节、当前指令、跳转目标和模块归属，不复制调试命令。


## 3. 内核 callback 监控点

EDR 注册的常见内核回调（原文关联 BYOVD 与内核读写风险，此处不保留回调移除操作）：

| API | 注册的回调时机 | 防御方用途 |
|-----|--------------|-----------|
| `PsSetCreateProcessNotifyRoutineEx` | 进程创建 / 退出 | 拦截可疑 child process |
| `PsSetCreateThreadNotifyRoutine` | 线程创建 / 退出 | 检测远程线程注入 |
| `PsSetLoadImageNotifyRoutine` | DLL / EXE 加载到任意进程 | 模块完整性 / 未签名拦截 |
| `CmRegisterCallback` / `CmRegisterCallbackEx` | 注册表操作 | 持久化检测 |
| `ObRegisterCallbacks` | `OpenProcess` / `OpenThread` 句柄请求 | 防止 LSASS 句柄获取 (T1003.001) |
| `MmRegisterPhysicalMemoryCallback` | 物理内存映射 | 防 DMA / 内存取证 |
| `IoRegisterFsRegistrationChange` | 文件系统注册 | minifilter 协同 |
| `KeRegisterNmiCallback` | NMI（极少 EDR 用） | 异常监控 |
| `EtwRegister` (内核侧) | 内核 ETW 上报 | 跟 ETW-TI 共生 |

### 已注册 callback 的结构证据


进程、线程和映像回调数组，以及 Callback 对象命名空间，是原材料列出的观察位置。回调地址需要关联实际驱动与版本，表中 API 名是机制线索，不是固定调试器操作。


或用 PChunter / DRVHV 这类工具，普通用户可视化看 callback 列表。

## 4. 静态 dump hook 表（IDA + windbg 流程）

### 流程 A：单进程比对


单进程比较的输入是加载基址/映像尺寸、同版本磁盘映像和内存内容；输出为导出入口差异、RVA、跳转目标及 trampoline 所属 DLL。原始 prologue 与入口跳转的对照帮助解释接管关系。


### 流程 B：批量 hook 表生成

HookHunter 是原材料中批量导出差异表的历史能力名称。


批量表按导出函数关联磁盘 `.text` 与进程内存，原例比较入口前 16 字节。输出包含函数、RVA、差异与跳转类别；固定窗口只是原实现选择，不能证明窗口之外没有修改。


## 5. 进程完整性检测与 tag 格式

pe-sieve 是原材料中进程完整性扫描的历史能力名称；其结果用于差异分类，不构成选择要求。


原进程扫描分为 shellcode、模块完整性、IAT Hook 和数据段四种视角，原等级范围为 0–3。输出是改动模块、地址、类别及可复核映像内容，不保留参数或启动代码。


输出会在 `hooks_dump/<pid>.<name>/` 下产生 `*.tag` 文件，列出 hook 地址：


`modified_modules.tag` 是原输出格式名；示例记录以分号分隔模块基址/模块名以及 Hook 地址/类型，`jmp_far` 与 `jmp_near` 表示跳转分类。地址可转换为 RVA 以对应反汇编视图，本页不保留真实进程路径。


RVA 将地址证据与反汇编位置关联。

### 完整性自检的含义

原 libpe-sieve 示例体现进程启动时的映像完整性检查。Hook 存在可能来自防护组件或分析环境，不能直接证明沙箱，也不自动要求任何后续修改。

## 6. API Monitor v2 动态观察

API Monitor v2（Rohitab）是原材料中关联 API、模块与时序的历史观察视角。


动态观察关联 Native Memory Management、Process/Thread 与 AMSI 类别，关注 NtAllocateVirtualMemory 的调用先后、是否由终端 DLL 中转、模块载入时间与来源。输入是目标进程行为，输出是调用与模块的时序关联。


## 7. 常见 EDR DLL（用户态）速查

| DLL | 厂商 | 备注 |
|-----|------|------|
| `umppc*.dll` | Microsoft Defender | MpClient userland |
| `mpoav.dll` | Microsoft Defender | AMSI provider |
| `aswAMSI.dll` | Avast | AMSI provider |
| `eamsi.dll` | ESET | AMSI provider |
| `IDPMServiceClient.dll` | Sophos | HMPA 注入 |
| `klsihk64.dll` | Kaspersky | 注入到目标进程 |
| `CrowdStrike.Sensor.dll` | CrowdStrike | 旧版本，新版主要靠内核 |
| `SentinelInjection64.dll` | SentinelOne | 用户态注入 |
| `TmUmEvt64.dll` | Trend Micro | 行为监控 |

组件归属帮助解释 DLL 与 Hook 表的对应关系。
