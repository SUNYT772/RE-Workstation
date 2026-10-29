# iOS Runtime Checklist

## Static Targets To Map First

- `Info.plist`, entitlements, URL schemes, universal links, embedded frameworks, provisioning clues
- Keychain access groups, app groups, local plist files, SQLite DBs, cache directories
- Certificate checks, jailbreak checks, request builders, crypto helpers, device-binding logic

## Preferred Hook Boundaries

- Request builder input and final signed headers
- Crypto helper plaintext and ciphertext
- Trust evaluator or pinning decision point
- Keychain read or write boundary
- Objective-C selector or Swift method that gates the accepted branch

## 关联证据

- Static location: class, selector, framework, plist key, entitlement, or bundle path
- Dynamic proof: hook log, returned value, header, request body, or accepted response
- State dependency: Keychain item, plist value, DB row, nonce, device flag, or local token

## 常见误判

- Hooking only UI handlers and missing the real signer or trust evaluator
- Capturing a signed request without the plaintext or local state that generated it
- Mixing original IPA, decrypted bundle, and patched runtime output without labeling them separately

来源：原 `CTF-Sandbox-Orchestrator/competition-ios-runtime/references/ios-runtime.md`。
