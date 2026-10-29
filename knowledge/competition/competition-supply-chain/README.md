# 供应链

适用问题：问题涉及来源证明、依赖差异、构建输出、发布流程或实际交付的运行产物。

## 主题与术语

CI/CD、registry、依赖差异、恶意包、产物来源、镜像构建、构建/发布篡改、签名与实际运行消费者。

## 分析关注点

- Split the problem into source, dependency resolution, build, packaging, publish, and runtime consumption.
- Decide where the first divergence occurs between intended artifact and runtime artifact.
- Keep provenance as a compact chain, not a scattered set of observations.
- Reproduce the smallest possible build or package path that still shows the issue.
- Separate checked-in intent from what the pipeline actually emitted.

## 技术关系与判断

以下条目说明对象关系、验证证据与判断边界，可按问题组合参照。

### Trace Provenance End-To-End

- Map source checkout, lockfiles, dependency fetch, pre/post-install steps, build scripts, packaging, publish target, and runtime consumer.
- Compare declared version, resolved version, and shipped artifact.
- Note registry, cache, mirror, or CI environment differences.

### Reconcile Build-Time And Runtime

- Compare manifests with image layers, mounted secrets, generated files, and runtime hooks.
- Identify whether the decisive mutation happens in dependency install, build step, publish step, or runtime bootstrap.

### Report The Break Point

- State the earliest point where provenance diverges.
- Keep evidence in one short chain from source to runtime consumer.

## 细分参考与关联

- 参考：[supply-chain.md](references/supply-chain.md)：来源链检查项、证据组织方式与流水线常见失败原因。

## 证据字段

- Declared dependency, resolved dependency, and runtime artifact versions
- CI step names, registry pulls, artifact hashes, and image or package layers
- The runtime consumer that actually accepts or executes the artifact

来源：原 `CTF-Sandbox-Orchestrator/competition-supply-chain/SKILL.md`。
