# JWT Claim Confusion Checklist

## First Pass

- Token form: JWS, JWE, nested token, detached signature, or bearer wrapper
- Header fields: `alg`, `kid`, `typ`, `cty`, `jku`, x5u-like references, embedded keys
- Claim fields: `iss`, `aud`, `sub`, `exp`, `nbf`, `iat`, tenant, role, scope, custom privilege claims

## Validation Chain To Reconstruct

- Token parser invoked
- Key source selected
- Signature or decryption step performed
- Claim validation and normalization applied
- App session or privilege accepted

## 关联证据

- Header side: fields, key source, lookup behavior, cache or remote key material
- Claim side: issuer, audience, subject, roles, scopes, time claims, normalization rules
- Acceptance side: session cookie, route unlock, backend action, or privilege-bearing claim usage

## 常见误判

- Stopping at successful decode without proving authorization acceptance
- Focusing on one claim without showing the whole validation chain
- Ignoring key lookup or normalization behavior that matters more than the raw claim value

来源：原 `CTF-Sandbox-Orchestrator/competition-jwt-claim-confusion/references/jwt-claim-confusion.md`。
