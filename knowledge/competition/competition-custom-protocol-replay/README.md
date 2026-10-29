# 自定义协议状态与重现

适用问题：需要还原协议接受消息所依赖的准确顺序与会话状态，而不只是识别协议名称。

## 主题与术语

自定义二进制/文本协议、握手、分帧、序列控制、校验和、会话状态与消息顺序重现。

## 分析关注点

- Identify client and server roles, session boundaries, and reset conditions before decoding field semantics.
- Recover framing, lengths, delimiters, sequence numbers, checksums, nonces, and state transitions before broad replay attempts.
- Keep one canonical transcript of a successful exchange.
- Change one field or one message at a time while replaying.
- Reproduce the smallest accepted conversation that proves the decisive branch.

## 技术关系与判断

以下条目说明对象关系、验证证据与判断边界，可按问题组合参照。

### Map The Session State Machine

- Identify handshake, negotiation, authentication, keepalive, command, and teardown phases.
- Record which fields are static, which are derived, and which depend on prior messages.
- Keep message order, direction, and timing tied to the same session identity.

### Recover Framing And Integrity

- Reconstruct lengths, delimiters, type bytes, checksums, MACs, counters, compression, or encryption boundaries.
- Distinguish transport framing from application-level framing.
- Note exactly where server acceptance changes when one field or step is mutated.

### Build The Minimal Replay Harness

- Reduce the path to the smallest transcript that reaches the accepted state, parser branch, command effect, or artifact.
- Preserve both the original captured sequence and the replayed minimal sequence.
- If the problem is mainly generic PCAP or stream decoding with no stateful replay requirement, see related PCAP knowledge.

## 细分参考与关联

- 参考：[custom-protocol-replay.md](references/custom-protocol-replay.md)：状态机、会话记录检查项与证据组织方式。

## 证据字段

- Canonical transcript, message types, field boundaries, checksums, counters, and session identifiers
- Original capture slices and the replay harness inputs that produce acceptance
- The exact mutation that flips the protocol from rejected to accepted, or vice versa

来源：原 `CTF-Sandbox-Orchestrator/competition-custom-protocol-replay/SKILL.md`。
