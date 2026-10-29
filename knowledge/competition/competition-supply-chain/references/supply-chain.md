# Supply Chain And CI Checklist

## Provenance Chain

Track this order explicitly:

- Source checkout
- Dependency declaration
- Lockfile or resolver result
- Build step
- Packaging or signing step
- Publish target
- Runtime consumer

## High-Value Evidence

- Version drift between source, lockfile, and fetched artifact
- Registry or mirror pulls that differ from expected origin
- Build scripts with preinstall, prepare, postinstall, or codegen steps
- Signed or packaged artifact hash compared to runtime-loaded artifact

## 常见误判

- Stopping at lockfile drift without proving runtime consumption
- Treating checked-in manifests as equivalent to live pipeline behavior
- Losing the earliest divergence point inside a wall of CI logs

来源：原 `CTF-Sandbox-Orchestrator/competition-supply-chain/references/supply-chain.md`。
