# 容器运行状态

适用问题：需要判断正在运行的容器或 Pod 的实际行为，并与仓库中的部署声明比对。

## 主题与术语

容器/Pod、挂载秘密、sidecar、namespace、init container、入口偏差、路由到容器的解析。

## 分析关注点

- Split intent from reality: manifest, image, startup, live mount, live route, live process.
- Map host -> proxy -> container or pod -> mounted volume -> consuming process.
- Keep secrets, rendered config, init output, and sidecar output separate from static manifests.
- Prove one minimal live path from mounted or injected state to reachable behavior.
- Reproduce the effect with the smallest runtime-specific chain.

## 技术关系与判断

以下条目说明对象关系、验证证据与判断边界，可按问题组合参照。

### Map The Live Runtime

- Compare compose or kube manifests against running containers, pods, mounted volumes, env, sidecars, init containers, and entrypoints.
- Identify which process actually consumes the mounted secret, rendered config, or shared volume output.

### Trace Route And Mount Boundaries

- Map virtual host, reverse proxy, service, container port, filesystem mount, and runtime-generated file paths together.
- Record whether the decisive state is image-baked, env-injected, mounted later, or written by an init/sidecar process.

### Report The Runtime Deviation

- State the earliest point where live runtime diverges from checked-in intent.
- Keep one compact evidence chain from manifest or compose intent to live consumer behavior.

## 细分参考与关联

- 参考：[container-runtime.md](references/container-runtime.md)：运行状态、挂载链检查项与静态声明和实际状态的常见差异。
- If the hard part is kube API permissions, service-account trust, RBAC edges, admission mutations, or controller-created workload drift, see [Kubernetes 控制面](../competition-k8s-control-plane/README.md).
- If the hard part is Host-header routing, path-prefix rewriting, or route-to-service mapping across nodes, see [运行路由](../competition-runtime-routing/README.md).
- If the hard part is proving container-to-host crossover, kernel attack-surface preconditions, or stable escape primitives, see [内核与容器边界](../competition-kernel-container-escape/README.md).
- If the hard part is replaying Linux secrets, socket trust edges, or host-to-host pivots after container foothold, see [Linux 身份与主机关系](../competition-linux-credential-pivot/README.md).

## 证据字段

- Compose/Kubernetes fragments tied to live mounts or routes
- Container IDs, pod names, mount paths, sidecar outputs, rendered config paths, and consuming processes
- The exact route or file path that becomes reachable only at runtime

来源：原 `CTF-Sandbox-Orchestrator/competition-container-runtime/SKILL.md`。
