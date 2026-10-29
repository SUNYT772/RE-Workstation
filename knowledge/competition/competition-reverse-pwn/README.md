# 逆向与 Pwn

适用问题：问题以二进制为主，关键路径涉及样本、解码层、进程行为、崩溃状态或利用原语。

## 主题与术语

原生逆向、恶意代码、DFIR、固件、Pwn、内存/PCAP、崩溃状态、解码层与利用原语。

## 分析关注点

- Preserve the original artifact before unpacking, patching, or instrumenting.
- Start with passive triage: type, headers, sections, imports, strings, entropy, resources.
- Decide whether the path is reverse-first, DFIR-first, or exploit-first.
- Tie every claim to an observable boundary: decode edge, persistence edge, crash edge, or leak edge.
- Reproduce the artifact or primitive from a clean baseline.

## 技术关系与判断

以下条目说明对象关系、验证证据与判断边界，可按问题组合参照。

### Reverse Or Forensic Triage

- Separate loader, payload, config, and post-decode behavior.
- Correlate files, memory, logs, registry, services, tasks, IPC, and PCAPs as one graph.
- Keep decoded or dumped artifacts separate from the pristine sample.

### Native And Exploit Path

- Map mitigations, loader behavior, libc or runtime, syscall and IPC surfaces, and protocol framing.
- Record the primitive, controllable bytes, leak source, target object, and final artifact separately.
- Compare host, libc, loader, and framing differences before doubting the primitive.

## 细分参考与关联

- 参考：[reverse-pwn.md](references/reverse-pwn.md)：初步分类顺序、原语与利用证据要求、常见失败原因。
- If the task is specifically about staged payload boundaries, config blobs, beacon parameters, or decoded IOC fields, see [恶意代码配置](../competition-malware-config/README.md).
- If the task is specifically about firmware partitions, boot chains, extracted filesystems, or update-package trust boundaries, see [固件布局](../competition-firmware-layout/README.md).
- If the task is specifically about upload parsing, previews, archive extraction, converters, or deserialization chains, see [文件解析链](../competition-file-parser-chain/README.md).
- If the task is specifically about source maps, emitted bundles, chunk registries, or reconstructing hidden runtime structure from served frontend assets, see [构建产物与源码映射](../competition-bundle-sourcemap-recovery/README.md).
- If the task is specifically about container-to-host boundary crossing, kernel exploit preconditions, namespace or cgroup crossover, or escape primitive verification, see [内核与容器边界](../competition-kernel-container-escape/README.md).
- If the task is specifically about reconstructing protocols, streams, or transferred artifacts from packet captures, see [PCAP 与协议](../competition-pcap-protocol/README.md).
- If the task is specifically about a custom binary or text protocol where replay state, message order, or checksum logic is the real blocker, see [自定义协议状态与重现](../competition-custom-protocol-replay/README.md).
- If the task is specifically about reconstructing chronology across EVTX, PCAP, registry, mail, or disk artifacts, see [取证时间线](../competition-forensic-timeline/README.md).

## 证据字段

- Offsets, hashes, section names, imports, config blobs, mutexes, registry keys
- Crash offsets, registers, heap or stack shape, leak addresses, and protocol steps
- Original, decoded, dumped, and instrumented artifacts as separate files

来源：原 `CTF-Sandbox-Orchestrator/competition-reverse-pwn/SKILL.md`。
