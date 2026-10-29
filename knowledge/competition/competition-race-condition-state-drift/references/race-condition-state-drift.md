# Race Condition State Drift Checklist

## First Pass

- Mutable rows, cache keys, counters, files, queue payloads
- Read-check-write boundaries, lock scope, retry behavior, idempotency handling
- Worker delays, commit timing, stale reads, invalidation timing

## Chain To Reconstruct

- Baseline ordering established
- Concurrent or reordered flow injected
- Check or lock boundary is bypassed
- Conflicting state mutation lands
- Decisive drift is reproduced

## 关联证据

- State side: key or row, initial value, final value, commit boundaries
- Timing side: request order, delays, retries, queue timing
- Effect side: duplicated action, privilege drift, balance drift, stale authorization

## 常见误判

- Running noisy stress tests without a minimal deterministic sequence
- Mixing several mutable keys without proving which key drives the effect
- Reporting timing sensitivity without final-state parity proof

来源：原 `CTF-Sandbox-Orchestrator/competition-race-condition-state-drift/references/race-condition-state-drift.md`。
