# Template Render Path Checklist

## First Pass

- Host, path, route params, route match, loader name, template or layout, response mode
- SSR HTML, inline data, hydration payload, client boot chunk, preview or feature flags
- Hidden fields, alternate partials, meta tags, host-based switches, tenant context

## Chain To Reconstruct

- Request reaches route match
- Loader or server data fetch runs
- Template or layout context is built
- HTML or hydration payload is emitted
- Client takeover or handler follow-up turns it into the decisive effect

## 关联证据

- Route side: host, path, matched route, params, loader or handler
- Render side: template or layout, server-only variables, hydration keys, hidden fields
- Effect side: rendered content, leaked data, privileged action, or bypassed branch

## 常见误判

- Trusting visible UI gating without checking loader or handler enforcement
- Looking only at HTML and ignoring hydration JSON or inline data blobs
- Mixing route selection bugs with client-only rendering differences without proving the server boundary

来源：原 `CTF-Sandbox-Orchestrator/competition-template-render-path/references/template-render-path.md`。
