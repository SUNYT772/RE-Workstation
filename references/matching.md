# 按问题命中知识

对象、用户想解决的问题、已有证据均可作为入口。旧路由中的具体工具维度重编排为知识主题；多个模块可以同时命中。

| 问题与信号 | 能力域 |
| --- | --- |
| APK、DEX、smali、加固、JNI | Android、通用逆向、Native、壳指纹 |
| EXE/DLL/SO/ELF、反汇编与源语义恢复 | 通用逆向、IDA方法、radare2方法 |
| WASM、Python字节码、.NET、Go、Rust、Swift | 语言专项、托管分析、Go与编译语言参考 |
| OLLVM、平坦化、虚假控制流、MBA、间接分支 | 混淆识别、去混淆、符号执行与结构恢复 |
| Hikari、Polaris、Pluto、O-MVLL、Arkari、goron、Tigress等 | OLLVM及变体参考，按已见特征选择 |
| PDB缺失、符号迁移、跨版本函数对应 | 二进制差分、补丁差分 |
| UAF、栈/堆、ROP、保护机制与错误状态 | Pwn链与原生逆向 |
| 驱动、Rootkit、LKM、内核接口 | 内核驱动逆向、恶意代码 |
| PCAP、内存转储、协议重组、状态机 | 协议逆向、取证与CTF专题 |
| 签名、加密参数、补环境、异步调用链 | JS逆向、算法模板、接口研究 |
| 网页、表单、页面状态、浏览器证据 | 浏览器自动化知识、Web运行专题 |
| iOS、IPA、ObjC、Swift、运行时 | 移动逆向、平台参考 |
| 固件、IoT、ARM/MIPS、文件系统、硬件接口 | 固件研究、平台与CTF布局专题 |
| Unity、IL2CPP、Mono、游戏、反作弊、内存状态 | 引擎逆向、托管/Native、经验库 |
| REST、GraphQL、RPC、BOLA/BFLA、OAuth/JWT | API安全、SRC专题、竞赛身份与接口专题 |
| Web漏洞、业务逻辑、认证、缓存、文件、并发 | 渗透方法、SRC完整专题与分类参考 |
| Windows、域、证书、票据、权限边界 | 链式安全研究、SRC内网参考、竞赛身份专题 |
| 云、容器、K8s、元数据与工作负载身份 | 相关安全研究和CTF专题 |
| LLM、提示注入、RAG、agent边界 | LLM安全、API、供应链与对应竞赛专题 |
| SBOM、依赖、构建、CI/CD与分发 | 供应链安全 |
| 壳、恶意样本、配置、驻留、检测 | 恶意代码、EDR研究、取证 |
| 历史问题、陷阱、先例与失败路径 | field-journal经验库 |
| 技术报告、Writeup、README、API文档 | 文档与报告 |
| 流程、架构、时序、ER、状态、依赖、计划 | 图表与关系表达 |

具体页面见 [完整能力目录](../CATALOG.md) 与 [全部知识页](../CONTENTS.md)。这些入口不规定执行环境、具体工具或agent工作次序。
