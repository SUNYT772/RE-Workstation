# AI Agent, Prompt Injection, Cloud, And Supply Chain

Use this reference for prompt-injection challenges, toolchain or MCP tasks, agent memory issues, containerized deployments, CI/CD, cloud control plane analysis, or supply-chain tracing.

## Agent And Prompt Injection

- Treat prompts, tool schemas, retrieved chunks, planner notes, memory files, and handoff messages as challenge artifacts.
- Map the control stack: instruction layers, retrieval layers, memory layers, tool gates, auth material, and side effects.
- Prove one minimal exploit chain from untrusted content to model-visible instruction to tool side effect or exposed secret.
- Distinguish claimed capability from observed capability.

## Cloud, Containers, And CI/CD

- Split build-time, deploy-time, and runtime.
- Reconcile checked-in manifests with live containers, mounted secrets, env, logs, and traffic.
- Trace provenance end-to-end: source -> dependency resolution -> build -> packaging/signing -> publish -> runtime consumer.
- Map registries, metadata services, message buses, object stores, and IAM-looking identities as control surfaces when evidence places them in the analysis path.

## 证据字段

- Exact prompts, retrieved chunks, planner/tool transitions, and final tool args
- Compose/Kubernetes/Terraform snippets tied to live mounts or routes
- Artifact hashes, version drift, registry pulls, CI steps, and resulting runtime hooks

## 常见误判

- Treating a prompt string as authoritative without runtime confirmation
- Confusing infrastructure intent with live deployment truth
- Missing the boundary where a retrieved chunk becomes executable tool input

来源：原 `CTF-Sandbox-Orchestrator/ctf-sandbox-orchestrator/references/agent-cloud.md`。
