# agent 与云

适用问题：问题涉及提示到工具执行的关系、检索与记忆边界、部署差异，或构建发布产物的来源。

## 主题与术语

AI agent、提示注入、MCP 与工具链、检索污染、挂载秘密、云与容器、CI/CD、部署差异、注册表来源与构建发布产物。

## 分析关注点

- Decide whether the dominant path is agentic or infrastructure-driven.
- Map one minimal control chain: untrusted input -> visible context -> tool or deployment side effect.
- Distinguish checked-in intent from live runtime truth.
- Keep prompts, tool args, manifests, mounts, and provenance steps in compact evidence blocks.
- Reproduce the exploit or misconfiguration with minimal context and minimal instrumentation.

## 技术关系与判断

以下条目说明对象关系、验证证据与判断边界，可按问题组合参照。

### Agent And Prompt Injection

- Treat prompts, tool schemas, retrieved chunks, planner notes, memory files, and handoffs as challenge artifacts.
- Prove one minimal chain from untrusted content to model-visible instruction to tool side effect.
- Distinguish claimed capability from runtime-exposed capability.

### Cloud, Containers, And CI/CD

- Split build-time, deploy-time, and runtime.
- Reconcile compose or kube manifests with live mounts, env, logs, and traffic.
- Trace provenance from source to dependency resolution to build to publish to runtime consumer.

## 细分参考与关联

- 参考：[agent-cloud.md](references/agent-cloud.md)：控制层关系、实际部署状态与证据组织方式。
- If the task is specifically about prompt-boundary abuse or retrieved-content-to-tool drift, see [提示注入](../competition-prompt-injection/README.md).
- If the task is specifically about CI, dependency provenance, registry drift, or shipped artifacts, see [供应链](../competition-supply-chain/README.md).
- If the task is specifically about queue payloads, async worker drift, retries, or worker-only runtime state, see [队列与后台处理差异](../competition-queue-worker-drift/README.md).
- If the task is specifically about SSRF to internal control surfaces, metadata endpoints, or metadata-derived token pivots, see [服务端请求与元数据关系](../competition-ssrf-metadata-pivot/README.md).
- If the task is specifically about proxy-upstream parse differentials, ambiguous headers, path normalization drift, or request smuggling behavior, see [请求归一化与解析差异](../competition-request-normalization-smuggling/README.md).
- If the task is specifically about metadata-service access, instance or workload identity, link-local token paths, or metadata-derived privilege, see [云元数据与工作负载身份](../competition-cloud-metadata-path/README.md).
- If the task is specifically about kube API permissions, service-account trust, admission behavior, controller drift, or cluster secret exposure, see [Kubernetes 控制面](../competition-k8s-control-plane/README.md).
- If the task is specifically about live mounts, sidecars, init containers, or runtime-only secret exposure, see [容器运行状态](../competition-container-runtime/README.md).
- If the task is specifically about container-to-host boundary crossing, kernel-surface prerequisites, or escape primitive verification, see [内核与容器边界](../competition-kernel-container-escape/README.md).

## 证据字段

- Prompt snippets, retrieved chunks, planner transitions, and final tool args
- Compose or Kubernetes fragments tied to live mounts or routes
- Artifact hashes, dependency drift, CI steps, and the resulting runtime consumer

来源：原 `CTF-Sandbox-Orchestrator/competition-agent-cloud/SKILL.md`。
