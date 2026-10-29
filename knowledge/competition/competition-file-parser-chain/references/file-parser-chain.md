# File Parser Chain Checklist

## First Pass

- Upload request shape, field names, content type, filename, extension, magic bytes, temp storage
- Archive members, conversion outputs, previews, thumbnails, extracted docs, serialized objects
- Final consumers: parser, converter, renderer, importer, deserializer, worker

## Chain To Reconstruct

- File ingress accepted
- Temp or staged artifact created
- Extraction or conversion performed
- Parser or deserializer invoked
- Business-logic effect or artifact produced

## 关联证据

- Ingress side: request, filename, MIME, temp path, storage key
- Parser side: tool or library invoked, branch condition, derived artifact, parser choice
- Effect side: rendered output, parsed object, backend branch, worker task, or privilege-bearing result

## 常见误判

- Looking only at the original upload and ignoring derived intermediates
- Treating MIME or extension checks as proof of backend parser choice
- Mixing archive, preview, and deserialization stages without preserving each boundary separately

来源：原 `CTF-Sandbox-Orchestrator/competition-file-parser-chain/references/file-parser-chain.md`。
