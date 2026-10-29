# 云元数据与工作负载身份

适用问题：需要区分元数据可达、身份材料签发，以及该身份被下游接受并获得权限的不同边界。

## 主题与术语

云元数据、实例/工作负载/节点身份、link-local 凭据路径、角色承担、SSRF 到元数据的关系与下游权限接受。

## 分析关注点

- Identify which metadata surface is active: instance metadata, workload identity, node identity, task role, or platform-specific token endpoint.
- Record the exact reachability path: local process, pod, container, proxy, SSRF surface, or host route.
- Separate metadata reachability from credential issuance and from downstream privilege acceptance.
- Keep token format, role identity, scope, and accepting API in compact evidence blocks.
- Reproduce the smallest metadata-to-accepted-privilege path that proves the challenge edge.

## 技术关系与判断

以下条目说明对象关系、验证证据与判断边界，可按问题组合参照。

### Map Metadata Reachability

- Record the metadata endpoint, required headers, hop limits, session tokens, workload selectors, or path prefixes.
- Note whether access comes from direct local calls, pod networking, SSRF, sidecar, or host-level routing.
- Keep the reaching surface and the metadata endpoint in one chain.

### Prove Credential Or Identity Issuance

- Show how the metadata response becomes a token, temporary credential, signed identity doc, or platform-specific workload identity.
- Record expiration, role name, subject, audience, issuer, or cloud account mapping that matters downstream.
- Distinguish raw metadata from usable credential material.

### Reduce To The Decisive Trust Path

- Compress the result to the smallest sequence: reaching surface -> metadata call -> credential issued -> accepted cloud or cluster action.
- State clearly whether the weakness lives in reachability, metadata config, role trust, downstream policy, or workload binding.
- If the challenge narrows to RBAC or cluster mutation after credential issuance, see related control-plane knowledge.

## 细分参考与关联

- 参考：[cloud-metadata-path.md](references/cloud-metadata-path.md)：可达性、令牌检查项与证据组织方式。
- If the hard part is first proving a server-side fetch primitive, SSRF reachability, or internal endpoint traversal before metadata itself, see [服务端请求与元数据关系](../competition-ssrf-metadata-pivot/README.md).

## 证据字段

- Metadata endpoints, required headers, reachability path, issued tokens or creds, and accepted APIs
- Role names, audiences, issuers, account bindings, and privilege-bearing actions
- The smallest replayable metadata-to-privilege chain

来源：原 `CTF-Sandbox-Orchestrator/competition-cloud-metadata-path/SKILL.md`。
