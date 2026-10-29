# 取证时间线

适用问题：需要把多个来源的取证痕迹关联成可复核的时间顺序与执行链。

## 主题与术语

DFIR、跨痕迹关联、持久化链、EVTX、PCAP、注册表、磁盘、内存、邮箱与浏览器时间线。

## 分析关注点

- Pick the smallest reliable anchor: first execution, first logon, first network session, first file write, or first mailbox action.
- Normalize timestamps, time zones, hostnames, users, process IDs, message IDs, and file paths before correlating.
- Build one minimal chain from foothold to persistence, execution, access, or exfiltration.
- Separate confirmed event order from inferred gaps.
- Reproduce the decisive timeline segment that yields the artifact or privilege conclusion.

## 技术关系与判断

以下条目说明对象关系、验证证据与判断边界，可按问题组合参照。

### Establish Timeline Anchors

- Collect only the active surfaces: EVTX, Sysmon, registry, Amcache, prefetch, browser artifacts, mail traces, PCAPs, memory, or filesystem metadata.
- Record clock source, timezone, and any drift or truncation that could reorder events.
- Link shared identifiers across sources: PID, logon ID, GUID, message ID, hostname, username, IP, or hash.

### Correlate The Execution Graph

- Track process tree, service or task creation, network sessions, file writes, registry changes, mailbox rules, or token use as one path.
- Distinguish causal edges from coincidence by matching identifiers and adjacency, not just nearby timestamps.
- Keep raw artifact and parsed summary side by side so every step can be traced back.

### Compress To The Decisive Story

- Reduce the timeline to the smallest sequence that proves initial access, persistence, lateral movement, collection, or artifact recovery.
- Call out missing validation steps separately instead of mixing them into confirmed chronology.
- If the task becomes mainly about malware config extraction or a Windows pivot edge, see related specialized knowledge.

## 细分参考与关联

- 参考：[forensic-timeline.md](references/forensic-timeline.md)：时间锚点选择、跨来源关联与证据组织方式。
- If the hard part is packet reassembly, protocol framing, or transferred-object extraction from a capture, see [PCAP 与协议](../competition-pcap-protocol/README.md).

## 证据字段

- Source file paths, event IDs, logon IDs, message IDs, PIDs, hashes, and timestamps with timezone noted
- One compact timeline table or ordered list for the decisive segment
- Raw artifacts, parsed output, and inferred edges kept separate

来源：原 `CTF-Sandbox-Orchestrator/competition-forensic-timeline/SKILL.md`。
