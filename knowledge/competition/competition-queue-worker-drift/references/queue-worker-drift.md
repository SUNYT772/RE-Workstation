# Queue Worker Drift Checklist

## First Pass

- Enqueue endpoint, queue or topic name, payload schema, worker process, schedule or delay, downstream store
- Worker env, mounts, credentials, feature flags, config files, retry policy, dedupe behavior
- Final effects: file write, cache mutation, email, report, artifact generation, privilege-bearing action

## Async Chain To Reconstruct

- Request or cron enqueue occurs
- Payload stored or scheduled
- Worker picks it up under its own runtime state
- Retry, backoff, or failure path taken if relevant
- Side effect lands in file, DB, cache, email, or service

## 关联证据

- Enqueue side: route, payload, queue name, task ID
- Worker side: process, env, config, retry metadata, dedupe or lease state
- Effect side: resulting artifact, downstream mutation, timestamps, and replay prerequisites

## 常见误判

- Explaining only the request path and never proving the worker branch
- Ignoring worker-only env or mount differences
- Treating eventual side effects as synchronous behavior without isolating the async boundary

来源：原 `CTF-Sandbox-Orchestrator/competition-queue-worker-drift/references/queue-worker-drift.md`。
