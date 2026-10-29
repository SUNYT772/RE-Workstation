# Web, API, And Runtime

Use this reference for web challenges, API challenges, SSR/frontend issues, queue-backed app flows, or any case where browser state and request order matter.

## Default Ladder

- Inspect entry HTML, boot scripts, route registration, hydration data, and runtime config.
- Inspect browser persistence: cookies, localStorage, sessionStorage, IndexedDB, Cache Storage, service workers, and transient globals.
- Capture the real request order before theorizing from source.
- Map backend entrypoints, middleware order, handlers, workers, retries, and downstream state changes.
- Re-run the smallest flow with one variable changed.

## High-Value Targets

- Hidden routes, debug pages, preview modes, experiment toggles, alternate hostnames
- Auth and session flow: minting, refresh, header injection, role derivation, cookie scope
- Upload, import, export, template, archive, and deserialization boundaries
- Background jobs, queues, cron tasks, and event consumers
- Host-header handling, base-URL derivation, proxy headers, path-prefix routing

## 证据字段

- Exact requests: host, path, query, headers, cookies, and body
- Exact responses or failures that reveal parser behavior or hidden branches
- Storage keys, feature flags, worker names, and queue payloads
- Concrete file paths, function names, route names, or runtime hook points

## 常见误判

- Treating UI gating as backend enforcement
- Trusting checked-in source over actively served assets
- Missing the real request order because only the visible UI was inspected
- Ignoring container/proxy routing inputs such as `Host` or forwarded headers

来源：原 `CTF-Sandbox-Orchestrator/ctf-sandbox-orchestrator/references/web-api.md`。
