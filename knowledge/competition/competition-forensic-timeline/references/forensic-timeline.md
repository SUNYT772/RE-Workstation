# Forensic Timeline Checklist

## Anchor Selection

- Earliest trustworthy markers: process creation, service install, scheduled task, mailbox rule write, browser download, network connect, or credential replay
- Note the time source for each artifact: event log local time, UTC network time, filesystem timestamp, mailbox server time, or packet capture timestamp
- Record drift, missing zones, daylight-saving ambiguity, or truncation before merging sources

## Cross-Source Correlation

Match on shared identifiers whenever possible:

- Process ID, parent process ID, or command line
- Logon ID, SID, ticket cache, mailbox item ID, or message trace ID
- Hostname, IP, MAC, session ID, GUID, or hash
- File path, registry path, service name, task name, or attachment name

## Timeline Compression

- A raw chronology preserves the event record; a short decisive chronology presents the sequence supporting the finding
- Separate observed events from inferred transitions
- If an edge is inferred, state the missing validation step explicitly

## 常见误判

- Sorting by timestamp alone when clock drift or delayed logging exists
- Mixing mailbox, browser, host, and network artifacts without shared identifiers
- Reporting a long event dump instead of the smallest sequence that proves the challenge path

来源：原 `CTF-Sandbox-Orchestrator/competition-forensic-timeline/references/forensic-timeline.md`。
