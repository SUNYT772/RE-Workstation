# DPAPI 数据与访问链

适用问题：关键 Windows 秘密受 DPAPI 保护，需要判断解封所依赖的上下文与明文被接受的位置。

## 主题与术语

DPAPI blob、masterkey、vault、浏览器凭据存储、受保护秘密、域备份密钥与解封后的访问接受关系。

## 分析关注点

- Separate protected blob, masterkey, decrypting context, and final accepting service.
- Record SID, user or machine context, masterkey path, vault or browser store, and target replay point before broad conclusions.
- Keep DPAPI source artifact, unwrap step, plaintext secret, and acceptance edge in one chain.
- Distinguish local user DPAPI, machine DPAPI, domain backup key use, and application-specific wrapping.
- Reproduce the smallest DPAPI-to-accepted-access path that proves the decisive edge.

## 技术关系与判断

以下条目说明对象关系、验证证据与判断边界，可按问题组合参照。

### Map Protected Secret And DPAPI Context

- Record blob source, masterkey location, SID, protector scope, profile path, credential store, and any application wrapper such as browser encryption or vault metadata.
- Note whether the decisive value lives in Credential Manager, Vault, browser cookies, browser passwords, Wi-Fi profiles, RDP files, or custom app storage.
- Keep protected artifact, masterkey candidate, and account or machine context tied together.

### Prove Unwrap And Acceptance

- Show how the secret is decrypted: user logon material, machine context, domain backup key, or another recovered protector.
- Record plaintext type, target host or service, replay method, and resulting session, token, or data access.
- Distinguish successful blob decryption from actual accepted access.

### Reduce To The Decisive DPAPI Chain

- Compress the result to the smallest sequence: protected artifact -> masterkey or unwrap context -> plaintext secret -> accepted replay or access -> resulting capability.
- State clearly whether the decisive edge lives in masterkey recovery, DPAPI scope confusion, application wrapper handling, or the service that accepts the recovered secret.
- If the task broadens into generic LSASS ticket material or full Windows pivoting, see related host or pivot knowledge.

## 细分参考与关联

- 参考：[dpapi-credential-chain.md](references/dpapi-credential-chain.md)：受保护数据、主密钥检查项与证据组织方式。

## 证据字段

- Blob paths, masterkey paths, SIDs, protector scope, store names, and application wrapper details
- The exact accepting service or dataset unlocked by the recovered plaintext
- One minimal protected-artifact-to-accepted-access sequence that proves the edge

来源：原 `CTF-Sandbox-Orchestrator/competition-dpapi-credential-chain/SKILL.md`。
