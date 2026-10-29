# 服务端请求与元数据关系

适用问题：关键路径涉及服务端请求能力、内部服务可达性或源自元数据的凭据。

## 主题与术语

SSRF 来源与可达性、服务端请求、内部路由、元数据服务、link-local 令牌、ServiceAccount 凭据与令牌接受。

## 分析关注点

- Separate the SSRF source, forwarding layer, reachable target, and accepted downstream credential edge.
- Record request method, URL construction, header behavior, redirects, DNS or host overrides, and response shaping before mutation.
- Map internal host, metadata endpoint, token extraction, and accepting service as one chain.
- Distinguish read-only reachability from credential-bearing access.
- Reproduce the smallest SSRF-to-accepted-access path.

## 技术关系与判断

以下条目说明对象关系、验证证据与判断边界，可按问题组合参照。

### Map SSRF Reachability

- Record source primitive: URL parameter, webhook, image fetcher, importer, proxy endpoint, or backend callback.
- Note normalization steps: scheme filtering, host allowlists, redirects, DNS resolution, path rewrite, and header injection.
- Keep target host, protocol, and response behavior tied to the exact SSRF source.

### Trace Metadata And Credential Pivot

- Show whether metadata endpoints, internal control APIs, or workload identity services are reachable.
- Record token fields, role scope, service account, expiration, and where the token is accepted.
- Distinguish credential extraction success from accepted privilege at a downstream service.

### Reduce To Decisive SSRF Chain

- Compress to: SSRF source -> internal or metadata target -> credential or sensitive response -> accepted replay or API access.
- State whether the decisive edge is parser bypass, allowlist bypass, redirect abuse, header confusion, or metadata trust.
- If the task becomes mostly cloud identity policy analysis, see related cloud metadata knowledge.

## 细分参考与关联

- 参考：[ssrf-metadata-pivot.md](references/ssrf-metadata-pivot.md)：SSRF 检查项、元数据关联路径与证据组织方式。

## 证据字段

- SSRF source point, URL construction rules, reachable hosts, and response deltas
- Extracted token or credential fields, scope, and accepting service
- One minimal SSRF-to-accepted-access replay path

来源：原 `CTF-Sandbox-Orchestrator/competition-ssrf-metadata-pivot/SKILL.md`。
