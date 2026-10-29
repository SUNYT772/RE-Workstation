# OAuth OIDC Chain Checklist

## First Pass

- Entry route, issuer, client ID, redirect URI, callback route, token endpoint, refresh path
- Authorize parameters: `response_type`, `scope`, `state`, `nonce`, PKCE challenge, audience, prompt
- Browser redirects, backend token exchanges, stored session state, final app acceptance

## Chain To Reconstruct

- Entry request or login trigger
- Authorize redirect built
- Callback parameters received
- Code or token exchange performed
- Claims or token accepted by app or backend

## 关联证据

- Redirect side: route, parameters, callback values, issuer
- Token side: type, claims, scopes, audience, expiration, subject
- Acceptance side: session created, claims mapped, tenant selected, route unlocked, or privilege granted

## 常见误判

- Stopping at the callback without proving token exchange or claim acceptance
- Treating possession of a token as authorization without showing where it is accepted
- Mixing browser-only redirect evidence and backend-only auth evidence without linking them

来源：原 `CTF-Sandbox-Orchestrator/competition-oauth-oidc-chain/references/oauth-oidc-chain.md`。
