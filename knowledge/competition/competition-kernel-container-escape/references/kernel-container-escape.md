# Kernel Container Escape Checklist

## First Pass

- Kernel version, runtime type, namespace map, cgroup mode, capability set
- Seccomp profile, AppArmor or SELinux mode, mounts, privileged sockets
- Potential primitives: syscall path, fs boundary, runtime API, namespace leak

## Chain To Reconstruct

- Isolation baseline recorded
- Exploit or misconfig primitive triggered
- Boundary crossover evidence captured
- Host-relevant capability appears
- Chain reproduces from reset baseline

## 关联证据

- Context side: kernel and runtime config, namespace and capability state
- Primitive side: controllable input, trigger, affected object, observables
- Effect side: identity change, host visibility, privileged action, persistence edge

## 常见误判

- Treating container root as host compromise without crossover proof
- Mixing several primitives before proving one decisive chain
- Claiming escape from crash artifacts without stable capability evidence

来源：原 `CTF-Sandbox-Orchestrator/competition-kernel-container-escape/references/kernel-container-escape.md`。
