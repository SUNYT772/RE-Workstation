# 完整能力目录

按当前问题选择入口；细节在对应模块中展开。原有22个能力域、SRC嵌套知识、CTF专题、经验、常量与模板共同组成工作站。

## 22 个能力域

| 能力 | 知识内容 |
| --- | --- |
| [通用逆向与高阶方法](knowledge/skills/reverse-engineering/README.md) | 汇编、调用约定、程序结构、语言/平台、密码与混淆、符号执行、内核与游戏 |
| [原生二进制深度分析](knowledge/skills/ida-reverse/README.md) | 函数、交叉引用、类型、结构与反编译语义 |
| [二进制结构与控制流](knowledge/skills/radare2/README.md) | 区段、符号、函数、调用图、数据与运行状态 |
| [二进制版本差分](knowledge/skills/binary-diff/README.md) | 函数匹配、符号迁移、跨版本对应与置信度 |
| [补丁与漏洞成因](knowledge/skills/patch-diff-exploit/README.md) | 修补位置、触发条件、根因与影响验证 |
| [原生漏洞与利用链原理](knowledge/skills/pwn-chain/README.md) | 内存破坏、保护机制、栈堆、状态与链式条件 |
| [托管程序与 .NET](knowledge/skills/dotnet-reverse/README.md) | IL、元数据、混淆、运行时、程序集与互操作 |
| [Android / APK](knowledge/skills/apk-reverse/README.md) | DEX、smali、JNI、保护、数据流与运行行为 |
| [移动平台与 iOS](knowledge/skills/mobile-reverse/README.md) | 应用结构、ObjC/Swift、运行时与平台边界 |
| [固件与 IoT](knowledge/skills/firmware-pentest/README.md) | 固件布局、文件系统、架构、接口与更新链 |
| [JavaScript 与前端](knowledge/skills/js-reverse/README.md) | 签名、混淆、环境依赖、调用链与协议参数 |
| [浏览器与界面交互知识](knowledge/skills/browser-automation/README.md) | 页面语义、定位、状态、证据、异步与稳定性 |
| [API 与身份安全](knowledge/skills/api-security/README.md) | REST、GraphQL、对象/功能授权、JWT与OAuth |
| [安全研究与 SRC 知识](knowledge/skills/pentest-tools/README.md) | 完整方法、行业知识、专题手册与分类参考 |
| [链式安全分析](knowledge/skills/attack-chain/README.md) | 入口、信任与权限边界、条件关联与影响证据 |
| [检测与对抗机制研究](knowledge/skills/edr-bypass-re/README.md) | 检测面、调用与行为、内存和遥测差异 |
| [恶意代码分析](knowledge/skills/malware-analysis/README.md) | 静态结构、行为、配置、持久化、取证与归因边界 |
| [LLM 与 agent 安全](knowledge/skills/llm-security/README.md) | 提示注入、模型应用、上下文、RAG、权限与信任边界 |
| [软件供应链](knowledge/skills/supply-chain-security/README.md) | 依赖、SBOM、构建、CI/CD、分发与身份链 |
| [实战经验与先例](knowledge/skills/field-journal/README.md) | 30份经验与索引材料、反例、失败路径与知识复用 |
| [技术文档与报告](knowledge/skills/docs-generator/README.md) | 逆向、安全、CTF、签名、README与API排版 |
| [图表与关系表达](knowledge/skills/diagram-generator/README.md) | 流程、架构、时序、状态、ER、类、依赖与计划 |

## 安全研究的细分知识

- [SRC 研究方法](knowledge/skills/pentest-tools/src-hunter/README.md)：线索、优先级、业务/行业、证据与报告。
- [漏洞专题](knowledge/skills/pentest-tools/src-hunter/references/playbooks/README.md)：API、认证、逻辑、并发、上传、注入、请求解析、缓存与内网等。
- [分类参考](knowledge/skills/pentest-tools/src-hunter/references/payloader/README.md)：Web/内网分类、场景、机制、信号与原始参考的独有知识。

## CTF 与竞赛专题

[CTF 完整目录](knowledge/competition/README.md) 包含原总览与40个专题入口，覆盖 Web运行、身份、云、容器、逆向、Pwn、密码、移动、取证、协议、隐写与组合题型。

## 经验、特征与模板

- [特征与常量](knowledge/patterns/README.md)：密码常量、保护壳、Android检测。
- [原模板方法](knowledge/templates/README.md)：运行时类、数据流、TLS、HTTP、签名、XOR与Native标注。
- [跨域分析地图](references/analysis-map.md)：对象、静态结构、行为、验证与知识沉淀。
- [问题匹配](references/matching.md)：按对象、意图与证据检索。
- [知识编排参考](references/knowledge-layout.md)：模块与经验的组织方式。

[全部知识页](CONTENTS.md) 提供逐文件目录；[来源记录](SOURCES.md) 说明原材料对应关系。
