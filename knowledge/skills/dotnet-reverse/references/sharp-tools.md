# Sharp 系列程序：结构、配置与历史能力分类

来源：skills/dotnet-reverse/references/sharp-tools.md

## 适用范围与概念

Sharp 系列 C# 程序可用于理解检测项、命令分派、P/Invoke、协议数据、资源和配置来源。原文含特征修改、安装矩阵和 MCP 注册；本页保留程序结构与技术目标，将运行与平台配置移除，不规定工具依赖。

## 核心原理与技术细节

### 常见 Sharp 系列程序

| 历史程序名 | 原用途类别 | 分析关注点 |
| --- | --- | --- |
| Rubeus | Kerberos，包括 AS-REP、Kerberoast、S4U、票据相关操作 | 参数解析、Interop、本机 Kerberos 接口 |
| SharpHound | BloodHound 数据采集 | LDAP 查询及采集属性集合 |
| SharpShell/SharpWS | 远程执行与横向相关 | WMI/WinRM 调用、命令表示 |
| Seatbelt | 信息收集 | 收集项和条件判断 |
| SharpRoast | Kerberos 服务票据相关 | 票据请求与解析 |
| Inveigh/SharpSploit | 中间人与通用框架 | 反射加载、API 调用链 |

程序名称提供背景，结论仍应落到实际方法、参数、协议字段与调用点。

### 通用分析结构

Program.Main 或入口分派关联子命令与实现类；Interop 命名空间和本机声明揭示托管/本机边界；资源树与静态字段关联配置、证书与模板。少量样本可能加 ConfuserEx，入口表面可读不意味着资源和方法体没有保护。

原特征修改段涉及命令字符串、banner、API 调用和常量，其承载的知识是这些表面特征与真实行为的关系：命名改变可能不改变协议和调用语义，修改调用顺序则可能影响行为。这里保留差异解释，不复制规避步骤。

### Rubeus 结构示例

原结构例由 `Rubeus.CommandLineParser` 解释 args，`switch(command)` 把 kerberoast 子命令关联到 Ask.TGS，`Rubeus.Interop.Lsa*`/Native.cs 连接本机 Kerberos API。`LsaCallAuthenticationPackage` 与 `KERB_RETRIEVE_TKT_REQUEST` 是观察票据请求结构及边界的具体名称。

这些是静态结构标记，不提供调用脚本、请求构造或命令执行。记录可区分参数解析、命令路由、协议结构与本机交互四层，避免只凭 banner 判断行为。

### 内嵌配置提取的知识对象

manifest resources、程序集字段和加密 byte[] 可承载 C2、密钥角色、证书及模板。资源名和资源内容是不同证据；运行时解密返回值与后续使用点解释配置含义。原反射资源枚举承载资源索引目标，本页不保留加载程序集的执行代码。真实密钥及私密证书内容不复制进知识副本。

### 原工具安装矩阵的能力转写

原 Windows GUI 背景包含反编译、IL 检查/修改和交互调试；Linux/macOS 背景包含无界面反编译、metadata/IL 表示和运行时兼容性。dnSpyEx、ILSpy/ilspycmd、de4dot、DIE、dnlib、Mono/.NET 是历史实现名称。

跨平台的关键不是安装指令，而是目标框架、运行时版本、程序集格式和所需观察面是否相符。原“.NET 6/7/8”例说明版本关联，不能据此固定当前依赖。图形界面可用性与底层 metadata 可解释性是不同能力；本页不把原“唯一 GUI”等历史评价当作当下普遍结论。

### dnSpy MCP 集成的知识语义

原 MCP 项目把反编译、IL 检查、分诊和脱混淆观察暴露为结构化接口，有的运行于独立进程，有的与 GUI 扩展关联。其技术目标是让同一程序集、方法 token、IL 和结果字段之间保持一致。项目注册表、客户端名称、工具数量、配置文件、command/args 与扩展安装均属平台运行资料，已移除。

## 观察项与判断依据

入口、命令分派、实现方法、Interop 声明、请求结构和资源内容构成完整解释链。LDAP 属性、WMI/WinRM 行为、Kerberos 结构和反射加载各自需要相应代码依据；字符串修改不能自动证明这些行为改变。

## 故障定位与经验

静态资源不含明文时可能存在运行时解密；P/Invoke 的托管声明与本机实际调用需要分别理解；native loader 与托管程序集不能混为一个格式。反编译视图丢失状态机或异常细节时，IL 与运行时对象值能提供补充证据。

## 证据与报告结构

报告可记录程序身份与混淆特征、入口及命令分派表、实现方法、Interop 结构、协议字段、采集项、资源/配置来源和可观察值。涉及修改时用前后语义解释差异，实际密钥、凭证和执行 payload 不属于知识页资产。

## 相关知识与维护说明

原社区索引包括 Washi《misconceptions-about-dotnet》、dnSpyEx、de4dot、dnlib 项目，Medium 的 .NET/C# spyware 分析、YouTube 的 EXE/DLL 修改教程、看雪 .NET 版、Guided Hacking 工具比较与 Reverse Engineering StackExchange 的 DynamicMethod 讨论。它们作为历史资料名称保留，无包外链接。

原工作站还关联 Codegate 2013 两阶段 XOR+AES-CBC、.NET 速记和 Unity IL2CPP 本机侧知识，这些仅以主题名称表示。状态机、解密与初始化器见 [IL、状态机与运行时观察](common-workflow.md)。
