# 内核与容器边界

适用问题：需要证明容器上下文与宿主机或更高权限内核上下文之间是否发生边界跨越。

## 主题与术语

内核攻击面、namespace/cgroup、容器隔离、syscall、capability、容器到宿主机边界与逃逸原语验证。

## 分析关注点

- Map runtime isolation first: namespaces, cgroups, seccomp, capabilities, LSM, and mount boundaries.
- Separate exploit prerequisite, primitive, and boundary-crossing proof.
- Record kernel version, config hints, runtime options, and reachable syscall surface.
- Keep instrumented observations separate from pristine challenge path.
- Reproduce one minimal primitive-to-boundary-crossing chain.

## 技术关系与判断

以下条目说明对象关系、验证证据与判断边界，可按问题组合参照。

### Map Isolation And Kernel Surface

- Record namespace map, cgroup mode, capabilities, seccomp profile, AppArmor or SELinux state, mounted filesystems, and runtime sockets.
- Note kernel version, distro build hints, module exposure, and container runtime behavior.
- Keep host and container observations linked to exact node and context.

### Prove Exploit Primitive And Crossover

- Show controllable input, trigger condition, affected object, and observable kernel or runtime state change.
- Capture before and after identity, namespace, mount, or process visibility to prove boundary crossing.
- Distinguish crash-only behavior from stable capability gain.

### Reduce To Decisive Escape Chain

- Compress to: prerequisite state -> primitive trigger -> boundary crossing evidence -> resulting host-level capability.
- State whether root cause is kernel vulnerability, runtime misconfiguration, capability overgrant, or namespace leak.
- If path relies mostly on credential replay after initial foothold, see Linux credential pivot knowledge.

## 细分参考与关联

- 参考：[kernel-container-escape.md](references/kernel-container-escape.md)：隔离与原语检查项、本地和目标运行条件的一致性判断。

## 证据字段

- Kernel and runtime context, capability set, seccomp or LSM state, and namespace map
- Primitive trigger data, boundary crossing evidence, and resulting capability
- One minimal reproducible chain from container context to host-relevant effect

来源：原 `CTF-Sandbox-Orchestrator/competition-kernel-container-escape/SKILL.md`。
