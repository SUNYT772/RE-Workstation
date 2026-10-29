# Mailbox Abuse Checklist

## 核心对象

- Consent grants, app registrations, delegated or application scopes, sign-in events
- Inbox rules, forwarding settings, delegate access, shared mailbox permissions, transport rules
- Message traces, mailbox audit events, sender or recipient pairs, message IDs, folder actions

## Abuse Chain To Reconstruct

- Lure, consent, or credential acquisition
- Token or delegate edge obtained
- Mailbox or transport mutation applied
- Message flow altered, hidden, or exfiltrated

## 关联证据

- Identity side: user, app, scope, token claim, delegate edge
- Mail side: rule predicate, forward target, folder action, message ID, transport action
- Result side: delivered, forwarded, deleted, rerouted, or silently hidden effect

## 常见误判

- Reporting a token without showing the mailbox effect it actually enables
- Mixing inbox rules and transport rules without proving which one caused the observed message flow
- Ignoring shared mailbox or delegate permissions when the mailbox owner itself never logged in

来源：原 `CTF-Sandbox-Orchestrator/competition-mailbox-abuse/references/mailbox-abuse.md`。
