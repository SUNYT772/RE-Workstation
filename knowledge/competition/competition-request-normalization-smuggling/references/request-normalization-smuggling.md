# Request Normalization Smuggling Checklist

## First Pass

- Front proxy, gateway, backend parser, internal service chain
- Path decode rules, slash and dot handling, case normalization, host derivation
- Header canonicalization, duplicate header behavior, CL/TE handling, chunk parsing

## Chain To Reconstruct

- Baseline request path captured
- Differential request crafted with one delta
- Parser or router divergence observed
- Unintended route or request body boundary reached
- Decisive effect reproduced

## 关联证据

- Request side: raw baseline and differential requests
- Hop side: each parser decision and route match per hop
- Effect side: hidden endpoint access, auth bypass branch, or state mutation

## 常见误判

- Changing multiple fields at once and losing root-cause attribution
- Looking only at frontend proxy logs without backend route evidence
- Reporting parser mismatch without reproducing final effect

来源：原 `CTF-Sandbox-Orchestrator/competition-request-normalization-smuggling/references/request-normalization-smuggling.md`。
