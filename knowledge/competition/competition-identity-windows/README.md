# 身份与 Windows

适用问题：问题涉及身份流转、可重用凭据、Windows 主机痕迹、企业邮件或跨主机访问。

## 主题与术语

Active Directory（AD）、Kerberos、LDAP、OAuth、Windows 主机取证、凭据材料、企业邮件与跨主机身份关系。

## 分析关注点

- Map the identity or pivot chain before diving into every host artifact.
- Separate credential possession from accepted privilege.
- Correlate identity evidence, host evidence, and mail evidence on one timeline.
- Keep tickets, SIDs, event IDs, mailbox rules, and pivot hosts in compact evidence blocks.
- Reproduce the privilege edge or mail effect from the smallest viable chain.

## 技术关系与判断

以下条目说明对象关系、验证证据与判断边界，可按问题组合参照。

### Identity And AD

- Trace principal origin, sync path, token or ticket minting, claims transformation, group resolution, and accepting service.
- When Kerberos matters, record ticket type, SPN, delegation mode, PAC or group data, encryption type, and cache location.

### Windows Host And Pivoting

- Correlate SAM, SECURITY, SYSTEM, NTDS, DPAPI, LSA secrets, ETW, Sysmon, PowerShell, services, tasks, WMI, WinRM, SMB, and RDP as one pivot graph.
- Express movement as a concrete chain: foothold -> recovered artifact -> replay path -> pivot host -> resulting capability.

### Enterprise Messaging

- Keep phishing lures, consent logs, mailbox rules, and identity-provider events tied together so the mail path and privilege path stay connected.

## 细分参考与关联

- 参考：[identity-windows.md](references/identity-windows.md)：票据、主机与企业消息检查项。
- If the task is primarily a host-to-host pivot, Kerberos replay, or Windows privilege chain, see [Windows 身份与主机关系](../competition-windows-pivot/README.md).
- If the task is specifically about constrained delegation, unconstrained delegation, RBCD, S4U, or ticket-acceptance proof, see [Kerberos 委派](../competition-kerberos-delegation/README.md).
- If the task is specifically about AD CS, certificate templates, EKUs, enrollment rights, PKINIT, or cert-based privilege, see [AD 证书与身份映射](../competition-ad-certificate-abuse/README.md).
- If the task is specifically about OAuth or OIDC claims, callback flow, scopes, consent, or accepted login identity, see [OAuth / OIDC 信任链](../competition-oauth-oidc-chain/README.md).
- If the task is specifically about DPAPI masterkeys, vault blobs, browser or vault secrets, backup-key use, or protected-secret-to-access chains, see [DPAPI 数据与访问链](../competition-dpapi-credential-chain/README.md).
- If the task is specifically about LSASS memory, ticket caches, LUID-linked material, DPAPI context, or replayable host credential artifacts, see [LSASS 与票据材料](../competition-lsass-ticket-material/README.md).
- If the task is specifically about mailbox rules, forwarding, OAuth consent, delegate access, or transport-level mail abuse, see [邮件与委托关系](../competition-mailbox-abuse/README.md).
- If the task is specifically about forced authentication, relay targets, or proving which service accepts relayed auth, see [认证触发与中继链](../competition-relay-coercion-chain/README.md).

## 证据字段

- SIDs, SPNs, ticket fields, event IDs, mailbox rules, and replay points
- Exact host-to-host pivot order and the service that accepts the credential or ticket
- Raw artifacts, parsed summaries, and derived timelines as separate outputs

来源：原 `CTF-Sandbox-Orchestrator/competition-identity-windows/SKILL.md`。
