# 逆向工作站 · RE-Workstation

**让每一次逆向与安全研究，都有方法可查、证据可循、经验可复用。**

从一段 JavaScript 签名、一个 APK、一份固件，到补丁差分、恶意代码、API 鉴权与 CTF 复盘，逆向工作站把分散的专项知识组织成统一入口：根据任务找到领域方法，沿着证据深入分析，再把结果整理成可复核的技术结论。

> **22 个核心能力域 · 40 个 CTF 专题 · 30 份经验与索引材料 · 400+ 个文件 · 1 个统一 Skill 入口**

面向逆向工程师、安全研究人员、CTF 选手，以及希望为 AI Agent 补充专项知识的开发者。支持直接阅读，也支持作为 Agent Skill 按任务查阅。

- **调用入口**：[SKILL.md](SKILL.md)
- **查找能力**：[完整能力目录](CATALOG.md)
- **查阅细分内容**：[全部知识页](CONTENTS.md)
- **组织结果**：[分析结构](references/framework.md) · [证据表达](references/evidence.md)
- **使用与许可**：[GPLv3 许可证](LICENSE) · [来源与许可范围](NOTICE.md) · [免责声明](DISCLAIMER.md)

## 核心亮点

- **围绕真实分析对象组织知识**：程序、APK、固件、接口、流量、日志、身份材料与模型应用，都有对应的专项入口。
- **覆盖问题之间的关联**：从静态结构、运行行为，到版本变化、信任边界和影响证据，支持跨领域组合查阅。
- **保留判断依据与失败经验**：关注适用条件、观察信号、反例与验证方式，帮助解释“为什么成立”以及“哪里还缺证据”。
- **兼顾研究与交付**：包含技术报告、证据组织、流程图、架构图、时序图与关系表达的参考结构。
- **完整目录便携使用**：UTF-8 文档与包内相对链接，可放入本地知识库，也可随完整 Skill 目录迁移。

## 22 个核心能力域

| 方向 | 覆盖能力 |
| --- | --- |
| **程序与二进制 · 7 域** | 通用逆向与高阶方法、IDA 原生二进制分析、radare2 结构与控制流、二进制版本差分、补丁与漏洞成因、Pwn 与利用链原理、.NET 托管程序 |
| **移动与设备 · 3 域** | Android / APK、移动平台与 iOS、固件与 IoT |
| **Web 与接口 · 5 域** | JavaScript 逆向、浏览器与界面交互知识、API 与身份安全、安全研究与 SRC、链式安全分析 |
| **对抗与新型安全 · 4 域** | 检测与对抗机制、恶意代码分析、LLM / Agent 安全、软件供应链 |
| **经验与技术交付 · 3 域** | 实战经验与先例、技术文档与报告、图表与关系表达 |

每个领域的细分说明和知识入口见 [完整能力目录](CATALOG.md)。

## 40 个 CTF 专题与细分知识

[CTF 专题目录](knowledge/competition/README.md) 覆盖 Web、逆向与 Pwn、密码与移动、云与容器、Windows 身份、协议与取证、隐写、提示注入和供应链等方向，并展开 OAuth / OIDC、JWT、Kerberos、AD 证书、GraphQL / RPC、DPAPI、PCAP、请求解析差异和内核容器边界等专题。

安全研究部分还包含 [SRC 方法与行业知识](knowledge/skills/pentest-tools/src-hunter/README.md)、[漏洞专题手册](knowledge/skills/pentest-tools/src-hunter/references/playbooks/README.md) 和 [Web / 内网分类参考](knowledge/skills/pentest-tools/src-hunter/references/payloader/README.md)。经验、特征常量与分析模板分别保留独立入口。

## 可以怎样使用

| 当前任务 | 可查阅的内容 |
| --- | --- |
| 分析已授权 APK 的 JNI 调用与数据流 | APK、通用逆向、移动平台、经验库 |
| 追踪 JavaScript 签名参数的生成逻辑 | JS 逆向、浏览器状态、协议与签名模板 |
| 比较修补前后的二进制，解释漏洞根因 | 二进制差分、补丁分析、影响验证与证据表达 |
| 检查自有 API 的身份与授权边界 | API 安全、SRC 专题、JWT / OAuth、链式分析 |
| 在隔离环境分析可疑样本或固件 | 恶意代码、固件结构、行为证据与报告 |
| 复盘一道 CTF，形成可复用记录 | CTF 专题、经验库、技术文档与关系图 |

### 作为知识库阅读

1. 从 [CATALOG.md](CATALOG.md) 选择当前问题对应的领域。
2. 阅读模块首页，再按需要进入细分参考；跨领域任务可组合查阅。
3. 结合实际材料验证判断，记录证据、前提和不确定性。
4. 参考报告与图表结构整理结果。

### 作为 Agent Skill 使用

保留整个工作站目录，将其放入宿主支持的技能安装位置。入口是根目录的 `SKILL.md`；`CATALOG.md`、`knowledge/` 与 `references/` 必须一起保留。

以 Codex 用户级目录为例：

```text
~/.agents/skills/re-workstation/
├── SKILL.md
├── CATALOG.md
├── knowledge/
└── references/
```

其他 Agent 的安装路径和触发规则以各自文档为准。安装后可明确要求使用 `re-workstation`，例如：

> 使用 re-workstation，分析这份已授权 APK 的 JNI 调用与数据流，给出证据位置、关键判断和待验证项。

工作站提供知识、分析参考与结果结构。反编译、调试、网络访问和样本执行等操作由宿主环境已有的工具及权限决定；安装本项目不会自动安装这些工具或启动分析任务。

## 内容结构

| 位置 | 内容 |
| --- | --- |
| `knowledge/skills/` | 原有 22 个能力域及其细分参考 |
| `knowledge/competition/` | CTF 总览与各专题知识 |
| `knowledge/patterns/` | 常量、指纹、反调试识别知识 |
| `knowledge/templates/` | 原模板承载的方法与输入输出结构 |
| `references/` | 跨领域的知识导航和排版参考 |
| `LICENSE` / `NOTICE.md` | 许可证全文、授权范围与第三方来源说明 |
| `DISCLAIMER.md` | 授权研究、验证风险与责任范围说明 |

从当前任务的对象、问题或已有证据命中知识页，按需要组合。知识页中的分析路径和检查项是可选参考；任务如何推进，由当前 agent 结合用户需求组织。

## 便携阅读

文档使用 UTF-8 与包内相对链接，可作为一个目录存放在本地、云端文件服务或知识库。入口和目录负责定位知识，具体文件读取方式由所在环境提供。

## 来源与版本

根据原 RE-Workstation 专项文档、经验库、模板与 CTF 知识重编排。知识整理日期：2026-10-28；发布说明更新：2026-10-29。来源对应关系见 [内容来源](SOURCES.md) 与 [来源映射](SOURCE-MAP.json)。

## 许可证与免责声明

作者有权授权的原创内容与新增整理文档采用 **GNU GPL v3.0（GPL-3.0-only）**，完整条款见 [LICENSE](LICENSE)。第三方内容保留各自的版权与许可，适用范围见 [NOTICE.md](NOTICE.md)。

使用前请阅读 [免责声明](DISCLAIMER.md)。请在合法授权范围内开展研究，并结合实际环境独立核实结论。免责声明不构成 GPL 之外的额外用途限制，也不替代法定责任或第三方许可。

## 作者与交流

- **作者 QQ**：2856150283
- **QQ 通知群**：1106305314
- **TG 群组**：[https://t.me/ty77252](https://t.me/ty77252)
- **T链 AI CCMAX/GPT 满血中转站**：[https://bblb.site](https://bblb.site)
