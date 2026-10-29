# 移动运行时对象、宿主与证书层

来源：skills/mobile-reverse/references/frida-objection-deep.md

## 主题与范围

原 Frida/Objection 用法转为 Java、native、ObjC、数据存储和应用内运行时的知识。保留非 Root/越狱方式的结构含义，不保留部署、连接、调用或注入代码。

## 知识正文

### Java 运行时（Android）

类表示、实例、静态方法、构造函数与已加载类清单是不同对象。源例 System.getProperty 的 String 重载提供属性名及原返回；File 的 String 构造提供文件路径与对象创建时机；已加载类清单说明该时刻可见范围；RootDetector.isDeviceRooted 的改变结果属于干预，不是原始 Root 状态。完整签名与加载时机有助于定位目标类和重载。

### Native 层（Android 与 iOS）

导出符号 open 的路径输入与返回说明文件访问；模块名/基址/相对偏移描述自定义函数位置；线程上下文、准确回溯及符号化提供调用来源。源 strcmp 示例把不相等结果改为零，知识点是字符串比较结果如何影响分支；这里不复制替换代码。固定示例偏移不具有跨版本含义。

### ObjC 运行时（iOS）

ViewController 的实例方法 viewDidLoad 表示生命周期观察点；ObjC 已加载类清单反映运行时类范围；NSString 的构造/方法语义体现对象与字符串之间的桥接。方法 selector 与 implementation 的对应决定观察对象，不能仅按显示名称推断原生函数签名。

### 通用能力的技术目标

原命令清单涵盖交互会话及只加载不等待的会话方式、Gadget 随包加载、单纯重签名、应用数据目录、文件清单、文件输入/输出，以及 SQLite 的表清单与查询结果。知识上区分：应用目录是路径上下文，文件传输是材料来源，数据库表/行是结构化证据；它们不相互替代。本文不保留上传/下载调用或 SQL 执行示例。

### Android 专用观察面

Root 与证书判定、类清单、类方法清单、某类方法的覆盖范围、Activity/Intent 入口、堆内对象实例以及 Keystore 条目分别对应环境、行为、入口、状态和密钥元信息。Keystore 条目可见不等于密钥可导出，也不在知识页保存密钥。

### iOS 专用观察面

越狱与证书判定、Keychain 的条目与保护、NSUserDefaults 的持久化值、NSURLCache 的 HTTP 缓存、Cookies、pasteboard、UI 层次结构与 Info.plist 是原文的完整范围。报告说明这些位置是否含敏感信息及来源关系，不复制真实凭据值。

### Android Gadget 的结构含义

原材料将架构对应的共享库放入 APK 的 ABI 库目录，并在主 Activity.onCreate 或 attachBaseContext 一类初始化位置涉及 System.loadLibrary；知识点是 ABI 匹配、宿主加载时机、重建样本及重签名身份。是否包含资源变化也是样本差异的一部分。此处不规定如何修改 smali 或打包/安装。

### iOS Gadget 的结构含义

原材料涉及 App Store 二进制的解密状态、FridaGadget.dylib、Mach-O Load Commands 的依赖记录、@executable_path 相对定位与重签名身份。Xcode sideload/AltStore 是历史分发名称，不附部署步骤。由此产生的样本与原签名样本不同，需说明观察条件。

### Android 多层证书观察

CertificatePinner.check 的主机名和证书列表、Conscrypt TrustManagerImpl.verifyChain、WebView SslErrorHandler.proceed、Manifest 的 android:networkSecurityConfig 与 @xml/network_security_config 各有不同作用。源 proceed 示例回调仍调用原 proceed，仅凭该片段不能证明忽略了其他错误；原“返回空链”的语义也需按方法契约解释。

### iOS 多层证书观察

NSURLSession 所关联的 Security/SecTrustEvaluate 有函数返回状态及通过指针写出的信任结果两个层次；源例将后者设为 kSecTrustResultProceed=1、前者设为 errSecSuccess=0。Alamofire 的 ServerTrustManager.evaluate 则是框架层决策。保留这两个层次与常量含义，不复制改变结果的回调。

## 观察与判断

类枚举、方法清单、对象实例与实际业务调用是不同覆盖面；记录原输入/返回与改变后的行为需分开。随包加载、解密状态或改签名可能影响完整性检测，不能将修改样本的效果直接归给原始样本。

## 记录与报告

记录平台/架构、类/selector/符号、签名与地址语境、调用栈、存储位置、宿主加载边界、样本差异及证书校验层。原参考名称为 Frida 文档、Objection wiki、OWASP MSTG。
