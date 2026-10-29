# GraphQL / RPC 契约差异

适用问题：需要比较声明的接口契约与实际处理器，辨认隐藏、陈旧或具有特权的操作。

## 主题与术语

GraphQL schema、persisted query、RPC manifest、生成客户端、OpenAPI 差异、隐藏操作与契约到处理器的差异。

## 分析关注点

- Collect the declared contract surface first: schema, manifest, generated client, persisted query map, or OpenAPI spec.
- Record actual request shapes, operation names, variables, method, path, and auth context before mutating anything.
- Compare declared contract, generated client behavior, and live handler behavior side by side.
- Preserve one accepted operation and one drifted or hidden operation with the smallest delta.
- Reproduce the smallest contract-to-handler mismatch that proves the decisive branch.

## 技术关系与判断

以下条目说明对象关系、验证证据与判断边界，可按问题组合参照。

### Map The Declared Contract Surface

- Record GraphQL schema, introspection output, persisted query ids, RPC manifests, generated clients, or OpenAPI documents that define the intended surface.
- Note versioned endpoints, client-only guards, hidden enums, optional fields, and operation naming conventions.
- Keep document source and generation path tied to the observed requests.

### Prove Live Handler Behavior

- Capture the real request and response pairs, including operation name, variables, headers, cookies, and status.
- Compare client-side validation, schema expectations, and live handler normalization or fallback behavior.
- Record hidden operations, stale fields, undocumented methods, or handler-only branches that still execute.

### Reduce To The Decisive Drift Path

- Compress the result to the smallest sequence: declared contract -> actual request -> handler branch -> resulting capability.
- State clearly whether the decisive drift lives in generated client assumptions, persisted query mapping, schema version skew, RPC manifest mismatch, or handler-side hidden logic.
- If the task shifts into generic JWT, OAuth, or queue behavior after acceptance, see related specialized knowledge.

## 细分参考与关联

- 参考：[graphql-rpc-drift.md](references/graphql-rpc-drift.md)：接口契约、实际处理器检查项与证据组织方式。

## 证据字段

- Schemas, manifests, generated clients, persisted query ids, operation names, and version markers
- One accepted and one drifted request pair that proves the mismatch
- One minimal contract-to-handler sequence that reaches the decisive effect

来源：原 `CTF-Sandbox-Orchestrator/competition-graphql-rpc-drift/SKILL.md`。
