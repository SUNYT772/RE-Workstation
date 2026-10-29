# Android 高级逆向参考

来源：skills/apk-reverse/references/android-advanced.md

## 主题与范围

Native SO/JNI、Java/native 运行时观察、内存表示、SSL Pinning、Root 判据、加固与 DEX、React Native 和 Flutter。原调用模板转为对象、字段、语义和证据，不保留注入代码。

## 知识正文

### Native SO 逆向：构成与 JNI 入口

APK 内的 .so 所属 ABI、文件基本信息、符号、字符串、依赖和入口共同决定 native 分析上下文。Java 声明的 native 方法、库中密钥相关字符串/URL/错误信息的交叉引用，以及 AES/MD5/SHA 等密码学调用，都是定位关键逻辑的线索；不复制实际密钥值。

### JNI 函数注册

静态注册的名称形态是 Java_包名_类名_方法名；源例将 Security.getSign 对应到 native_getSign 的意义，是建立 Java 声明与原生实现的关系。动态注册通常可从 JNI_OnLoad、JavaVM、JNIEnv、FindClass 和 RegisterNatives 关联到注册表。

JNINativeMethod 记录方法名、JNI 签名及函数地址。源例签名 (Ljava/lang/String;)Ljava/lang/String; 表示接收 String、返回 String。JNI_VERSION_1_6 是源例的环境版本标记。实例方法的首参数是 JNIEnv*，随后为对象引用；正确的 JNI 类型有助于解释 FindClass、GetMethodID 等间接调用。

### JNI 类型与间接表识别

原文把 RegisterNatives 第三个参数视为 JNINativeMethod 数组，可由其中的方法名、签名及函数地址重建映射。原文还记载“ARM64 vtable offset 0x35C”作为定位线索；该数值仅是原材料中的待核对线索，本文未校验其与实际 ABI/头文件的一致性，不将固定偏移写成通用事实。类型与实际调用参数比单个数值更能支撑结论。

### Native 函数观察

libc 的 open 等文件访问边界可体现检测读取了什么路径；自定义库中函数可用模块基址与相对偏移描述位置。参数的 UTF-8 字符串解释、返回值是指针还是整数，以及调用前后同一事件的对应，是源模板承载的观察目标。源例曾改变 su/magisk 路径访问的返回值；知识上保留为“检测依赖文件可见性”的行为假设，不提供替换实现。

### Java 方法、构造与重载

实例方法的输入与原始返回说明业务转换；构造函数中参数说明初始化来源；同名重载由完整参数类型区分。源例分别涉及 getSign、接收 String 的构造函数以及接收 String/int 的 encrypt。记录完整重载可避免将相同方法名的不同语义混在一起。

### 内存搜索与修改的解释

模块名、基址、大小和匹配地址共同定义内存搜索范围。源例的 48 65 6C 6C 6F 是 ASCII “Hello”的表示，说明字节模式与字符串之间的对应。指令补丁例的知识点是 ARM64 指令长度、地址、原始指令与替换后的控制流差异；本文不保留 patch 调用和固定修改地址，也不将示例偏移视作真实目标地址。

### SSL Pinning：通用层次

X509TrustManager 的 checkClientTrusted、checkServerTrusted 与 getAcceptedIssuers 分别涉及客户端链、服务端链及受信发行者；SSLContext 初始化将信任管理器与 TLS 上下文关联；应用框架还可能叠加独立证书绑定逻辑。原例改变这些层的校验结果，说明“系统信任链通过”与“应用绑定通过”是不同命题，不表示一种修改通用于所有应用。

### 各框架证书观察面

| 框架 | 原文对应的证书/网络对象 |
|---|---|
| OkHttp3 | CertificatePinner.check，主机名与证书列表 |
| Retrofit | 底层 OkHttp 的绑定逻辑 |
| Volley | HurlStack 的 SSL 工厂 |
| Flutter | dart:io 的 SecurityContext 与 native TLS 逻辑 |
| React Native | OkHttpClientProvider |
| WebView | WebViewClient.onReceivedSslError 与错误处理结果 |

### Flutter 证书专项

源例将 libflutter.so 中 ssl_verify_peer_cert 作为 native 验证候选，并用 ARM64 特征搜索来定位。知识要点是引擎版本、架构、匹配范围和候选函数语义之间的对应；特征命中本身不能证明函数身份。源字节签名和直接替换回调不作为可复用 payload 复制。

### Root 检测：可见性与属性

| 检测类别 | 具体观察项 | 原材料所表达的行为差异 |
|---|---|---|
| 文件存在性 | /system/app/Superuser.apk；su、Superuser、magisk、busybox、xposed 相关路径 | File.exists 的结果影响本地判定 |
| 命令相关性 | Runtime.exec 接收的 su 检查意图 | 执行结果或异常影响判定 |
| 挂载与目录 | /proc/self/mounts；/data/adb/；opendir/access | 挂载内容及目录可见性影响判定 |
| 包信息 | Magisk 等管理组件 | PackageManager 可见的包身份影响判定 |
| 系统属性 | ro.debuggable、ro.secure | 属性值是检测输入，不能机械统一赋值 |
| 完整性服务 | SafetyNet / Play Integrity | 与本地文件检查不同的完整性信号 |

源文提到 Magisk Hide、Zygisk、Shamiko 与包名随机化，保留为历史检测对抗概念，不要求使用或安装。源例将两个系统属性同时写成同一值，本文仅保留字段和判据，不将示例常量当作未 Root 的证明。

### 加固/壳识别

| 原文厂商 | 识别线索 | 涉及的知识对象 |
|---|---|---|
| 360 | libjiagu.so；com.stub.StubApp | 壳入口与运行时 DEX |
| 腾讯乐固 | libshell*.so；com.tencent.StubShell | 壳入口、ClassLoader 与 DEX |
| 梆梆 | libDexHelper.so；com.secneo.apkwrapper | 动态解密/加载边界 |
| 爱加密 | libexec.so；s.h.e.l.l | 壳层与内存中的代码 |
| 网易易盾 | libnesec.so | 保护库归因 |
| 娜迦 | libnaga.so | 保护库归因 |

这些是原文的识别线索，不构成仅凭文件名判断厂商或保护强度的证据。

### 通用脱壳方法的知识含义

FART 表示 ART/ClassLoader 加载的 DEX 观察；Frida DEX Dump 表示 DexFile::OpenMemory 与内存代码边界；BlackDex 在原文中是免 Root 类方案；手工观察将应用 ClassLoader、DexFile 对象、内存区域与持久化结果关联。源示例的 ClassLoader 列举仅展示加载器和 DEX 名称线索，不等于已恢复所有 DEX 内容。

### React Native 逆向

assets/index.android.bundle 常承载 JavaScript，可读线索包括 API 地址、签名逻辑及敏感常量引用。Hermes 字节码 .hbc 与普通 JS 文本不同，需按其格式理解；hermes-dec 是源文中的历史反编译名称。Java 层 ReactBridge 是 JS/native 交互的观察边界。

### Flutter 逆向

libapp.so 承载 Dart AOT，不能直接等同于 Dart 源码；libflutter.so 属于引擎侧。源文中的 reFlutter 关注引擎与 snapshot，Doldrums 关注 Dart snapshot 的类/函数信息，运行时观察关注引擎关键函数。源文“Flutter 不走系统代理”应理解为该场景中系统代理未必覆盖实际网络栈；证书与网络路径需要单独取证，不能由框架名称推断实际流量路径。

### 历史能力与参考概念

Java 反编译、资源重建、动态观察、移动安全评估、运行时 DEX 恢复、Hermes 字节码、Dart snapshot、Root 可见性对应原文的 jadx、apktool、Frida、Objection、MobSF、BlackDex、FART、hermes-dec、reFlutter、Magisk/Shamiko。参考名称保留 OWASP MASTG、FridaBypassKit、SSL-bypass、awesome-frida、Android Security Awesome；不附安装或连接步骤。

## 观察与判断

Java 与 native 归因需有 JNI 映射；证书失败需区分信任链、绑定与框架层；Root 信号需区分文件、进程、属性、包和完整性服务；运行时 DEX 名称不等于 DEX 恢复完成。原例的常量地址、设备属性和特征码均为示意，不是跨版本不变知识。

## 记录与报告

记录 ABI、模块、入口与 JNI 表映射、函数类型和参数、内存范围、证书校验层、检测输入、保护线索、ClassLoader/DEX 关联及框架归属。说明观察值与主动改变行为后的结果之间的差别，并标记未验证偏移及未解释保护层。
