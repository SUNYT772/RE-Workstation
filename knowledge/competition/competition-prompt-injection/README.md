# 提示注入

适用问题：问题主要涉及 agent 系统内部的信任边界与不可信内容影响执行的路径。

## 主题与术语

提示注入、检索污染、记忆污染、planner 偏移、MCP/工具边界、工具参数污染与 agent 秘密暴露。

## 分析关注点

- Identify the first untrusted content that becomes model-visible.
- Map the chain from retrieval, memory, or transcript into planner or executor behavior.
- Record the exact point where text becomes a tool argument, file path, network target, or secret request.
- Prove one minimal exploit chain before exploring variants.
- Keep prompt snippets and tool transitions in compact evidence blocks.

## 技术关系与判断

以下条目说明对象关系、验证证据与判断边界，可按问题组合参照。

### Map The Control Stack

- Track system, developer, user, retrieved, memory, planner, and tool-response layers separately.
- Distinguish claimed capability from runtime-exposed capability.
- Note what the model can actually call, read, or mutate.

### Prove The Boundary Crossing

- Reproduce one chain from untrusted text to changed planner behavior, changed tool args, or secret exposure.
- Keep the decisive transcript compact: source chunk, rewritten planner state, final tool invocation.
- Prefer the smallest transcript that still demonstrates the bug.

### Report By Boundary

- State which layer failed: retrieval, summarizer, planner, executor, tool normalization, or output post-processing.
- Separate instruction drift from actual side effect.

## 细分参考与关联

- 参考：[prompt-injection.md](references/prompt-injection.md)：提示边界检查项、证据排版与常见误判。

## 证据字段

- Original malicious chunk or prompt
- Intermediate summary or planner drift if it matters
- Final tool args, file paths, or exposed secret surface

来源：原 `CTF-Sandbox-Orchestrator/competition-prompt-injection/SKILL.md`。
