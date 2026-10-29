# Crypto, Stego, And Mobile

Use this reference for ciphertexts, encoded blobs, steganography, APK/IPA analysis, request-signing chains, device state, and mobile trust-boundary work.

## Crypto And Encoding

Recover the transform chain in order:

- Container or serialization
- Compression or chunking
- Encoding or substitution
- XOR or custom transforms
- Cryptography
- Integrity or final parse

Record exact parameters whenever they matter: keys, IVs, nonces, salts, tags, padding rules, alphabets, offsets, and byte order.

## Steganography

Inspect metadata, chunk layout, palettes, alpha planes, LSBs, thumbnails, appended trailers, and transcoding artifacts before assuming the payload is absent.

## Mobile

- Start with manifest/plist structure, exported components, deeplinks, native libs, bundled web assets, local DBs, shared prefs, and configs.
- Map how trust and state are derived: token storage, SSL pinning, device checks, feature flags, protobuf/RPC boundaries, and local caches.
- When hooking is required, instrument the narrowest boundary that proves the behavior: signer, crypto helper, protobuf edge, native bridge, keystore access, or WebView message channel.

## 证据字段

- Decode stages and the bytes proving each stage
- Magic bytes, trailer offsets, channel-specific outputs, or recovered payload names
- Hook points, function names, signed strings, request headers, and local storage paths

## 常见误判

- Naming an algorithm without reproducing the actual plaintext or downstream artifact
- Brute-forcing every transform with no evidence ordering
- Hooking too late and missing the trust boundary that matters

来源：原 `CTF-Sandbox-Orchestrator/ctf-sandbox-orchestrator/references/crypto-mobile.md`。
