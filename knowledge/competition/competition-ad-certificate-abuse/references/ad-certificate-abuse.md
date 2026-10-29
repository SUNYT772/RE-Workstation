# AD Certificate Abuse Checklist

## 核心对象

- Enterprise CA configuration, issuance policy, enrollment permissions, manager approval, authorized signatures
- Template flags, EKUs, subject or SAN controls, enrollment agent settings, superseded templates
- PKINIT, smartcard logon, Schannel, certificate mapping, relay paths, accepting services

## Abuse Chain To Reconstruct

- Principal with enrollment or relay path identified
- Template or CA weakness established
- Certificate issued with exploitable identity material
- Service or logon path accepts the cert
- Resulting privilege or account effect confirmed

## 关联证据

- Template side: name, rights, EKUs, flags, SAN controls, issuance requirements
- Cert side: subject, SAN, serial, validity, issuer, thumbprint
- Acceptance side: PKINIT or service mapping, target account, resulting privilege

## 常见误判

- Stopping at template misconfiguration without proving cert issuance
- Proving issuance without proving where the cert is actually accepted
- Mixing certificate abuse with Kerberos delegation when the decisive edge is the template or mapping itself

来源：原 `CTF-Sandbox-Orchestrator/competition-ad-certificate-abuse/references/ad-certificate-abuse.md`。
