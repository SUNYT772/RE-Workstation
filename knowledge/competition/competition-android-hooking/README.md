# Android 运行时

适用问题：关键路径经过 Android 应用的运行时信任边界，需要结合运行行为判断静态字符串的含义。

## 主题与术语

Android APK、Frida、请求签名恢复、SSL pinning、root 检查、Java/JNI/native 边界、共享偏好与应用数据库。

## 分析关注点

- Preserve the original APK, extracted resources, and decompiled output before patching or resigning.
- Start with manifest, exported components, deeplinks, native libs, prefs, local DBs, and bundled configs.
- Decide the narrowest runtime boundary to hook: signer, crypto helper, JNI bridge, WebView bridge, or request builder.
- Correlate static evidence and dynamic traces before claiming a trust edge is understood.
- Reproduce the signed request, accepted token, or gated branch from the smallest hook set.

## 技术关系与判断

以下条目说明对象关系、验证证据与判断边界，可按问题组合参照。

### Static Triage Before Hooks

- Map package structure, exported activities, services, receivers, providers, and deeplink handlers.
- Note SSL pinning logic, root checks, feature flags, token storage, shared prefs, SQLite tables, and protobuf or RPC boundaries.
- Identify whether the sensitive logic sits in Java, Kotlin, JNI, or a bundled WebView.

### Hook The Narrowest Boundary

- Prefer hooking request signers, crypto helpers, keystore access, protobuf encode or decode, or JNI marshaling instead of broad UI hooks.
- Record plaintext inputs, signed strings, headers, nonces, and outputs at the boundary that actually changes trust.
- If pinning or environment checks block progress, patch or hook only enough to expose the real request path.

### Replay The Accepted Path

- Rebuild the smallest sequence that reaches the accepted server-side branch: local state, nonce, request body, signature, and headers.
- Keep hook logs, captured request shapes, and local storage paths tied to the same account or session state.
- If the challenge becomes more about transform recovery than Android runtime, see related crypto or mobile knowledge.

## 细分参考与关联

- 参考：[android-hooking.md](references/android-hooking.md)：Hook 观察边界、存储检查项与证据组织方式。

## 证据字段

- Hook points, class names, JNI symbols, signer inputs and outputs, and header names
- Shared prefs, local DB rows, deeplinks, exported components, and token storage paths
- The smallest replayable request or branch that proves the trust boundary

来源：原 `CTF-Sandbox-Orchestrator/competition-android-hooking/SKILL.md`。
