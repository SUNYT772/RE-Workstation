# Custom Protocol Replay Checklist

## Transcript First

- Establish roles, stream IDs, ports, session resets, handshake boundaries, and successful transcript examples
- Separate transport segmentation from application messages
- Record retransmits or duplicate messages so they do not pollute the protocol model

## Recovery Order

- Framing and message boundaries
- Direction and sequence state
- Integrity fields: checksum, MAC, counter, nonce, or signature
- Compression, encoding, or crypto boundary
- Accepted vs rejected transcript deltas

## 关联证据

- Message identity: type, offset, length, direction, sequence, or delimiter
- Acceptance state: prior message dependency, negotiated value, checksum, or nonce source
- Replay proof: minimal transcript, harness input, and server response or side effect

## 常见误判

- Trying broad replay before framing and state dependencies are understood
- Mixing messages from separate sessions into one replay model
- Claiming protocol recovery without producing an accepted replay or meaningful state transition

来源：原 `CTF-Sandbox-Orchestrator/competition-custom-protocol-replay/references/custom-protocol-replay.md`。
