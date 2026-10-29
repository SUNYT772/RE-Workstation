# OAuth / OIDC 信任链

适用问题：需要还原 OAuth 或 OIDC 流程的参数、交换过程与最终被服务端接受的结果。

## 主题与术语

OAuth/OIDC、重定向/callback、state/nonce、PKCE、scope、令牌交换/刷新、同意授权与声明映射。

## 分析关注点

- Map the auth chain in order: entry route, redirect, authorize request, callback, token exchange, refresh, and final accepting service.
- Record scopes, state, nonce, PKCE material, redirect URIs, and claim-bearing tokens before mutating anything.
- Separate token possession from actual identity acceptance.
- Keep browser-visible redirects and backend-visible token exchange in one compact chain.
- Reproduce the smallest redirect-to-acceptance flow that proves the decisive identity edge.

## 技术关系与判断

以下条目说明对象关系、验证证据与判断边界，可按问题组合参照。

### Map The Redirect And Token Path

- Record issuer, client ID, redirect URI, authorize parameters, callback parameters, token endpoint, and refresh path.
- Note which values are user-controlled, derived, cached, or validated: `state`, `nonce`, PKCE verifier, audience, scope, or prompt.
- Keep browser redirects, server-side exchanges, and resulting session state tied together.

### Prove Token-To-Identity Acceptance

- Show how code, ID token, access token, or refresh token turns into app session, claims mapping, tenant selection, or accepted privilege.
- Record token claims, expiration, audience, subject, scopes, and the exact accepting app or backend edge.
- Distinguish UI login success from backend authorization success.

### Reduce To The Decisive OAuth Chain

- Compress the result to the smallest sequence: entry request -> redirect -> callback -> token or claim acceptance -> resulting capability.
- Keep one canonical good flow and one minimal mutated flow if a parameter change matters.
- If the task broadens into generic web routing or storage behavior outside the auth chain, see related web-runtime knowledge.

## 细分参考与关联

- 参考：[oauth-oidc-chain.md](references/oauth-oidc-chain.md)：重定向、令牌检查项与证据组织方式。
- If the hard part is JWT header parsing, claim normalization, key lookup, or token validation confusion after issuance, see [JWT 声明混淆](../competition-jwt-claim-confusion/README.md).

## 证据字段

- Redirect URIs, parameters, codes, token claims, scopes, and the accepting service or callback
- The exact point where claims or tokens become accepted app identity
- One minimal replayable redirect-to-acceptance sequence

来源：原 `CTF-Sandbox-Orchestrator/competition-oauth-oidc-chain/SKILL.md`。
