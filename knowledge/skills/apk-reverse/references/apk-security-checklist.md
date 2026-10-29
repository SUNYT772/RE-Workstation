# APK 安全检查知识

来源：skills/apk-reverse/references/apk-security-checklist.md

## 主题与范围

原文基于 OWASP MASTG，覆盖静态、动态、网络通信、数据存储、认证授权和代码保护。保留检查项与风险含义，移除执行命令、批量请求与 payload。

## 知识正文

### 静态分析：Manifest 审计

| 项目 | 观察与判定依据 |
|---|---|
| android:debuggable | true 表示可调试，需说明是否为生产构建 |
| android:allowBackup | true 表示声明允许备份，实际可提取性仍需环境证据 |
| android:exported | 暴露的 Activity/Service/Receiver/Provider 与其访问约束 |
| 自定义权限 protectionLevel | normal 与 signature 的信任边界不同；原文关注需要签名约束却只采用普通权限的情况 |
| intent-filter 的 scheme | 自定义 deeplink 的归属和处理是否可被其他应用影响 |
| android:usesCleartextTraffic | true 表示允许明文 HTTP 的策略线索 |
| minSdkVersion | 较旧支持范围可能缺少较新安全特性；版本值本身不是漏洞证明 |

### 静态分析：代码审计关键点

- 硬编码敏感值：key、secret、password、api_key 等词定位密钥/Token 的使用位置，不在知识页复制值。
- 随机性：java.util.Random 与 SecureRandom 的用途不同，关注安全敏感计算实际使用的来源。
- 密码学：ECB、DES、将 MD5 用于密码等是源文风险项，需结合用途说明。
- WebView：setJavaScriptEnabled、addJavascriptInterface 暴露的接口、加载内容和信任边界共同决定风险，不能只凭两项同时出现就认定 RCE。
- SQL：rawQuery 中用户输入的字符串拼接与绑定参数差别。
- 路径：ContentProvider.openFile 是否验证路径边界。
- 日志：Log.d/Log.i 的敏感内容。
- 剪贴板：ClipboardManager 中敏感数据的可见性。
- 隐式 Intent：sendBroadcast 未指定包名时的数据接收范围。

### 静态分析：第三方库

记录 OkHttp/Retrofit、WebView 内核、SDK 的实际版本与已知漏洞对应关系；广告 SDK 的采集范围、推送 SDK 的 Token 配置泄露分别属于数据范围与配置问题。源文提及 CVE 对照，本文不新增或断言具体漏洞版本。

### 动态分析：观察目标

| 目标 | 原文定位对象 | 证据内容 |
|---|---|---|
| 登录认证 | LoginActivity.login | 凭证处理、参数流向 |
| 签名生成 | 含 Sign/sign/encrypt 的方法 | 原始输入、转换与输出 |
| 证书绑定 | CertificatePinner.check | 主机名、证书列表、校验结论 |
| Root 判定 | 含 root/su/magisk 的逻辑 | 检测读取及分支 |
| 加密 | javax.crypto.Cipher | 算法、模式、输入输出及密钥/IV 来源；不复制密钥值 |
| Token 存储 | SharedPreferences.getString | 存储键、读写位置、生命周期 |
| 网络构造 | OkHttpClient.newCall | 请求构造来源与最终请求 |

原一行追踪命令承载的覆盖范围是 Cipher、OkHttp、SharedPreferences 与 Java_* native 边界。原 Objection 命令承载的对象是 activities、services、证书绑定、Root、剪贴板、应用目录和 SQLite；这里保留其技术目标，不保留调用。

### 网络通信：证据路径

系统代理与 CA 信任、VPN 流量视角、进程内网络边界是不同取证路径。Android 7+ 的 network_security_config 影响用户证书信任；仅有代理和证书未必可见应用绑定后的明文。VPN 视角可看到流量但不意味着能够解密证书绑定的 TLS。进程内观察的边界不同于代理/VPN，仍需说明采样位置。

### 网络通信：完整检查项

所有 API 是否使用 HTTPS；是否存在 SSL Pinning；证书验证是否接受不受信自签名；是否有证书透明度 CT；API 密钥是否明文传输；Token 过期机制；请求签名防篡改；nonce/timestamp 防重放；WebSocket 是否加密；URL 参数是否包含会进入日志的敏感数据。绑定、信任验证、CT、签名和重放防护是不同机制，不互相替代。

### 数据存储：位置与可见性

| 位置 | 风险/判据 |
|---|---|
| SharedPreferences；shared_prefs XML | 明文 Token/密码、读写来源 |
| SQLite；databases | 敏感数据是否明文与加密情况 |
| 外部存储；/sdcard/Android/data/pkg/ | 应用外可见性需结合实际访问控制，源文将其列为重点风险 |
| 应用日志 | 敏感信息与调试内容 |
| 备份文件 | 与 allowBackup 及实际备份能力关联 |
| 键盘缓存 | inputType 是否表达 textPassword 等敏感输入意图 |
| 截图保护 | 敏感页面 FLAG_SECURE 的实际效果 |

### 加密存储方案对比

SharedPreferences 明文在高权限环境可直接暴露；EncryptedSharedPreferences 对应 AndroidX Security 的加密存储；SQLCipher 对应 SQLite 加密；Android Keystore 对应密钥保护且硬件支持需实际确认；自定义 AES 的安全性取决于密钥管理，不能只凭“AES”作结论。

### 认证与授权

| 问题 | 应保留的判断关系 |
|---|---|
| 弱密码策略 | 服务端接受规则及其强度证据 |
| 无锁定/无频率限制 | 登录失败处理、速率和锁定策略 |
| Token 不过期 | 登出或过期后旧 Token 的实际有效性 |
| 越权访问 | 身份与 user_id 等资源归属是否一致 |
| 短信验证码风险 | 4/6 位码空间与请求频率限制的关系 |
| OAuth 配置错误 | redirect_uri 的约束与校验结果 |
| 生物认证 | BiometricPrompt 的本地结果与真实授权是否绑定 |
| 设备绑定 | device_id 的可信来源及服务端约束 |

源 payload 所承载的判据保留为：用户 A 的身份不能访问用户 B 私有资源；登出后旧 Token 应被拒绝，源例预期为 401；验证码空间小且缺少频控构成风险。批量枚举、HTTP 调用、示例手机号和 Token 字面量不复制。

### 代码保护评估

| 保护 | 观察信号 | 源文的相对复杂度含义 |
|---|---|---|
| ProGuard 名称混淆 | a/b/c 类名 | 低，仅改名不等于算法隐藏 |
| 字符串加密 | 解密函数、运行时明文 | 中，需解释解密边界 |
| 反调试 | 调试附加前后行为 | 中，需定位检测点 |
| Root 检测 | 不同设备状态下的判定 | 中，检测范围决定效果 |
| 模拟器检测 | 模拟环境特征 | 低至中 |
| 完整性校验 | APK 改动与校验结果 | 中 |
| 加固/壳 | 入口类、保护 .so、运行时 DEX | 中至高 |
| Native 保护 | 核心算法位于 .so | 高，需 native 语义 |
| VMP | 虚拟化执行结构 | 极高，原文未给具体还原算法 |

这些难度是源文经验排序，不是固定工时或普遍保证。

### 快速评估覆盖

原“30 分钟”分配表达 5 类覆盖：Manifest 的 debuggable/allowBackup/exported/cleartext；代码中的 password/key/secret/token/明文协议线索；网络中的明文或弱加密；shared_prefs/databases；关键函数动态证据。原 5/10/5/5/5 分钟为历史时间安排，保留覆盖面，不作为 agent 时间或阶段约束。

## 观察与判断

配置、源码线索、运行时行为和服务端结果应分别说明。静态可疑点需要实际可达路径；保护存在不等于完全有效；观察失败也不能自动归因于某一保护。

## 记录与报告

按六个维度记录对象、位置、信任边界、直接证据、风险条件与未验证项。引用组件、类/方法、请求字段、存储位置和验证结论，不输出攻击请求或批量枚举脚本。
