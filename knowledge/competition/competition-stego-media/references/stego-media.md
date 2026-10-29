# Stego Media Checklist

## First Pass

- Verify magic bytes, headers, chunk or atom layout, dimensions, duration, sample rate, and metadata
- Check for appended data, malformed trailers, duplicate thumbnails, embedded archives, or sidecar files
- Record entropy shifts, unexpected padding, or container slack regions

## Candidate Channels

- Alpha or palette anomalies
- LSBs in image or audio samples
- Frame-order or delta anomalies in video
- Document object streams, attachments, or hidden layers
- Transform-domain residue, QR fragments, or watermark-style carriers

## 证据编排

- Correlate offsets, channels, observed extraction transforms, parameters, and recovered bytes in one evidence block
- Keep each extracted stage as a separate file
- Note clearly whether the result is plaintext, another container, compressed data, or ciphertext

## 常见误判

- Jumping into brute force before checking trailers and metadata
- Mixing several partial decode attempts without tracking which channel produced which bytes
- Treating a suspicious pattern as success without reproducing the hidden payload

来源：原 `CTF-Sandbox-Orchestrator/competition-stego-media/references/stego-media.md`。
