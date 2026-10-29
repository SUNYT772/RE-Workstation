# LSASS Ticket Material Checklist

## First Pass

- Logon sessions, LUIDs, package names, ticket caches, SSP artifacts, DPAPI context, account names
- Material types: plaintext, NTLM hash, TGT, service ticket, delegated ticket, DPAPI secret, gMSA material
- Acceptance candidates: SMB, WinRM, service ticket use, Schannel, DPAPI unwrap, service logon

## Chain To Reconstruct

- Host or memory artifact located
- Credential or ticket material extracted
- Replay or unwrap path identified
- Service or host accepts it
- Resulting logon, token, or privilege confirmed

## 关联证据

- Host side: process, dump or cache source, LUID, package, account
- Material side: type, ticket flags, SPN, encryption, DPAPI context, cache location
- Acceptance side: target service, target host, resulting session or privilege change

## 常见误判

- Treating every extracted secret as replayable without proving an accepting service
- Mixing several logon sessions and ticket caches into one vague story
- Describing ticket presence without tying it to a concrete privilege or pivot edge

来源：原 `CTF-Sandbox-Orchestrator/competition-lsass-ticket-material/references/lsass-ticket-material.md`。
