# 跨版本符号迁移

有旧版符号或逆向结果、目标版本缺少符号时，可利用函数语义、引用与结构对应理解新位置。入口提供导航，完整锚点、字段和验证细节位于知识页。

## 阅读导航

| 知识页 | 内容 |
| --- | --- |
| [符号迁移原理与证据](overview.md) | 锚点可靠性、迭代关系、五类符号、版本场景、上下文与误差 |
| [符号对应记录格式](references/prompt-template.md) | 输入字段、YAML 字段、完整示例、目标地址与引用地址、缓存和解析 |

## 内容组织

原 Prompt 和批量 API 骨架转写为独立的数据契约与知识结构，不保留模型调用、密钥或固定 agent 流程。传统图差分与补丁根因属于相关领域。

来源范围（纯文本）：skills/binary-diff/SKILL.md；skills/binary-diff/references/prompt-template.md。
