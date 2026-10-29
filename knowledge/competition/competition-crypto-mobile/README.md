# 密码与移动

适用问题：问题依赖变换链还原、媒体隐藏载荷、移动端签名路径或本地信任边界。

## 主题与术语

密码、编码、隐写、APK/IPA、变换链与密钥还原、移动端请求签名和本地信任边界。

## 分析关注点

- Decide whether the dominant path is crypto, stego, or mobile.
- Recover transforms in order; do not jump straight to the fanciest algorithm.
- Record exact parameters and boundaries that affect the result.
- Hook the narrowest mobile boundary that proves the behavior.
- Reproduce the plaintext, payload, signed request, or accepted branch.

## 技术关系与判断

以下条目说明对象关系、验证证据与判断边界，可按问题组合参照。

### Crypto And Encoding

- Reconstruct the chain step by step: container, compression, encoding, xor or substitution, crypto, integrity, final parse.
- Keep exact keys, IVs, nonces, salts, tags, offsets, and byte order.

### Stego

- Inspect metadata, chunk layout, palettes, alpha planes, LSBs, thumbnails, trailers, and transcoding artifacts.
- Rank decode attempts by evidence, not by brute-force curiosity.

### Mobile

- Start with manifest or plist, exported components, deeplinks, native libs, shared prefs, local DBs, and configs.
- Trace signer logic, token storage, SSL pinning, protobuf or RPC boundaries, and native bridge calls.

## 细分参考与关联

- 参考：[crypto-mobile.md](references/crypto-mobile.md)：变换链检查项、Hook 观察边界与证据组织方式。
- If the task is specifically about Android dynamic tracing, signer hooks, JNI boundaries, or pinning checks, see [Android 运行时](../competition-android-hooking/README.md).
- If the task is specifically about iOS runtime tracing, Keychain access, Objective-C or Swift hooks, or pinning checks inside an IPA, see [iOS 运行时](../competition-ios-runtime/README.md).
- If the task is specifically about media carriers, hidden channels, thumbnails, or appended trailers, see [媒体隐写](../competition-stego-media/README.md).

## 证据字段

- Decisive bytes proving each decode stage
- Hook points, signed strings, headers, and local storage paths
- Component names, protobuf fields, channel-specific outputs, or trailer offsets

来源：原 `CTF-Sandbox-Orchestrator/competition-crypto-mobile/SKILL.md`。
