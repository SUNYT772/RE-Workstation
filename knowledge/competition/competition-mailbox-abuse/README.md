# 邮件与委托关系

适用问题：关键路径涉及邮箱行为、同意授权流程或消息路由效果，需要结合具体邮件证据判断。

## 主题与术语

企业邮件、OAuth 同意授权、收件箱/转发/传输规则、共享邮箱、委托访问、钓鱼链与消息追踪。

## 分析关注点

- Decide whether the active path is phishing-to-consent, token-to-mailbox, rule-based persistence, or transport-level mail rerouting.
- Keep mailbox evidence, identity evidence, and message-trace evidence tied to the same user, mailbox, token, or message ID.
- Separate possession of a token or delegate edge from the actual mailbox effect it enables.
- Record forwarding targets, rule predicates, consent scopes, shared mailbox edges, and resulting mail flow in compact blocks.
- Reproduce the smallest mail effect that proves persistence, exfiltration, or privilege.

## 技术关系与判断

以下条目说明对象关系、验证证据与判断边界，可按问题组合参照。

### Map The Mail Trust Path

- Identify the principal, mailbox, token or session, consent grant, delegate edge, shared mailbox relationship, or app registration involved.
- Record consent scopes, mailbox permissions, rule ownership, transport actions, and message-trace identifiers.
- Distinguish client-visible symptoms from server-side mailbox or transport state.

### Prove The Mailbox Effect

- Correlate consent logs, sign-ins, message traces, inbox rules, transport rules, forwarding settings, and mailbox audit events.
- Show which rule or token produces which concrete effect: silent forwarding, marking read, deletion, delegate access, or message rerouting.
- Keep message IDs, sender or recipient pairs, and timestamps aligned across logs.

### Reduce To The Decisive Abuse Chain

- Compress the path to the smallest sequence: lure or grant -> token or delegate edge -> mailbox or transport mutation -> resulting mail effect.
- State clearly whether persistence lives in consent, mailbox rules, transport config, or shared mailbox permissions.
- If the task broadens into host pivots or Kerberos acceptance, see related identity knowledge.

## 细分参考与关联

- 参考：[mailbox-abuse.md](references/mailbox-abuse.md)：同意授权、邮箱规则检查项与证据组织方式。

## 证据字段

- Consent scopes, token claims, mailbox permissions, rule definitions, forwarding targets, and message IDs
- Message-trace lines, audit events, and mailbox effects tied to the same mail path
- The smallest replayable sequence that proves persistence, exfiltration, or delegate access

来源：原 `CTF-Sandbox-Orchestrator/competition-mailbox-abuse/SKILL.md`。
