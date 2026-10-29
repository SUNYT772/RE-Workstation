# LSASS 与票据材料

适用问题：关键主机痕迹位于 LSASS、票据缓存或相关凭据材料中，需要判断哪些材料确实可重用。

## 主题与术语

LSASS、Windows 登录会话、Kerberos 票据缓存、DPAPI、SSP 与可重用凭据材料。

## 分析关注点

- Separate raw credential material from actually usable replay edges.
- Record logon session, LUID, ticket cache, package, account, and target service before broad conclusions.
- Keep host artifact, extracted secret, replay attempt, and resulting acceptance in one chain.
- Distinguish password, hash, ticket, DPAPI secret, SSP residue, and token by where each can actually be used.
- Reproduce the smallest host-artifact-to-accepted-privilege path that proves the decisive edge.

## 技术关系与判断

以下条目说明对象关系、验证证据与判断边界，可按问题组合参照。

### Map LSASS And Adjacent Credential State

- Record logon sessions, LUIDs, ticket caches, package names, SSPs, DPAPI context, and any service-account material tied to the active path.
- Note whether the decisive value is a TGT, service ticket, delegated ticket, DPAPI secret, plaintext, hash, or package-specific secret.
- Keep host source, account context, and cache location tied together.

### Prove Replay Or Acceptance

- Show where the extracted material is accepted: SMB, WinRM, service ticket use, DPAPI unwrap, Schannel, or another host or service edge.
- Record SPN, target host, logon session, ticket flags, encryption type, and resulting privilege or token change.
- Distinguish material that is present from material that is actually replayable in this path.

### Reduce To The Decisive Credential Chain

- Compress the result to the smallest sequence: host artifact -> extracted material -> accepted replay or unwrap -> resulting capability.
- State clearly whether the decisive edge lives in LSASS memory, ticket cache reuse, DPAPI context, or accepting service behavior.
- If the task broadens into full host-to-host pivoting, see related Windows pivot knowledge.

## 细分参考与关联

- 参考：[lsass-ticket-material.md](references/lsass-ticket-material.md)：会话与材料重用检查项、证据组织方式。
- If the task is specifically about DPAPI masterkeys, protected blobs, browser or vault stores, or proving which recovered DPAPI secret is accepted, see [DPAPI 数据与访问链](../competition-dpapi-credential-chain/README.md).

## 证据字段

- LUIDs, session IDs, ticket types, SPNs, encryption types, package names, and cache or memory source
- The exact accepting host or service and the resulting privilege or logon effect
- One minimal host-artifact-to-replay sequence that proves the edge

来源：原 `CTF-Sandbox-Orchestrator/competition-lsass-ticket-material/SKILL.md`。
