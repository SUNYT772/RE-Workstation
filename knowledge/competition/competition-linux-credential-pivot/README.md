# Linux 身份与主机关系

适用问题：关键边界是 Linux 凭据材料的来源，以及哪些主机或服务实际接受这些材料。

## 主题与术语

Linux 凭据、服务令牌、SSH、云与容器秘密、socket 信任、ServiceAccount、sudo/capability 与跨主机访问链。

## 分析关注点

- Separate credential storage from accepted privilege.
- Record user, process, namespace, socket, key file, and service trust boundary before conclusions.
- Keep artifact recovery, replay path, and resulting capability in one chain.
- Distinguish local escalation from lateral host pivot.
- Reproduce one minimal artifact-to-accepted-access path.

## 技术关系与判断

以下条目说明对象关系、验证证据与判断边界，可按问题组合参照。

### Map Credential And Trust Artifacts

- Record SSH keys, agent sockets, kubeconfigs, cloud tokens, service-account secrets, env vars, config files, and process memory clues.
- Note sudoers rules, capabilities, setuid binaries, systemd unit context, and namespace boundaries.
- Keep each artifact tied to owner, scope, and expected accepting service.

### Prove Replay And Pivot

- Show where key, token, socket, or secret is accepted: SSH, API, Unix socket, container runtime, or control-plane endpoint.
- Record host target, protocol, principal, and resulting session or privilege.
- Distinguish authentication success from useful capability gain.

### Reduce To Decisive Linux Pivot Chain

- Compress to: recovered artifact -> accepted replay path -> pivot host or privilege transition -> resulting capability.
- State whether root cause is weak key handling, token leakage, socket trust, sudo or capability abuse, or namespace crossover.
- If the chain pivots into kernel exploit boundaries, see kernel container escape knowledge.

## 细分参考与关联

- 参考：[linux-credential-pivot.md](references/linux-credential-pivot.md)：痕迹检查项、材料与接受位置的对应矩阵、证据组织方式。

## 证据字段

- Artifact path, owner, scope, accepting service, and resulting principal
- Exact pivot order with protocol and target host or namespace
- One minimal replayable chain proving capability gain

来源：原 `CTF-Sandbox-Orchestrator/competition-linux-credential-pivot/SKILL.md`。
