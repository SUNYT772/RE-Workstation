# 认证触发与中继链

适用问题：需要证明从触发身份认证到目标服务实际接受中继身份的完整关系链。

## 主题与术语

强制身份认证、认证触发原语、中继链、目标选择、NTLM 等接受路径与认证到权限/注册/执行结果的关系。

## 分析关注点

- Split the chain into coercion source, captured auth, relay target, acceptance point, and resulting effect.
- Record transport, protocol, and service identity at each hop.
- Separate forced-auth generation from relay success and from downstream privilege.
- Keep coercion trigger, relay transcript, and accepting service in one evidence chain.
- Reproduce the smallest coercion-to-acceptance path that proves the decisive edge.

## 技术关系与判断

以下条目说明对象关系、验证证据与判断边界，可按问题组合参照。

### Map The Coercion Source

- Identify the service, RPC, file path, printer path, WebDAV edge, or protocol trigger that forces authentication.
- Record source host, coerced principal, transport, and any environmental preconditions.
- Keep one compact note of exactly what causes the auth to leave the source.

### Trace The Relay Target

- Record where the authentication lands, how it is forwarded, and which protocol or service consumes it.
- Distinguish capture-only, replay-only, and actual relay acceptance.
- Keep service name, target host, protocol, relay transcript, and acceptance response tied together.

### Reduce To The Decisive Relay Chain

- Compress the result to the smallest sequence: coercion trigger -> relayed auth -> accepted service -> resulting privilege or artifact.
- State clearly whether the decisive weakness lives in the coercion source, the relay target, signing settings, or the accepted downstream service.
- If the path ultimately becomes a certificate-enrollment issue or a pure Kerberos delegation edge, see related specialized knowledge.

## 细分参考与关联

- 参考：[relay-coercion-chain.md](references/relay-coercion-chain.md)：认证触发、中继检查项与证据组织方式。

## 证据字段

- Coercion trigger details, source host, coerced identity, target host, accepting service, and resulting effect
- Relay transcripts, error or acceptance responses, and the exact protocol used at each hop
- The smallest replayable coercion-to-acceptance sequence

来源：原 `CTF-Sandbox-Orchestrator/competition-relay-coercion-chain/SKILL.md`。
