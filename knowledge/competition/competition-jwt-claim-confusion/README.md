# JWT 声明混淆

适用问题：需要判断 JWT 头部、声明和密钥选择如何转化为服务端接受的身份。

## 主题与术语

JWT/JWS/JWE、头部解析、密钥选择、kid/alg、issuer/audience、声明接受、角色推导与令牌到身份的映射。

## 分析关注点

- Split the token path into parse, key lookup, signature or decryption, claim validation, and final acceptance.
- Record header fields, claims, key source, issuer, audience, and role mapping before mutating anything.
- Separate possession of a token from the exact service that accepts it.
- Keep parser behavior, trust policy, and resulting app session or privilege in one chain.
- Reproduce the smallest token-to-acceptance flow that proves the decisive confusion.

## 技术关系与判断

以下条目说明对象关系、验证证据与判断边界，可按问题组合参照。

### Map Header And Key Selection

- Record header fields such as `alg`, `kid`, `typ`, `cty`, `jku`, or embedded key material when present.
- Note where keys come from: static config, JWKS, local file, cache, or dynamic lookup.
- Keep token parser, key selection path, and validation mode tied together.

### Prove Claim-To-Privilege Acceptance

- Show how subject, audience, issuer, tenant, scope, role, or custom claims become app session, route access, or backend privilege.
- Record expiration, not-before, clock skew, issuer matching, audience matching, and claim normalization behavior.
- Distinguish token parse success from actual authorization success.

### Reduce To The Decisive JWT Path

- Compress the result to the smallest sequence: token supplied -> parser or key path taken -> claim accepted -> resulting capability.
- Keep one canonical accepted token path and one mutated token path if confusion or bypass depends on a delta.
- If the task broadens into a larger OAuth redirect chain, see related OAuth knowledge.

## 细分参考与关联

- 参考：[jwt-claim-confusion.md](references/jwt-claim-confusion.md)：JWT 头部、声明检查项与证据组织方式。

## 证据字段

- Raw headers, claims, key source, JWKS or local key path, and the accepting service
- The exact validation or normalization step that turns the token into accepted identity
- One minimal replayable token-to-acceptance sequence

来源：原 `CTF-Sandbox-Orchestrator/competition-jwt-claim-confusion/SKILL.md`。
