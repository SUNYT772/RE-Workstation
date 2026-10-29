# 浏览器持久状态

适用问题：决定性分支依赖浏览器保存的状态，需要结合可见 HTML 与后端实现分析。

## 主题与术语

Cookie、localStorage、sessionStorage、IndexedDB、Cache Storage、Service Worker、离线缓存与客户端会话持久化。

## 分析关注点

- Identify the active persistence surface first: cookie jar, localStorage, sessionStorage, IndexedDB, Cache Storage, or service worker.
- Record origin, scope, domain, path, expiry, and key names before mutating state.
- Tie stored state to one concrete effect: request header, rendered branch, cached response, offline behavior, or hidden route access.
- Separate boot-time state from runtime-mutated state.
- Reproduce the smallest stateful sequence that reaches the decisive branch.

## 技术关系与判断

以下条目说明对象关系、验证证据与判断边界，可按问题组合参照。

### Map Browser State Surfaces

- Inspect cookies, storage buckets, service worker registrations, cache entries, and transient globals exposed during boot.
- Record which origin, host, route, or feature flag each state item actually applies to.
- Keep auth tokens, refresh material, CSRF state, cached responses, and feature toggles in separate evidence blocks.

### Tie State To Runtime Behavior

- Show how stored state becomes request headers, role derivation, route visibility, cached API data, or offline fallback behavior.
- Compare clean-state and mutated-state runs with one variable changed at a time.
- Distinguish UI-only state from backend-accepted state.

### Reduce To The Decisive Persistence Chain

- Compress the result to the smallest chain: initial page or login -> state persisted -> subsequent request or render branch -> resulting capability.
- Keep extracted storage, service worker scripts, and replay steps tied to the same origin and route.
- If the problem broadens into general web routing or worker behavior outside browser persistence, see related web-runtime knowledge.

## 细分参考与关联

- 参考：[browser-persistence.md](references/browser-persistence.md)：浏览器状态、Service Worker 检查项与证据组织方式。

## 证据字段

- Cookie attributes, storage keys, database names, cache keys, service worker scopes, and origin boundaries
- The exact request or render effect caused by each decisive state item
- Clean-state vs mutated-state reproduction steps for the smallest working path

来源：原 `CTF-Sandbox-Orchestrator/competition-browser-persistence/SKILL.md`。
