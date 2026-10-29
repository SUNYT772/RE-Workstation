# AD 证书与身份映射

适用问题：证书是关键身份边界，需要判断模板或 CA 策略如何转化为服务端接受的权限。

## 主题与术语

AD CS、ESC 类问题、证书模板、注册权限、EKU、SAN 与主体控制、注册代理、PKINIT/智能卡登录、CA 策略与证书映射。

## 分析关注点

- Identify the CA, template, enrolling principal, and accepting service before diving into every certificate detail.
- Separate template enrollability from cert-based authentication or privilege acceptance.
- Record EKUs, subject or SAN controls, issuance requirements, enrollment rights, and mapping behavior in compact blocks.
- Tie the issued cert to one accepted path: PKINIT, Schannel, LDAPS, WinRM, or another mapped service.
- Reproduce the smallest certificate issuance-to-acceptance chain that yields the decisive privilege.

## 技术关系与判断

以下条目说明对象关系、验证证据与判断边界，可按问题组合参照。

### Map CA And Template Trust

- Record CA configuration, template name, enrollment permissions, manager approval, authorized signatures, EKUs, subject requirements, and SAN behavior.
- Note whether the path depends on alternate subject names, `UPN`, DNS names, enrollment agent behavior, or template supersedence.
- Keep principal, template, and issuance policy tied together.

### Prove Cert-To-Privilege Acceptance

- Show how the issued certificate is mapped or accepted: PKINIT, smartcard logon, Schannel auth, service mapping, or explicit certificate mapping.
- Record serial, subject, SAN, EKU, validity, and the exact service or domain edge that accepts it.
- Distinguish certificate issuance from the separate step where privilege is actually granted.

### Reduce To The Decisive Abuse Chain

- Compress the path to the smallest sequence: enrollment right or misconfig -> issued cert -> accepted mapping -> resulting privilege.
- State clearly whether the weakness lives in template config, CA policy, mapping logic, relay path, or enrollment rights.
- If the task is really about delegation or ticket transformation after PKINIT, see related Kerberos knowledge.

## 细分参考与关联

- 参考：[ad-certificate-abuse.md](references/ad-certificate-abuse.md)：AD CS、证书模板检查项与证据组织方式。

## 证据字段

- CA names, template names, rights, EKUs, issuance flags, SAN controls, and mapping details
- Issued certificate fields, serials, subjects, SANs, and the accepting service or logon path
- The smallest reproducible enrollment-to-privilege chain

来源：原 `CTF-Sandbox-Orchestrator/competition-ad-certificate-abuse/SKILL.md`。
