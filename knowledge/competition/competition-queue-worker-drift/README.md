# 队列与后台处理差异

适用问题：关键效果发生在入队之后、后台处理器内部，或与请求路径不同的异步运行状态中。

## 主题与术语

队列、异步 worker、cron、延迟任务、重试、dead-letter、worker 配置差异与载荷到副作用的关系。

## 分析关注点

- Map the async chain first: enqueue point, queue payload, worker consumer, retries, and final side effect.
- Keep request-time state separate from worker-time state.
- Record queue name, message shape, worker config, retry policy, and downstream store in one chain.
- Compare synchronous path and async path when behavior diverges.
- Reproduce the smallest enqueue-to-side-effect flow that proves the decisive async drift.

## 技术关系与判断

以下条目说明对象关系、验证证据与判断边界，可按问题组合参照。

### Map Enqueue And Worker Identity

- Record queue names, topics, cron schedules, delayed jobs, dead-letter queues, worker processes, and consumer groups.
- Note which config, env vars, feature flags, or credentials exist only in the worker environment.
- Keep enqueue request, stored payload, and worker identity tied together.

### Trace Worker-Only State And Retries

- Show how worker runtime differs from the request path: different env, files, mounts, caches, permissions, or clocks.
- Record retry count, backoff, dedupe keys, failure handling, dead-letter flow, and idempotency behavior.
- Distinguish immediate request success from eventual worker success or failure.

### Reduce To The Decisive Async Chain

- Compress the result to the smallest sequence: enqueue -> worker runtime -> retry or branch -> resulting effect.
- State clearly whether the decisive difference lives in payload shape, worker config, retry path, or downstream consumer.
- If the issue is really about the file parser invoked by the worker, see related file-parser knowledge.

## 细分参考与关联

- 参考：[queue-worker-drift.md](references/queue-worker-drift.md)：队列、重试检查项与证据组织方式。

## 证据字段

- Queue names, payloads, worker identities, retry metadata, dead-letter edges, and downstream effects
- The exact worker-only config or state that changes behavior
- One minimal enqueue-to-side-effect reproduction chain

来源：原 `CTF-Sandbox-Orchestrator/competition-queue-worker-drift/SKILL.md`。
