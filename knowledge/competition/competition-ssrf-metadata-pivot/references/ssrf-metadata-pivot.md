# SSRF Metadata Pivot Checklist

## First Pass

- SSRF source parameter, URL builder, method, headers, redirect handling
- Host allowlist or denylist logic, DNS behavior, scheme checks, proxy layers
- Reachable internal services, metadata endpoints, token responses

## Chain To Reconstruct

- Server-side fetch primitive is confirmed
- Internal or metadata endpoint is reachable
- Credential or sensitive response is extracted
- Downstream service accepts the recovered material
- Resulting capability is reproduced

## 关联证据

- Source side: endpoint, parameter, URL construction, normalization rules
- Pivot side: target host, metadata path, token fields, scope, expiry
- Acceptance side: target API or service, replay method, resulting privilege

## 常见误判

- Treating metadata reachability as equivalent to privilege without acceptance proof
- Ignoring redirects, host normalization, or proxy-injected headers
- Skipping the final accepted-access step and reporting only response leakage

来源：原 `CTF-Sandbox-Orchestrator/competition-ssrf-metadata-pivot/references/ssrf-metadata-pivot.md`。
