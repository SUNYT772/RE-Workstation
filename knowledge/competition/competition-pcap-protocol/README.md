# PCAP 与协议

适用问题：决定性证据依赖数据包顺序、协议分帧或流重组，需要超出单个 IOC 或主机日志分析。

## 主题与术语

PCAP、TCP/UDP 会话重组、HTTP/WebSocket/DNS、自定义 C2/二进制协议、beacon 时序与流量到进程的关联。

## 分析关注点

- Establish the capture boundaries first: hosts, time span, interfaces, missing packets, retransmits, and stream count.
- Group traffic into sessions before decoding payload semantics.
- Record protocol framing, sequence, timing, and transferred artifacts together instead of as isolated packets.
- Correlate packet evidence with host, malware, or app behavior only after the session is reconstructed.
- Reproduce the smallest decoded stream or transferred artifact that proves the challenge path.

## 技术关系与判断

以下条目说明对象关系、验证证据与判断边界，可按问题组合参照。

### Build The Session Map

- Identify endpoints, protocols, ports, TLS handshakes, DNS lookups, websocket upgrades, and long-lived streams.
- Note missing capture coverage, asymmetric routing, packet loss, or reassembly issues before drawing conclusions.
- Separate control channels, bulk transfers, keepalives, and noise.

### Decode The Protocol Boundary

- Reassemble TCP streams or UDP conversations before interpreting fields.
- Recover framing, message order, custom headers, binary fields, compression, encryption boundaries, and object transfers.
- Keep payload direction, timing, and session state aligned with each decoded message.

### Tie Packets To Behavior

- Show which packet sequence maps to which host event, malware branch, login flow, upload, exfiltration step, or command channel.
- Distinguish protocol recognition from artifact recovery: naming HTTP, DNS, or a custom C2 is not enough without decoded content or proven downstream effect.
- If the task becomes mostly a host timeline problem after decode, see related forensic timeline knowledge.

## 细分参考与关联

- 参考：[pcap-protocol.md](references/pcap-protocol.md)：会话、解码检查项与证据组织方式。
- If the hard part is a WebSocket or SSE handshake, subscription flow, realtime frames, or frame-driven state, see [WebSocket 与实时状态](../competition-websocket-runtime/README.md).
- If the hard part is a custom handshake, framing, checksum, sequence dependency, or deterministic replay harness, see [自定义协议状态与重现](../competition-custom-protocol-replay/README.md).

## 证据字段

- Stream IDs, endpoint pairs, packet ranges, timestamps, protocol framing, and object boundaries
- Decoded requests, responses, commands, transferred files, and the session that carried them
- The exact packet sequence or reconstructed stream that proves the challenge behavior

来源：原 `CTF-Sandbox-Orchestrator/competition-pcap-protocol/SKILL.md`。
