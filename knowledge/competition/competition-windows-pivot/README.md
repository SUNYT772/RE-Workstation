# Windows 身份与主机关系

适用问题：关键路径涉及跨主机访问、可重用票据材料或 Windows 权限边界。

## 主题与术语

Kerberos、WinRM、SMB、RDP、Windows 凭据、可重用票据、委托与跨主机权限关系。

## 分析关注点

- Compress the pivot into a concrete chain: foothold -> recovered artifact -> replay path -> pivot host -> resulting capability.
- Separate stored credential material from usable privilege.
- Keep host evidence, ticket evidence, and privilege effect on one timeline.
- Record the exact accepting service or host for every replayed artifact.
- Reproduce the smallest pivot that still proves the privilege edge.

## 技术关系与判断

以下条目说明对象关系、验证证据与判断边界，可按问题组合参照。

### Recover The Replay Material

- Inspect SAM, SECURITY, SYSTEM, NTDS, DPAPI, LSA secrets, browser stores, PowerShell history, ETW, Sysmon, and event logs in the active path.
- Distinguish password, hash, ticket, cookie, vault blob, or gMSA material by where it can actually be used.

### Trace The Pivot Chain

- Map the protocol actually used: WinRM, SMB, RDP, WMI, admin shares, remote registry, or service control.
- When Kerberos matters, record SPN, delegation, PAC or group data, encryption type, and the accepting service.
- When AD edges matter, inspect ACLs, GPO links, SIDHistory, delegation, certificate templates, and replication rights.

### Report The Edge

- Keep the pivot path concrete and replayable.
- State what artifact crossed which boundary and what capability appeared on the destination host.

## 细分参考与关联

- 参考：[windows-pivot.md](references/windows-pivot.md)：跨主机关系检查项、Kerberos 证据组织与常见重用误判。
- If the task is specifically about DPAPI masterkeys, browser or vault stores, protected blobs, or proving where a recovered DPAPI secret is accepted, see [DPAPI 数据与访问链](../competition-dpapi-credential-chain/README.md).
- If the task is specifically about LSASS memory, ticket caches, replayable session material, or host-resident credential extraction, see [LSASS 与票据材料](../competition-lsass-ticket-material/README.md).
- If the task is specifically about delegation edges, SPN trust, S4U flow, or which service accepts the delegated ticket, see [Kerberos 委派](../competition-kerberos-delegation/README.md).
- If the hard part is forced authentication, coercion primitives, relay targets, or the service that accepts relayed auth, see [认证触发与中继链](../competition-relay-coercion-chain/README.md).

## 证据字段

- Host names, logon IDs, SIDs, SPNs, ticket fields, service names, and event IDs
- Exact replay point and resulting logon session, token, or group change
- Raw host artifacts and derived timeline separately

来源：原 `CTF-Sandbox-Orchestrator/competition-windows-pivot/SKILL.md`。
