# Linux Credential Pivot Checklist

## First Pass

- SSH keys, agent sockets, kubeconfigs, cloud tokens, service-account secrets
- Env vars, config files, systemd units, sudoers, capabilities, setuid binaries
- Namespace context, container runtime sockets, control-plane endpoints

## Chain To Reconstruct

- Credential artifact recovered
- Accepting service identified
- Replay or auth path executed
- New session, token, or privilege gained
- Pivot or lateral effect reproduced

## 关联证据

- Artifact side: file or socket path, owner, scope, lifetime
- Replay side: target host, service, protocol, principal
- Effect side: privilege change, new shell, control-plane action, lateral movement

## 常见误判

- Listing secrets without proving acceptance on a target service
- Mixing local privilege escalation and lateral movement in one vague chain
- Ignoring namespace or socket ownership context

来源：原 `CTF-Sandbox-Orchestrator/competition-linux-credential-pivot/references/linux-credential-pivot.md`。
