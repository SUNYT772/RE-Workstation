# K8s Control Plane Checklist

## First Pass

- Namespaces, service accounts, kubeconfigs, tokens, Roles, ClusterRoles, bindings
- Admission webhooks, mutating or validating policy, controllers, operators, CRDs
- Secrets, ConfigMaps, projected volumes, generated Jobs, and owner references

## Trust Chain To Reconstruct

- Principal or token identified
- RBAC or admission edge established
- Object create, patch, or read action performed
- Controller or scheduler turns object into workload state
- Secret, route, workload, or artifact effect observed

## 关联证据

- Principal side: service account, token source, namespace, binding, verb, resource
- Mutation side: object manifest, admission change, controller output, owner refs
- Effect side: mounted secret, env var, spawned pod, reachable route, or recovered artifact

## 常见误判

- Stopping at a RoleBinding without proving the resulting API action
- Explaining pod behavior without showing which cluster object created it
- Mixing static YAML and live cluster objects without accounting for admission or controller drift

来源：原 `CTF-Sandbox-Orchestrator/competition-k8s-control-plane/references/k8s-control-plane.md`。
