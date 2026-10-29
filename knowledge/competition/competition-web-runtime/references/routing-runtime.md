# Routing And Runtime Checklist

## 识别线索

- Entry HTML, boot scripts, lazy chunks, hydration data, runtime config
- Browser storage, service workers, cached responses, transient globals
- Proxy headers, host headers, path prefixes, and base-URL derivation

## Web 研究对象

- Hidden routes, debug pages, preview modes, experiment toggles
- Auth minting, refresh logic, cookie scope, header injection, role derivation
- Upload, import, export, archive, template, and deserialization boundaries
- Queue producers, background workers, cron tasks, and downstream state mutations

## 证据字段

- Exact request order and the one request that changes the conclusion
- Decisive request and response snippets instead of full trace dumps
- Storage keys, feature flags, queue names, and worker identifiers
- Route names, handler names, and proxy rules tied to runtime behavior

## 常见误判

- Trusting visible UI over backend enforcement
- Trusting checked-in source over actively served assets
- Ignoring proxy or routing inputs that alter behavior
- Expanding sideways before one minimal flow is reproduced

来源：原 `CTF-Sandbox-Orchestrator/competition-web-runtime/references/routing-runtime.md`。
