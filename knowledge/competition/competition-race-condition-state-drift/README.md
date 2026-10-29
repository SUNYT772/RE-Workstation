# 并发与状态差异

适用问题：决定性行为依赖请求时序、异步顺序、锁间隙或陈旧状态。

## 主题与术语

竞争窗口、顺序问题、幂等性失败、锁间隙、并发 worker、陈旧读取、重复动作与权限/余额状态差异。

## 分析关注点

- Identify mutable state first: rows, cache keys, queue payloads, session fields, counters, or files.
- Reproduce with smallest concurrent sequence and fixed timing assumptions.
- Capture one baseline run and one racing run with only one variable changed.
- Track read, check, write, enqueue, and commit boundaries separately.
- Prove final state drift from a clean reset.

## 技术关系与判断

以下条目说明对象关系、验证证据与判断边界，可按问题组合参照。

### Map Mutable Boundaries

- Record transaction scope, lock behavior, retry logic, idempotency keys, cache invalidation, and queue handoff.
- Note where read-check-write is split across requests, workers, or services.
- Keep each boundary tied to exact timestamps or sequence numbers.

### Reproduce Timing Window

- Build deterministic concurrent inputs with controlled delay, duplicate requests, or reordered worker execution.
- Compare accepted and rejected paths under identical payloads.
- Record which condition flips when ordering changes.

### Reduce To Decisive Race Chain

- Compress to: request A and B ordering -> stale check or lock gap -> conflicting writes -> resulting capability or artifact.
- State whether root cause is missing lock, weak idempotency, stale cache read, delayed async commit, or retry side effect.
- If the path becomes queue-dominant, see queue worker drift knowledge.

## 细分参考与关联

- 参考：[race-condition-state-drift.md](references/race-condition-state-drift.md)：并发验证思路、证据组织与运行条件一致性检查。

## 证据字段

- Mutable keys, transaction boundaries, lock behavior, and idempotency markers
- Timestamped or sequenced traces for baseline and race runs
- One minimal replayable concurrent sequence proving drift

来源：原 `CTF-Sandbox-Orchestrator/competition-race-condition-state-drift/SKILL.md`。
