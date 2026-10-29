# Identity, Windows, And Enterprise Messaging Checklist

## Identity

- Map principal origin, token or ticket minting, claims or group transformation, and final accepting service
- Distinguish credential material from effective privilege
- Inspect ACLs, GPO links, SIDHistory, delegation, certificate templates, service accounts, and replication rights when AD edges matter

## Windows Host

- Check SAM, SECURITY, SYSTEM, NTDS, DPAPI, LSA secrets, browser stores, PowerShell history, ETW, Sysmon, prefetch, jump lists, Amcache, SRUM, shimcache
- Correlate services, tasks, WMI, WinRM, SMB, RDP, admin shares, and remote registry as one pivot graph

## Enterprise Messaging

- Correlate phishing lures, attachment chains, consent logs, login traces, message-trace logs, and mailbox-rule changes

## 证据字段

- One compact block for SIDs, SPNs, ticket fields, event IDs, logon IDs, or mailbox rules
- One compact block for host pivots, replayed artifacts, and resulting privilege changes

## 常见误判

- Treating possession of a hash or ticket as proof of resulting privilege
- Saying "domain compromise" without a reproducible edge-by-edge chain
- Separating host evidence from identity evidence until the story stops making sense

来源：原 `CTF-Sandbox-Orchestrator/competition-identity-windows/references/identity-windows.md`。
