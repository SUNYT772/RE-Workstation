# 请求归一化与解析差异

适用问题：请求在代理、中间件与后端解析器之间具有不同的解释或归一化结果。

## 主题与术语

HTTP 解析差异、归一化、歧义头部、路径解码、传输分帧、Host/forwarded headers、CL/TE 与跨跳路由。

## 分析关注点

- Map every parsing hop: client-facing proxy, gateway, app server, and downstream service.
- Record path normalization, header canonicalization, transfer framing, and host derivation at each hop.
- Capture one accepted baseline request and one differential request with minimal delta.
- Prove which hop interprets the request differently.
- Reproduce one minimal differential path that yields decisive behavior.

## 技术关系与判断

以下条目说明对象关系、验证证据与判断边界，可按问题组合参照。

### Map Parse And Routing Boundaries

- Record `Host`, forwarded headers, path decoding, slash collapsing, dot-segment handling, and case behavior.
- Note `Content-Length`, `Transfer-Encoding`, chunk framing, and connection reuse behavior when relevant.
- Keep edge parser and backend parser decisions side by side.

### Prove Differential Interpretation

- Build paired requests that differ in one canonicalization dimension only.
- Capture proxy logs, backend logs, route match, and downstream request shape.
- Show where route, auth scope, or body boundary diverges.

### Reduce To Decisive Smuggling Chain

- Compress to: crafted request -> parser differential across hops -> unintended routed request or hidden endpoint reach -> resulting effect.
- State whether root cause is path normalization drift, header ambiguity, transfer framing differential, or host-derivation confusion.
- If the chain becomes primarily runtime routing without framing tricks, see runtime routing knowledge.

## 细分参考与关联

- 参考：[request-normalization-smuggling.md](references/request-normalization-smuggling.md)：解析差异检查项与证据组织方式。

## 证据字段

- Raw request pairs, hop-by-hop interpretation, and final routed target
- Exact normalization or framing delta that flips behavior
- One minimal replayable differential request path

来源：原 `CTF-Sandbox-Orchestrator/competition-request-normalization-smuggling/SKILL.md`。
