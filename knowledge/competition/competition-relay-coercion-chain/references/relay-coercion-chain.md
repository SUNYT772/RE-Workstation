# Relay Coercion Checklist

## Chain Segments

- Coercion source and trigger
- Captured or forwarded authentication
- Relay target and protocol
- Acceptance point and resulting effect

## Reconstruction Order

- What triggers the source to authenticate
- Which identity leaves the source
- Where that authentication is relayed
- Which service actually accepts it
- What privilege, enrollment, or artifact results

## 关联证据

- Source side: host, service, trigger, protocol, coerced principal
- Relay side: listener, transcript, target host, protocol, response
- Effect side: accepted service, resulting account effect, privilege, or issued artifact

## 常见误判

- Stopping at “forced auth happened” without proving relay acceptance
- Proving relay acceptance without showing what capability it produced
- Mixing several candidate relay targets without isolating the one that actually accepted the relayed auth

来源：原 `CTF-Sandbox-Orchestrator/competition-relay-coercion-chain/references/relay-coercion-chain.md`。
