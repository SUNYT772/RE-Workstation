# Graphql And Rpc Drift Checklist

## First Pass

- GraphQL schema, introspection output, persisted query ids, RPC manifest, generated client, OpenAPI doc
- Operation names, fields, variables, method, path, version headers, auth context
- Live handlers, fallback routes, hidden operations, stale fields, client-only guards

## Chain To Reconstruct

- Declared contract is identified
- Real request shape is captured
- Handler normalization or hidden branch is observed
- Contract drift or hidden operation is confirmed
- Resulting capability or artifact is reproduced

## 关联证据

- Contract side: schema, manifest, generated client, version marker, persisted query id
- Request side: operation name, variables, method, path, headers, cookies
- Effect side: hidden data, accepted action, privilege, or backend state change

## 常见误判

- Treating generated clients as proof of the only supported operations
- Looking at GraphQL schema or OpenAPI docs without capturing real requests
- Describing hidden operations without proving the exact handler branch they hit

来源：原 `CTF-Sandbox-Orchestrator/competition-graphql-rpc-drift/references/graphql-rpc-drift.md`。
