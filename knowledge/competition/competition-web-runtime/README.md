# Web 与接口运行

适用问题：问题主要涉及 Web 行为、浏览器状态、服务端路由、API 调用顺序或后台处理驱动的应用流程。

## 主题与术语

Web/API、SSR、前端、认证/会话、隐藏路由、上传、请求顺序、队列/worker 与前后端状态差异。

## 分析关注点

- Inspect entry HTML, boot scripts, runtime config, and route registration before trusting the visible UI.
- Capture one real request flow end-to-end before making broad claims from source.
- Check browser persistence and backend state together.
- Re-run the smallest flow with one variable changed.

## 技术关系与判断

以下条目说明对象关系、验证证据与判断边界，可按问题组合参照。

### Map The Active Runtime

- Identify active hosts, paths, proxies, containers, and workers.
- Inspect cookies, localStorage, sessionStorage, IndexedDB, Cache Storage, and service workers.
- Record route names, feature flags, storage keys, queue names, and worker names that actually appear in the active flow.

### Capture The Real Request Order

- Record exact host, path, query, headers, cookies, and body for decisive requests.
- Compare successful and failing paths.
- Treat UI gating as a hint, not proof of backend enforcement.

### 路径关系与扩展判断

- Trace middleware order, handlers, auth/session boundaries, uploads, exports, and background jobs.
- Verify hidden routes, alternate hostnames, preview modes, or worker side effects with known input/output and runtime evidence.

## 细分参考与关联

- 参考：[routing-runtime.md](references/routing-runtime.md)：详细检查项、证据组织方式与 Web 常见误判。
- If the task is specifically about SSR loaders, template context, hydration payloads, preview rendering, or render-layer enforcement drift, see [模板与渲染路径](../competition-template-render-path/README.md).
- If the task is specifically about source maps, build manifests, chunk registries, emitted bundles, or recovering hidden runtime structure from served assets, see [构建产物与源码映射](../competition-bundle-sourcemap-recovery/README.md).
- If the task is specifically about GraphQL schemas, RPC manifests, persisted queries, generated clients, or contract-to-handler drift, see [GraphQL / RPC 契约差异](../competition-graphql-rpc-drift/README.md).
- If the task is specifically about SSRF input points, internal endpoint reachability, metadata-service pivots, or token extraction through server-side fetches, see [服务端请求与元数据关系](../competition-ssrf-metadata-pivot/README.md).
- If the task is specifically about race windows, ordering-dependent state mutation, duplicate action effects, or timing-sensitive drift, see [并发与状态差异](../competition-race-condition-state-drift/README.md).
- If the task is specifically about proxy-backend parse differentials, path normalization drift, header ambiguity, or request smuggling routes, see [请求归一化与解析差异](../competition-request-normalization-smuggling/README.md).
- If the task is specifically about browser cookies, storage, IndexedDB, Cache Storage, service workers, or cached auth state, see [浏览器持久状态](../competition-browser-persistence/README.md).
- If the task is specifically about OAuth or OIDC redirects, callback params, PKCE, scopes, token exchange, or claim acceptance, see [OAuth / OIDC 信任链](../competition-oauth-oidc-chain/README.md).
- If the task is specifically about JWT headers, claim normalization, key lookup, `kid`, `alg`, issuer or audience confusion, see [JWT 声明混淆](../competition-jwt-claim-confusion/README.md).
- If the task is specifically about upload parsing, previews, archive extraction, converters, or deserialization chains, see [文件解析链](../competition-file-parser-chain/README.md).
- If the task is specifically about queue payloads, worker-only behavior, retries, cron drift, or async side effects, see [队列与后台处理差异](../competition-queue-worker-drift/README.md).
- If the task is specifically about WebSocket or SSE handshakes, subscriptions, realtime frames, reconnect logic, or frame-driven state changes, see [WebSocket 与实时状态](../competition-websocket-runtime/README.md).
- If the task is specifically about Host headers, vhost routing, reverse proxies, or route-to-service resolution, see [运行路由](../competition-runtime-routing/README.md).
- If the only available evidence is a packet capture and the hard part is stream or protocol reconstruction, see [PCAP 与协议](../competition-pcap-protocol/README.md).

- [Cookie HMAC 密钥复用与声明信任](references/cookie-hmac-key-reuse-auth-bypass.md)：公开输入、签名关系、角色边界与服务端接受证据。

## 证据字段

- Exact requests and responses that prove behavior
- Concrete file paths, function names, route names, and storage keys
- Queue payloads, worker names, or retry behavior when async processing matters

来源：原 `CTF-Sandbox-Orchestrator/competition-web-runtime/SKILL.md`。
