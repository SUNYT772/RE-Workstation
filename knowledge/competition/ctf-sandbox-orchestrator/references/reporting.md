# Reporting And Output Style

Use this reference when the user wants progress updates, findings, reviews, or recovery notes.

## 排版参考

- 报告语言按当前受众与需求选择；简洁表述有助于呈现结论与证据。
- 清晰自然的技术说明有助于连接结论、证据与复核结果。
- 可参考的顺序：结果 → 关键证据 → 验证 → 后续问题。
- 报告结构按需求选用；固定字段模板也是可选排版方式。

## 按结果选择结构

- Web/API/debug: conclusion -> local status -> upstream deviation -> decisive request/response or code location -> next step
- Reverse/DFIR/pwn: verdict -> decisive artifact or primitive -> supporting offsets/files/logs -> verification or next step
- Crypto/stego/mobile: verdict -> transform or trust chain -> decisive bytes/hooks/components -> verification or next step
- Agent/cloud/identity/windows: verdict -> trust/deployment/pivot chain -> decisive evidence block -> verification or next step

## 证据编排

Group related evidence instead of scattering it:

- Paths and code locations in one support bullet
- Offsets, hashes, section names, registry keys, event IDs, or ticket fields in one compact block
- Observed prompt snippets, tool transitions and argument records, retrieved chunks, or manifest fragments in one compact block

Summarize long command output. Extract only the decisive lines unless the user explicitly asks for raw logs.

## 常见排版问题

- Mechanical shells like `[TARGET PARAMETER/ASSET]`
- One giant paragraph mixing conclusion, evidence, and next steps
- Repeating the same path or identifier across many bullets when one support bullet would do

来源：原 `CTF-Sandbox-Orchestrator/ctf-sandbox-orchestrator/references/reporting.md`。
