# iOS 运行时

适用问题：关键路径经过 iOS 的运行时信任边界，需要结合运行行为判断静态字符串与 plist 值的含义。

## 主题与术语

iOS IPA、Frida、Objective-C/Swift 方法、Keychain、plist、SSL pinning、URL scheme、deeplink/universal link 与请求签名。

## 分析关注点

- Preserve the original IPA, extracted bundle, and any decrypted or re-signed copy as separate artifacts.
- Start with `Info.plist`, entitlements, URL schemes, frameworks, Keychain usage, and local app storage before broad runtime hooks.
- Choose the narrowest runtime boundary that proves behavior: signer, trust evaluator, Keychain accessor, Objective-C or Swift method, or network request builder.
- Correlate static bundle evidence and live hook output before claiming the trust path is understood.
- Reproduce the accepted request, token, or gated branch from the smallest hook set.

## 技术关系与判断

以下条目说明对象关系、验证证据与判断边界，可按问题组合参照。

### Static iOS Triage

- Map bundle structure, `Info.plist`, entitlements, URL schemes, universal links, embedded frameworks, and app group paths.
- Record likely trust boundaries: request signers, device binding, certificate checks, jailbreak checks, Keychain access, or local cache loading.
- Note whether sensitive logic sits in Objective-C, Swift, embedded frameworks, or a bundled web surface.

### Hook The Runtime Boundary

- Prefer hooking request builders, crypto helpers, trust evaluators, Keychain reads, or Objective-C selectors instead of broad UI handlers.
- Record plaintext inputs, headers, nonces, signed strings, and outputs at the boundary that changes server acceptance.
- Patch or bypass pinning or environment checks only enough to expose the real request path.

### Replay The Accepted Path

- Rebuild the smallest stateful sequence: local token, device identifier, request body, signature, headers, and trust checks.
- Keep hook logs, bundle paths, plist keys, and local storage artifacts tied to the same session or account state.
- If the task becomes mostly about transform recovery instead of iOS runtime, see related crypto or mobile knowledge.

## 细分参考与关联

- 参考：[ios-runtime.md](references/ios-runtime.md)：Hook 观察边界、存储检查项与证据组织方式。

## 证据字段

- Bundle paths, entitlements, plist keys, selectors, class names, hook points, and header names
- Keychain items, local DB or plist paths, URL schemes, and app-group storage locations
- The smallest replayable request or branch that proves the iOS trust boundary

来源：原 `CTF-Sandbox-Orchestrator/competition-ios-runtime/SKILL.md`。
