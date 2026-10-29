# Kubernetes 控制面

适用问题：关键路径涉及 Kubernetes 控制面状态、API 权限或控制器行为，并可能超出单个容器的运行范围。

## 主题与术语

Kubernetes API、ServiceAccount、RBAC、RoleBinding/ClusterRoleBinding、admission webhook、controller、集群秘密与 namespace 范围的状态差异。

## 分析关注点

- Separate manifest intent from live cluster state: API objects, mutations, controllers, secrets, and resulting workloads.
- Identify the active principal first: service account, kubeconfig identity, node credential, webhook, or controller.
- Map the smallest control-plane edge to its workload effect.
- Keep RBAC, service accounts, owner references, namespace boundaries, and secret consumers in compact evidence blocks.
- Reproduce the smallest cluster action that yields the decisive workload or secret effect.

## 技术关系与判断

以下条目说明对象关系、验证证据与判断边界，可按问题组合参照。

### Map The API Trust Path

- Record namespaces, service accounts, Roles, ClusterRoles, bindings, admission hooks, controllers, and the resources they can mutate.
- Distinguish read access, create access, patch access, exec access, and secret access.
- Keep principal, verb, resource, namespace, and resulting object in one chain.

### Trace Mutation To Workload State

- Show how an API action becomes a pod, volume mount, secret exposure, env injection, job run, or controller-created artifact.
- Compare checked-in YAML against live objects after defaulting, admission mutation, or controller reconciliation.
- Distinguish pod-runtime behavior from cluster-level mutation logic.

### Reduce To The Decisive Cluster Path

- Compress the result to the smallest chain: principal -> API permission -> mutated object -> resulting workload, secret, or route effect.
- Keep kube objects, live describes, and consumed secret or config paths tied to the same namespace and controller.
- If the problem narrows down to one container's mount or runtime deviation, see related container-runtime knowledge.

## 细分参考与关联

- 参考：[k8s-control-plane.md](references/k8s-control-plane.md)：RBAC、控制器检查项与证据组织方式。
- If the hard part is metadata-service reachability, workload identity, instance credentials, or metadata-derived privilege, see [云元数据与工作负载身份](../competition-cloud-metadata-path/README.md).

## 证据字段

- Namespace, service account, verb, resource kind, RoleBinding or ClusterRoleBinding, and owner reference chains
- Admission mutations, generated workloads, mounted secrets, and controller-produced drift
- The exact API action or object diff that creates the decisive effect

来源：原 `CTF-Sandbox-Orchestrator/competition-k8s-control-plane/SKILL.md`。
