# WebSocket 与实时状态

适用问题：决定性行为依赖实时握手与消息帧流，需要结合连接状态分析。

## 主题与术语

WebSocket/SSE、握手、认证材料、订阅状态、实时消息 schema、重连与消息帧驱动的状态变化。

## 分析关注点

- Map the handshake first: origin, path, headers, cookies, query, auth token, and upgrade response.
- Separate connection setup, subscription messages, keepalives, server pushes, and reconnect logic.
- Record message schema, topic or channel identity, and state side effects in one chain.
- Tie frames to rendered, stored, or backend-visible effects.
- Reproduce the smallest handshake-plus-frame sequence that reaches the decisive state change.

## 技术关系与判断

以下条目说明对象关系、验证证据与判断边界，可按问题组合参照。

### Map The Realtime Handshake

- Record the initial HTTP or SSE request, upgrade headers, cookies, tokens, query params, origin checks, and negotiated protocol.
- Note whether auth material is carried by headers, cookies, query strings, or initial application frames.
- Keep route, subscription endpoint, and session identity tied together.

### Decode Message Flow

- Separate subscribe, unsubscribe, ack, heartbeat, server push, reconnect, and terminal frames.
- Recover message types, channel IDs, schema fields, and sequencing that matter to behavior.
- Distinguish transport keepalive from application-level business messages.

### Reduce To The Decisive Realtime Path

- Compress the result to the smallest sequence: handshake -> auth or subscribe frame -> pushed or accepted frame -> resulting state change.
- Keep canonical frame order and any replayed minimal order side by side.
- If the hard part is generic protocol reassembly without runtime UI or app-state linkage, see related protocol knowledge.

## 细分参考与关联

- 参考：[websocket-runtime.md](references/websocket-runtime.md)：握手、消息帧检查项与证据组织方式。

## 证据字段

- Handshake headers, cookies, query params, auth material, negotiated subprotocol, and channel IDs
- Frame schemas, subscription messages, server pushes, reconnect flow, and resulting state changes
- The smallest replayable realtime sequence that proves the decisive branch

来源：原 `CTF-Sandbox-Orchestrator/competition-websocket-runtime/SKILL.md`。
