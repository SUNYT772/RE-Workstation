# EDR 实现、Hook 与遥测知识

来源：skills/edr-bypass-re/SKILL.md

## 适用范围与领域关系

原材料面向终端检测评估、Purple Team、自有产品研究与 Windows 对抗场景，讨论 EDR / Defender / AV 的用户态、内核态与脚本扫描机制。攻击链负责跨系统信任关系，本模块聚焦端点实现；恶意样本分析提供检测与样本行为的反向视角。原路由、安装和 agent 执行规则均已转为知识结构。

## 核心原理

| 监控面 | 机制 | 可观察内容 |
|---|---|---|
| 用户态 ntdll Hook | 函数入口跳转、trampoline、映像内存差异 | 导出、RVA、原始/当前字节及接收模块 |
| kernel callback | Ps / Cm / Ob 等回调 | 进程、线程、映像、注册表和句柄事件 |
| ETW | provider、session、写入与消费链 | 事件来源、启用状态、时间与丢失范围 |
| AMSI | 脚本宿主到扫描接口/provider | 输入缓冲、结果、错误与宿主处理 |

原文的 Unhook、Peruns Fart、间接 syscall、Hell’s/Halo’s/Tartarus Gate、硬件断点、ETW/AMSI 修改、调用栈表象等能力在各专页保留为原理、前提、观察项和失效边界。单个用户态 Hook 的变化不能等同于整个端点失去可见性。

## 从实现得到证据

### 终端组件识别

进程、服务、驱动、minifilter、用户态 DLL 和已注册回调提供不同组件线索。组件名与版本、签名、加载关系应相互印证；不能由单个进程名断言产品实现。指纹表见 Hook 专页。

### Hook 表与映像对照

同版本磁盘映像与当前进程 `.text`、导出入口、跳转目标和 trampoline 接收 DLL 的差异，可构成 Hook 表。差异可能来自运行环境和版本，不能将每个字节差异自动归因于 EDR。原调试器、反汇编和进程扫描调用已转为输入输出描述。

### 多层关系与时间

ntdll inline Hook、ETW-TI、AMSI、内核回调和 Sysmon ProcessCreate 等提供互补证据。原文不同位置给出的修改顺序不完全相同，技术上表达的是动作本身可能先被其他层观察到，不能把一个固定次序当作跨产品有效保证。知识页保留事件因果和顺序失效例子，不形成组合执行链。

### 验证结果

源材料的隔离环境以 Defender AMSI、ETW-TI、Sysmon Event ID 1/7/8/10 和 EDR 控制台为观测来源。结果需要记录环境版本、策略、可见事件、是否阻断及告警关联；无告警可能来自缺失采集、延迟或覆盖不足，不能只据单一控制台证明不可检测。

## 典型场景

### Windows 11、Defender 与 Sysmon

源案例的目标是比较驻留组件运行/回连与云查杀和 Sysmon 告警。它涉及存储内容加密、脚本扫描、ETW、系统调用来源、父进程关系和休眠内存变化六个维度。每个维度的实际状态和检测证据需要独立记录，原组合拳与投递路径改为这些关联问题。

### medium IL 与 Sleep Mask

源场景中中等完整性身份已存在，长期驻留使内存扫描更有机会观察特征。原材料以 Ekko、Foliage、WaitForSingleObjectEx、CreateTimerQueueTimer、休眠期内存/栈变化、ROP 恢复和调用栈为背景。知识目标是解释活动/休眠的时序、页面权限、映像支持和线程恢复一致性，不复制加密或恢复实现。

## 报告与分类

原文引用 MITRE ATT&CK T1562、T1562.001、T1562.006、T1055、T1027，用于描述防御受损、指标阻断、注入与混淆等现象。报告保留组件与策略、Hook/回调/provider、观察时序、输入输出、事件和失败条件、结论范围。主机、IP、域、凭证和个人路径角色化；原授权与范围说明保留为案例背景，不写 agent 行为约束。

## 阅读导航

- [Hook 调研与组件/函数表](references/hook-survey.md)
- [Unhook、系统调用与调用栈](references/unhook-techniques.md)
- [ETW、AMSI 与反取证痕迹](references/telemetry-blinding.md)
