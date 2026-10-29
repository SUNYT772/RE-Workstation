# Kerberos 委派

适用问题：需要判断具体委托关系、签发的票据类型，以及实际接受该票据的服务。

## 主题与术语

Kerberos、SPN、S4U、RBCD、约束/非约束委托、服务票据签发与接受。

## 分析关注点

- Write the trust chain first: principal -> delegation edge -> ticket type -> target SPN -> accepting service -> resulting privilege.
- Separate ticket possession from accepted privilege.
- Keep SPNs, delegation mode, PAC/group data, encryption type, and service acceptance in one compact evidence block.
- Reproduce one minimal delegation chain before broadening into variants.
- Tie every privilege claim to a specific accepted ticket or service-side effect.

## 技术关系与判断

以下条目说明对象关系、验证证据与判断边界，可按问题组合参照。

### Identify The Delegation Edge

- Determine whether the path is constrained delegation, unconstrained delegation, resource-based constrained delegation, protocol transition, or another trust edge.
- Inspect SPNs, ACLs, service accounts, SIDHistory, certificate templates, and replication rights only when they affect the active path.

### Trace Ticket Minting And Acceptance

- Record TGT/TGS type, S4U steps when relevant, delegation flags, PAC or group data, encryption type, cache location, and target SPN.
- Prove which service actually accepts the ticket and what capability appears after acceptance.

### Report The Effective Edge

- Compress the chain into one replayable path, not a vague "domain compromise" statement.
- Separate candidate edges from the edge that really lands privilege.

## 细分参考与关联

- 参考：[kerberos-delegation.md](references/kerberos-delegation.md)：委托检查项、需保留的票据字段与常见举证误判。

## 证据字段

- SPN, ticket type, delegation mode, PAC/group data, encryption type, cache location, accepting service
- Service-side logs, event IDs, logon session changes, or group changes proving effective privilege
- The exact trust edge that makes the ticket replayable

来源：原 `CTF-Sandbox-Orchestrator/competition-kerberos-delegation/SKILL.md`。
