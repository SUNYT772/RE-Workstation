# .NET 与托管代码分析

保留 CLR/metadata、IL、状态机、混淆器、初始化器、配置与托管/本机边界等知识。dnSpyEx、ILSpy、de4dot、dnlib 和相关项目名称只用作历史背景，不规定工具、安装或 agent 调用。

## 阅读导航

| 知识页 | 内容 |
| --- | --- |
| [托管分析框架](overview.md) | 格式识别、IL 与 C#、混淆特征、证据与领域边界 |
| [IL、状态机与运行时观察](references/common-workflow.md) | IL 修改保真、async/yield、解密器、异常控制流、配置与产物 |
| [混淆器机制与故障特征](references/obfuscators.md) | ConfuserEx、SmartAssembly、Reactor、anti-tamper、anti-debug 及失效原因 |
| [Sharp 系列结构与配置知识](references/sharp-tools.md) | 程序分派、Interop、Kerberos 字段、资源、跨平台能力和历史资料 |

## 内容组织

入口只做导航，各来源保持独立知识页。NativeAOT、IL2CPP、本机 loader、符号迁移和调用链图以相关领域文字说明，未确认跨域文件不生成链接。

来源范围（纯文本）：skills/dotnet-reverse/SKILL.md；skills/dotnet-reverse/references/common-workflow.md；skills/dotnet-reverse/references/obfuscators.md；skills/dotnet-reverse/references/sharp-tools.md。
