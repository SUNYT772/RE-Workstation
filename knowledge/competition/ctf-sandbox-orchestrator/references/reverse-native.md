# Reverse, Malware, DFIR, And Native

Use this reference for binaries, shellcode, firmware, documents with embedded payloads, memory dumps, PCAPs, malware configs, pwnables, and native exploit chains.

## Reverse And DFIR Workflow

- Start with passive triage: file type, headers, sections, imports, exports, strings, entropy, resources, and embedded archives.
- Preserve original artifacts; derive unpacked, decrypted, or instrumented layers separately.
- Correlate process trees, filesystem writes, registry changes, services, tasks, IPC, and network beacons as one execution graph.
- Explain where each recovered value comes from and when it becomes plaintext.

## Native And Pwn Workflow

- Map mitigations, loader behavior, libc/runtime, syscall surface, IPC surface, and protocol framing.
- Separate primitive from proof: controllable bytes, leak source, target object, mitigation bypass, and final artifact.
- Tie crash offsets, register state, allocator layout, and protocol steps to observable state transitions.
- Keep PoCs, debugger scripts, core dumps, and recovered symbols separate from the pristine path.

## High-Value Evidence

- Offsets, section names, import tables, string-decode helpers, config blobs
- Mutexes, named pipes, startup keys, services, tasks, WMI, persistence traces
- Packet captures matched against process or config evidence
- Leak addresses, heap layout, stack shape, allocator metadata, accepted ticket or hook target

## 常见误判

- Treating one string hit as proof of the full behavior
- Mixing original and modified artifacts
- Skipping the boundary between loader, payload, config, and post-decode behavior
- Assuming the primitive is wrong before comparing libc, loader, or framing differences

来源：原 `CTF-Sandbox-Orchestrator/ctf-sandbox-orchestrator/references/reverse-native.md`。
