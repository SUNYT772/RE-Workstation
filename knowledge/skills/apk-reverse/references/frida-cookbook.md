# 移动运行时观察场景与算法

来源：skills/apk-reverse/references/frida-cookbook.md

## 主题与范围

原文从 awesome-frida、Frida-Mobile-Scripts、frida-codeshare-scripts 汇集 Java、密码学、网络、检测、存储与 DEX 模板。这里逐项保留模板承载的技术对象和观察契约，移除调用代码与行为替换 payload。

## 知识正文

### 任意 Java 方法

无参方法关注调用是否发生及返回；有参方法需记录完整重载类型，如 String/int，以及调用前入参和调用后返回。保留原调用语义与只观察返回值的区别：观察模板的目标是解释原结果，而主动改变返回的材料属于不同证据。

### 构造函数

接收 String 的构造参数可说明初始化值来源。构造时机、对象身份和其后方法调用之间的关联，比单独的一条参数日志更能说明生命周期。

### 枚举所有方法

getDeclaredMethods 提供类自身声明的方法及签名，是核对重载与可观察方法的依据；名称相同不代表签名或业务角色相同。

### AES 加解密

| 对象 | 原模板的观察字段 |
|---|---|
| javax.crypto.Cipher.doFinal 的字节数组重载 | 输入字节、输出字节、可确认时的模式 |
| SecretKeySpec 的字节数组/算法构造 | 算法名称与密钥来源关联；知识页不复制密钥值 |
| IvParameterSpec 的字节数组构造 | IV 来源、长度与该次计算的关联 |

源模板将无法取得的操作模式记为问号；不能把未知模式填成已确认事实。密钥、IV、输入和输出需属于同一计算上下文，才可用于解释加解密链路。

### MD5/SHA

MessageDigest 的算法名、含输入字节数组的 digest 重载与无参 digest 重载有不同的输入可见性。含参数重载直接观察该次输入；无参版本只展示算法与最终摘要，可能缺少之前更新的完整输入。保留这种证据局限，不把无参结果自动写成完整输入链已知。

### HMAC

javax.crypto.Mac 的算法、init 接收的 Key 来源、doFinal 的输入与 MAC 输出构成带密钥摘要的证据。密钥可能不可导出，getEncoded 对应的可见性需实际确认；本文保留“来源与计算关联”而不记录凭据值。

### OkHttp3 请求与响应

OkHttpClient、Interceptor、RealCall 与 execute 表示不同网络对象。源模板实际观察同步 RealCall.execute：请求 method、URL、逐项 header 名/值，以及响应 code。标题提到请求/响应，不表示已经覆盖响应体或全部异步路径；报告需要明确实际范围。

### URL 连接

java.net.URL.openConnection 的 URL 是连接层线索；该方法的出现说明连接对象构造，不单独证明网络请求已成功或响应已返回。

### WebView

android.webkit.WebView.loadUrl 反映页面加载目标；evaluateJavascript 反映 JavaScript 执行内容及 callback 上下文。源模板只打印脚本开头 200 个字符，因此片段不等于完整执行代码。页面 URL 与脚本来源、native 业务链之间需另外关联。

### SSL Pinning：对象与结果

- OkHttp3 CertificatePinner.check 的主机名与 java.util.List 证书集合，表达应用绑定边界。
- Conscrypt TrustManagerImpl.verifyChain 的 untrustedChain 与返回链表达系统信任验证边界。
- X509TrustManager 的 checkClientTrusted、checkServerTrusted、getAcceptedIssuers 表达信任管理契约。
- NetworkSecurityConfig.isCleartextTrafficPermitted 表达明文流量策略；与证书绑定不是同一种机制，虽在源“通用绕过”例中并列，也应分别解释。

原模板包含跳过检查、返回未验证链及自定义信任管理器的代码；这里保留改变了哪类校验的知识，不复制实现，也不把多个层次归为一个校验点。

### Root 检测

文件可见性线索包括 su、Superuser、magisk、busybox、xposed，及 /system/xbin/su、/system/bin/su、/sbin/su、/data/local/xbin/su、/data/local/bin/su。大小写规范化与子串匹配是原模板的匹配语义；子串匹配也可能把普通路径误判为目标。

Runtime.exec 中的 su/which 意图及异常行为，是命令型检测输入；Build.TAGS 中 release-keys 是构建标签线索。源模板改变文件结果、制造执行异常及改变标签，说明不同检测输入的可见性，不表示这些单个输入能代表真实完整性状态。

### 反调试

Debug.isDebuggerConnected 反映 Java 调试状态。Native 层的 fopen 对 /proc/.../status 的读取和 TracerPid 字段是另一种观察面。源例只展示识别文件访问，并在注释中提到 fgets；不能将其写成已完成了 TracerPid 内容修正或整个反调试已消失。

### 模拟器检测

Build 的 FINGERPRINT、MODEL、MANUFACTURER、BRAND、DEVICE、PRODUCT、HARDWARE 与 TelephonyManager 的 getDeviceId、getSubscriberId、getSimSerialNumber 是源例的完整字段范围。源文固定 Pixel 2/walleye 和号码值只作为示意，本文不复制设备身份值；保留指纹字段之间及电话信息之间的对应关系这一观察目标。

### SharedPreferences

SharedPreferencesImpl.getString 的 key、默认值与实际返回；EditorImpl.putString 的 key/value 与写入动作分别说明读取和写入语义。Token 的生命周期需结合多次读写事件；知识页保留键和流向字段，不承载真实 Token。

### SQLite

SQLiteDatabase.rawQuery 的 SQL 与绑定 args，execSQL 的 SQL，分别提供查询及执行语句的观察面。绑定参数与直接拼接需区分；只看到 SQL 字符串不说明查询结果或事务已成功。

### 通用 DEX 与 ClassLoader

BaseDexClassLoader.pathList → dexElements → dexFile → 名称，是源模板表达的加载器与 DEX 关系。发现名称仅表示有对应条目，模板注释中的“进一步 dump”不是已实现的内容恢复。ClassLoader.loadClass 的 String 参数与目标包名关联，说明何时出现业务类；按包名过滤会遗漏范围外依赖。

### 字节数组转十六进制：纯算法伪代码

```text
输入：字节序列 bytes
若输入为缺失值，返回文字 null
对每个字节 b：
    u = b 按位与 255
    h = u 的十六进制表示
    将 h 左侧补零至两位并追加到结果
返回所有两位片段的连接
```

按位与 255 保留低八位，避免有符号字节影响表示；两位补零保持长度和字节边界。这是原文已有数据表示算法，不含工具调用。

### 调用栈、字段与内存实例

Throwable 的栈信息说明调用来源；类的 getDeclaredFields、字段名/值和不可读取的异常说明对象状态；类实例枚举及各实例字段将状态与对象关联。源例设置字段可访问性及忽略个别错误只是运行模板机制；知识结论应保留“哪些字段可见、哪些失败”的边界，而非声称所有字段已知。

### 历史参考概念

Frida 文档、Frida CodeShare、awesome-frida、frida-codeshare-scripts、Objection 与 r2frida 是原文参考资源名称，不作为安装或调用要求。

## 观察与判断

模板标题的覆盖面可能大于实际代码：同步 execute 不等于所有网络路径；前 200 字符不等于完整脚本；DEX 名称不等于内容恢复；文件命中不等于已改变字段。构造参数、密码学计算和请求事件应有同一上下文关联。

## 记录与报告

记录类、方法、完整签名、对象/进程上下文、输入类型、结果、调用栈、覆盖范围与缺失信息。算法样例使用数据表示或字段来源，不保存密钥、Token、设备身份样本或注入代码。
