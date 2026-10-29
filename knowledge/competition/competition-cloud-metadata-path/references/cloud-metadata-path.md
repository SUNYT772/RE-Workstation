# Cloud Metadata Path Checklist

## First Pass

- Metadata endpoint, hop limit, required headers, session token requirements, link-local route, workload identity binding
- Reaching surface: local process, pod, container, proxy, SSRF, sidecar, or host namespace
- Downstream trust: role assumption, cloud API, cluster API, secret access, or signed identity use

## Chain To Reconstruct

- Reachable path to metadata established
- Metadata response or token obtained
- Usable identity or credential material extracted
- Downstream API or trust edge accepts it
- Resulting privilege or artifact confirmed

## 关联证据

- Reachability side: route, headers, namespace, container, SSRF primitive, or proxy path
- Identity side: role name, token claims, expiration, audience, issuer, account or project binding
- Acceptance side: API action, resource access, secret read, or spawned workload effect

## 常见误判

- Proving metadata access without proving a useful credential was actually issued
- Proving token issuance without showing which downstream API accepts it
- Mixing node identity and workload identity without showing which one actually drove the privilege edge

来源：原 `CTF-Sandbox-Orchestrator/competition-cloud-metadata-path/references/cloud-metadata-path.md`。
