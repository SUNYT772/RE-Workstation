# OWASP LLM & Agentic AI Top 10 (2025-2026)

来源：skills/llm-security/references/owasp-llm-top10.md

## OWASP Top 10 for LLM Applications v2.0 (2025)

| # | 风险 | 核心问题 | 测试方向 |
|---|------|---------|---------|
| LLM01 | Prompt Injection | 通过构造输入操控模型行为 | 直接注入、间接注入、编码绕过 |
| LLM02 | Sensitive Information Disclosure | PII/API Key/训练数据泄漏 | 提示词提取、输出分析 |
| LLM03 | Supply Chain | 投毒模型/库/数据集 | 模型来源验证、依赖扫描 |
| LLM04 | Data & Model Poisoning | 训练/微调数据后门 | 数据溯源、行为异常检测 |
| LLM05 | Improper Output Handling | 输出导致 XSS/SQLi/RCE | 下游系统注入测试 |
| LLM06 | Excessive Agency | 工具/自主权过大导致实际危害 | 权限审计、人在回路测试 |
| LLM07 | System Prompt Leakage | 提取隐藏指令/密钥/业务逻辑 | 级联提取、canary token |
| LLM08 | Vector & Embedding Weaknesses | RAG 管道攻击、嵌入反转 | 检索投毒、语义相似度攻击 |
| LLM09 | Misinformation | 幻觉在高风险场景构成安全风险 | 事实性验证、置信度校准 |
| LLM10 | Unbounded Consumption | DoS/Denial-of-Wallet | Token 消耗测试、速率限制 |

## OWASP Top 10 for Agentic Applications (ASI 2026)

| # | 风险 | 核心危害 | 测试方向 |
|---|------|---------|---------|
| ASI01 | Agent Goal Hijack | 恶意输入/工具输出劫持目标 | 指令覆盖、目标篡改 |
| ASI02 | Tool Misuse & Exploitation | 合法工具的非预期使用 | 工具链拼接、参数注入 |
| ASI03 | Identity & Privilege Abuse | Agent 越权操作 | 凭证窃取、委派链测试 |
| ASI04 | Agentic Supply Chain | MCP 描述符/第三方工具实时风险 | 动态供应链扫描 |
| ASI05 | Unexpected Code Execution | 提示→工具→脚本 RCE 链 | 多层代码执行测试 |
| ASI06 | Memory & Context Poisoning | 长期记忆/嵌入投毒 | 记忆持久化攻击 |
| ASI07 | Insecure Inter-Agent Communication | 智能体间通信篡改 | 中间人、重放攻击 |
| ASI08 | Cascading Failures | 单点故障触发系统级崩塌 | 故障传播测试 |
| ASI09 | Human-Agent Trust Exploitation | 操纵人类操作员批准危险操作 | 权威偏差/紧迫感测试 |
| ASI10 | Rogue Agents | Agent 自我复制/持续恶意行为 | 持久化后门检测 |

## 原材料的数据分布口径

原材料未给出样本集与统计方法，下列占比保留为历史描述，不能据此推断当前总体：
- LLM01 Prompt Injection: ~45%
- LLM06 Sensitive Info Disclosure: ~20%
- LLM08 Excessive Agency: ~15%
- 其余 7 项: ~20%

## 关键防御目标与观察项

1. 规划与执行分离描述意图解释与外部动作之间的边界
2. 身份、目的、范围、时效共同表达动作授权
3. 动作、记忆、通信提供可关联的安全遥测
4. 熔断、回滚和紧急停止影响故障传播范围
5. 自然语言输入与检索内容的来源决定其信任边界
6. 输出在渲染、执行和查询中的解释需要适配下游语法

## 编号与证据边界

原占比段使用 LLM06 表示敏感信息、LLM08 表示自主权，与上表 v2.0 中 LLM02 / LLM06 不一致；该段按风险名称理解，编号差异保留为源材料风险。规划/执行、身份/目的/范围/时效、动作/记忆/通信遥测、熔断/回滚/停止、输入来源和下游输出解释，是原六项防御的技术目标，不作为本页对 Agent 的强执行约束。
