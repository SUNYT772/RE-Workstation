# WebSocket Runtime Checklist

## Handshake First

- Path, query, cookies, auth headers, `Origin`, upgrade headers, negotiated protocol, upgrade response
- SSE setup path, auth carrier, retry directives, event names, last-event IDs when applicable
- Connection lifecycle: open, subscribe, heartbeat, reconnect, close

## Message Flow To Reconstruct

- Handshake or stream setup
- Auth or subscribe message
- Ack or server acceptance
- Push or command frames
- Persisted, rendered, or backend-visible side effect

## 关联证据

- Setup side: request shape, tokens, cookies, channel or room identity
- Frame side: type, topic, payload schema, order, reconnect behavior
- Effect side: UI update, storage mutation, route unlock, server action, or worker effect

## 常见误判

- Treating keepalive traffic as business logic
- Listing frames without showing which one changes state
- Ignoring reconnect or resubscribe behavior that is necessary to reproduce the issue

来源：原 `CTF-Sandbox-Orchestrator/competition-websocket-runtime/references/websocket-runtime.md`。
