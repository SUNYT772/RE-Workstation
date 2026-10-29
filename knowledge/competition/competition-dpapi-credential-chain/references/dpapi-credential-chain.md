# DPAPI Credential Chain Checklist

## First Pass

- Blob source, masterkey path, SID, protector scope, profile path, store name, wrapper metadata
- Secret locations: Credential Manager, Vault, browser Login Data, cookies, Wi-Fi profiles, app storage
- Acceptance targets: browser session replay, SMB, RDP, WinRM, app login, data decryption

## Chain To Reconstruct

- Protected artifact is identified
- Matching masterkey or unwrap context is found
- Plaintext secret is recovered
- Accepting service or dataset is confirmed
- Resulting capability or access is reproduced

## 关联证据

- Source side: blob path, store, SID, profile, machine or user scope
- Unwrap side: masterkey source, backup key use, wrapper details, plaintext type
- Acceptance side: target service, target host, unlocked data, resulting session or privilege

## 常见误判

- Treating DPAPI decryption as the end instead of proving an accepting service or dataset
- Mixing user and machine scope without recording which protector actually matches
- Ignoring application-specific wrapping around browser or vault secrets

来源：原 `CTF-Sandbox-Orchestrator/competition-dpapi-credential-chain/references/dpapi-credential-chain.md`。
