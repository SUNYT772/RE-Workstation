# Android Hooking Checklist

## Static Targets To Map First

- `AndroidManifest.xml`, exported components, deeplinks, intent filters, bundled configs
- Native libraries, JNI registration points, crypto helpers, request builders, protobuf models
- Shared prefs, SQLite DBs, WebView assets, root checks, SSL pinning logic

## Preferred Hook Boundaries

- Request signer input string and output signature
- Crypto helper plaintext and ciphertext
- JNI boundary arguments and return values
- Keystore access or device-binding checks
- WebView bridge messages or JS interface calls

## 关联证据

- Static location: class, method, symbol, or asset path
- Dynamic proof: hook log, returned value, request header, or accepted response
- State dependency: local token, DB row, pref key, nonce, or device flag

## 常见误判

- Hooking too high in the UI layer and missing the real signer boundary
- Capturing a signed header without the plaintext that produced it
- Ignoring local state prerequisites such as prefs, DB rows, or keystore material

来源：原 `CTF-Sandbox-Orchestrator/competition-android-hooking/references/android-hooking.md`。
