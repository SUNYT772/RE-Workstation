# CTF 专题与证据匹配矩阵

## 按主导证据检索

### Web 与运行状态

- General site, API, auth, session, upload, worker, or hidden route behavior -> [Web 与接口运行](../../competition-web-runtime/README.md)
- Browser storage, service workers, Cache Storage, IndexedDB -> [浏览器持久状态](../../competition-browser-persistence/README.md)
- WebSocket or SSE handshake, subscriptions, reconnect logic, realtime frames -> [WebSocket 与实时状态](../../competition-websocket-runtime/README.md)
- Host headers, vhost routing, reverse proxies, route-to-service resolution -> [运行路由](../../competition-runtime-routing/README.md)
- SSR loaders, templates, hydration payloads, render-layer enforcement drift -> [模板与渲染路径](../../competition-template-render-path/README.md)
- Source maps, build manifests, emitted bundles, chunk loading, runtime recovery -> [构建产物与源码映射](../../competition-bundle-sourcemap-recovery/README.md)
- GraphQL schema, persisted query, RPC manifest, generated client, contract drift -> [GraphQL / RPC 契约差异](../../competition-graphql-rpc-drift/README.md)
- OAuth or OIDC redirect, callback, PKCE, claim acceptance -> [OAuth / OIDC 信任链](../../competition-oauth-oidc-chain/README.md)
- JWT header, `kid`, `alg`, issuer, audience, role or claim confusion -> [JWT 声明混淆](../../competition-jwt-claim-confusion/README.md)
- Upload parsing, archive extraction, preview conversion, deserialization -> [文件解析链](../../competition-file-parser-chain/README.md)
- Queue payloads, retries, cron drift, worker-only behavior -> [队列与后台处理差异](../../competition-queue-worker-drift/README.md)
- SSRF reachability, internal fetch pivots, metadata token extraction, token acceptance -> [服务端请求与元数据关系](../../competition-ssrf-metadata-pivot/README.md)
- Race windows, ordering bugs, idempotency failures, timing-dependent state drift -> [并发与状态差异](../../competition-race-condition-state-drift/README.md)
- Proxy/backend parse differentials, path normalization drift, framing ambiguity, smuggling -> [请求归一化与解析差异](../../competition-request-normalization-smuggling/README.md)

### 逆向、原生与取证

- Binary triage, exploit primitives, crash behavior, native reversing -> [逆向与 Pwn](../../competition-reverse-pwn/README.md)
- Malware config, staged payload, beacon parameters, IOC decoding -> [恶意代码配置](../../competition-malware-config/README.md)
- Firmware partitions, boot chains, update packages, extracted filesystem -> [固件布局](../../competition-firmware-layout/README.md)
- Packet captures, protocol reconstruction, stream rebuilding -> [PCAP 与协议](../../competition-pcap-protocol/README.md)
- Custom protocol replay, state machine, checksums, accepted session reproduction -> [自定义协议状态与重现](../../competition-custom-protocol-replay/README.md)
- Timeline reconstruction across disk, logs, PCAP, registry, mailbox -> [取证时间线](../../competition-forensic-timeline/README.md)

### 密码、隐写与移动

- Encoding chain, crypto boundary, stego media, mobile trust boundary -> [密码与移动](../../competition-crypto-mobile/README.md)
- Image, audio, video, document, trailer, metadata stego -> [媒体隐写](../../competition-stego-media/README.md)
- Android APK, Frida, signer logic, SSL pinning, native bridge -> [Android 运行时](../../competition-android-hooking/README.md)
- iOS IPA, Objective-C or Swift runtime, Keychain, pinning, deeplink -> [iOS 运行时](../../competition-ios-runtime/README.md)

### agent、云、容器与供应链

- Prompt injection, retrieval poisoning, planner drift, tool misuse -> [提示注入](../../competition-prompt-injection/README.md)
- Agent, cloud, CI/CD, container, supply-chain as a broad cluster -> [agent 与云](../../competition-agent-cloud/README.md)
- Image layers, sidecars, mounts, runtime-vs-manifest mismatch -> [容器运行状态](../../competition-container-runtime/README.md)
- Registry provenance, dependency drift, build or release tampering -> [供应链](../../competition-supply-chain/README.md)
- K8s API, service account, RBAC, admission/controller behavior -> [Kubernetes 控制面](../../competition-k8s-control-plane/README.md)
- Metadata service, workload identity, link-local credential path -> [云元数据与工作负载身份](../../competition-cloud-metadata-path/README.md)
- Container-to-host boundary crossing, kernel preconditions, namespace or cgroup escape proof -> [内核与容器边界](../../competition-kernel-container-escape/README.md)

### 身份与 Windows

- Identity chain, Windows host evidence, enterprise messaging, mixed AD task -> [身份与 Windows](../../competition-identity-windows/README.md)
- Host-to-host pivot, WinRM, SMB, RDP, accepted replay edge -> [Windows 身份与主机关系](../../competition-windows-pivot/README.md)
- LSASS memory, ticket caches, replayable session material -> [LSASS 与票据材料](../../competition-lsass-ticket-material/README.md)
- DPAPI masterkeys, vault blobs, browser secret stores, unwrap-to-access chain -> [DPAPI 数据与访问链](../../competition-dpapi-credential-chain/README.md)
- Kerberos delegation, S4U, RBCD, delegated ticket acceptance -> [Kerberos 委派](../../competition-kerberos-delegation/README.md)
- AD CS template, PKINIT, enrollment rights, cert-based privilege -> [AD 证书与身份映射](../../competition-ad-certificate-abuse/README.md)
- Coercion primitive, relay target, forced auth acceptance -> [认证触发与中继链](../../competition-relay-coercion-chain/README.md)
- Mailbox rules, forwarding, OAuth consent, delegate abuse -> [邮件与委托关系](../../competition-mailbox-abuse/README.md)
- Linux secret replay, socket trust edges, SSH or token pivots, host-to-host Linux movement -> [Linux 身份与主机关系](../../competition-linux-credential-pivot/README.md)

来源：原 `CTF-Sandbox-Orchestrator/ctf-sandbox-orchestrator/references/router-matrix.md`。
