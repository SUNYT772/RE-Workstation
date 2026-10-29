# Prompt Injection And Tool Boundary Checklist

## Map These Layers Explicitly

- System and developer instructions
- User request
- Retrieved chunks or documents
- Memory or summaries
- Planner draft or chain-of-thought proxy artifacts
- Executor or tool adapter
- Final tool invocation and side effect

## Minimal Proof Chain

Use the smallest chain that proves the bug:

- Untrusted content enters context
- Model-visible instruction boundary changes
- Planner or executor behavior drifts
- Tool call or secret access changes
- Side effect becomes observable

## 证据字段

- One compact block for the malicious chunk or prompt
- One compact block for planner drift or intermediate rewrite
- One compact block for final tool args and side effect

## 常见误判

- Treating a malicious string as proof without a side effect
- Mixing several injection variants before one minimal chain is proven
- Forgetting to separate retrieval contamination from executor normalization bugs

来源：原 `CTF-Sandbox-Orchestrator/competition-prompt-injection/references/prompt-injection.md`。
