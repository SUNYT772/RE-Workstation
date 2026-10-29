# Runtime Routing Checklist

## Inputs That Commonly Matter

- `Host`, `X-Forwarded-Host`, `X-Forwarded-Proto`, `X-Forwarded-Port`
- Path prefixes, base path rewrites, tenant prefixes, cookie domain or path scope
- `Origin`, websocket upgrade headers, absolute URL generation, callback base URLs

## Route Map To Build

- Client-visible host, scheme, port, and path
- Reverse proxy or ingress rule that matches it
- Target service, container, pod, or local process
- Downstream store, worker, or peer node if the request fans out

## Verification Pattern

- Change one routing input at a time
- Compare both request traces and backend behavior
- Keep checked-in config separate from live config when they differ

## 常见误判

- Inferring node ownership or internal/external routing from a public-looking hostname without checking actual proxy configuration and traffic
- Mixing auth failures with route resolution failures
- Explaining the path from code only without capturing one real request that proves the route

来源：原 `CTF-Sandbox-Orchestrator/competition-runtime-routing/references/runtime-routing.md`。
