# 媒体隐写

适用问题：问题位于媒体容器、隐藏通道或追加载荷中，需要结合媒体结构分析。

## 主题与术语

图像/音频/视频/文档隐写、元数据、alpha/调色板、LSB、缩略图、追加尾部、QR 片段与转码痕迹。

## 分析关注点

- Confirm the real container type, dimensions, duration, codec, and chunk layout before guessing a hidden layer.
- Check metadata, thumbnails, sidecar files, and appended trailers before deeper signal-domain work.
- Rank candidate channels by evidence: alpha, palette, LSB, transform-domain residue, frame order, or container slack.
- Preserve each extracted layer separately so the transform chain stays reproducible.
- Stop when the hidden payload is reproduced, not merely suspected.

## 技术关系与判断

以下条目说明对象关系、验证证据与判断边界，可按问题组合参照。

### Establish Container Truth

- Inspect headers, chunk tables, EXIF or document metadata, container indexes, thumbnails, and file size anomalies.
- Compare declared format against observed structure to catch polyglots, appended archives, or malformed trailers.
- Record exact offsets, frame numbers, or channel boundaries that look promising.

### Inspect Candidate Channels

- Check alpha, palette order, RGB or YUV planes, LSBs, spectrogram features, document object streams, or video frame deltas.
- Prefer evidence-driven attempts over brute forcing every transform.
- Note whether the payload is plain bytes, another media layer, compressed data, or an encrypted blob.

### Reconstruct The Hidden Payload Path

- Keep the chain in order: container -> channel or carrier -> extraction -> decompression or decode -> final parse.
- Separate extraction success from final interpretation; a channel hit is not the same as artifact recovery.
- If the problem becomes primarily about cryptography after extraction, see related crypto knowledge.

## 细分参考与关联

- 参考：[stego-media.md](references/stego-media.md)：媒体检查项、通道优先级判断与证据组织方式。

## 证据字段

- File structure facts: offsets, chunks, frame numbers, stream names, metadata keys, and trailer size
- Intermediate extraction artifacts, observed tool or transform identity, exact parameters, and resulting bytes
- The final recovered payload and the channel that produced it

来源：原 `CTF-Sandbox-Orchestrator/competition-stego-media/SKILL.md`。
