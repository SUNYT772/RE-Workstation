# Agent, Toolchain, Cloud, And Supply-Chain Checklist

## Agentic Path

- Map instruction layers, retrieval layers, memory layers, tool gates, auth material, and side effects
- Keep one compact evidence block for prompts, retrieved text, planner drift, tool arguments, and side effect
- Prove one minimal exploit chain before exploring variants

## Cloud And Container Path

- Compare checked-in manifests to live mounts, env, sidecars, and logs
- Map metadata services, registries, message buses, object stores, and IAM-like identities as control surfaces when evidence places them in the analysis path.
- Track build-time, deploy-time, and runtime separately

## Supply Chain

- Keep a compact provenance chain: source -> dependency resolution -> build -> package/sign -> publish -> runtime consumer
- Focus on version drift, registry pulls, generated artifacts, and final runtime hook points

## 常见误判

- Trusting a prompt string without runtime confirmation
- Treating checked-in manifests as deployment truth
- Missing the point where retrieved content becomes executable tool input

来源：原 `CTF-Sandbox-Orchestrator/competition-agent-cloud/references/agent-cloud.md`。
