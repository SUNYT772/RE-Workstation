# Root、越狱、反调试与证书校验判据

来源：skills/mobile-reverse/references/anti-detection-bypass.md

## 主题与范围

源文的检测层次模型、Android/iOS 检测对象及证书校验层。绕过代码转为接口输入、返回含义、状态差异和证据边界。

## 知识正文

### 检测层次模型

| 层次 | 触发上下文 | 完整观察项 |
|---|---|---|
| Layer 1：静态/启动检测 | 安装时、启动时 | 包管理器中的 Cydia/apt/Magisk；su/busybox/frida-server 文件；ro.debuggable/ro.secure 属性 |
| Layer 2：持续运行检测 | 运行中 | frida-server/magiskd 进程；原文 27042 端口指纹；/proc/self/maps 的加载痕迹；插桩调用帧 |
| Layer 3：环境检测 | 按需触发 | ptrace 与 TracerPid；/proc/self/status；build.prop/test-keys；直接 syscall 而非 libc 边界 |

端口号是原材料的历史指纹，既不要求连接，也不能单独证明进程身份。检测可能跨层叠加，因此只解释某一 API 返回不等于环境判定整体改变。

### Android Root 检测库与服务

RootBeer 汇合多种本地检查；源例八项为 isRooted、isRootedWithBusyBox、checkSuExists、detectRootManagementApps、detectPotentiallyDangerousApps、detectTestKeys、checkForDangerousProps、checkForRWPaths。其名称分别表示总判断、BusyBox、su、管理应用、危险应用、构建标签、属性和可写路径。

SafetyNet 在源文中是 Google Play Services 的远程认证，Google Play Integrity 被描述为其后续完整性机制；它们不同于文件/包的本地可见性。Magisk Hide、Shamiko、Play Integrity Fix/PIF、Trickystore 是原文记录的历史对抗名称，不宣称它们对任意版本有效。

自定义 native 检测可能直接读取 syscall 结果或 /proc/self/status，故只观察 libc 不一定覆盖全部读取。

### Android 综合模板的知识含义

Build.TAGS 中 test-keys/release-keys 是构建标签信号。PackageManager.getPackageInfo 的包名与 flags 表示查询意图；源文对 de.robv.android.xposed.installer、magisk/frida 子串制造 NameNotFoundException，表达的是“查询视图被改变”，不能反证包实际不存在。RootBeer 的统一否定结果也不能说明各底层检查均无风险。

### iOS 越狱检测

文件线索完整包括 /Applications/Cydia.app、/var/lib/apt、/bin/bash、/usr/sbin/sshd、/etc/apt、/Library/MobileSubstrate，对应 NSFileManager 的文件可见性。fork 的成功与失败在源文中表达沙箱行为；LSApplicationWorkspace/defaultWorkspace/canOpenURL 和 cydia:// 表达 Scheme 可见性；MISValidateSignature 返回结果表达签名校验。不同层的输入需要与真正的拒绝分支关联。

### Android 反调试

ptrace/PTRACE_TRACEME 的自追踪意图、TracerPid 是否为零、/proc/self/status 的内容、Debug.isDebuggerConnected 的返回分别属于 native 追踪、进程状态及 Java 调试信号。源 fopen 示例只有路径观察和修改内容的注释，未形成完整伪造文件实现；不能将注释写成已完成效果。

### iOS 反调试

PT_DENY_ATTACH 在源例中对应 ptrace request 31，表达阻止附加的意图；sysctl 的 CTL_KERN/KERN_PROC/KERN_PROC_PID 返回 kinfo_proc，其 p_flag/P_TRACED 表示追踪状态。getppid 与源文中 launchd/父进程为 1 的假设提供另一条环境线索；特定环境中父进程身份仍需实际证据，不能只凭不等于 1 就确认调试。

### Android 五层证书模型

| 层 | 对象 | 技术含义 |
|---|---|---|
| 1 | TrustManager | 证书链的信任验证 |
| 2 | OkHttp CertificatePinner / pins | 应用对证书/公钥的绑定 |
| 3 | WebView SSL Error Handler | 页面证书错误后的业务处理 |
| 4 | Network Security Config XML | 用户证书等信任策略 |
| 5 | Native OpenSSL/BoringSSL；SSL_get_verify_result / X509_V_OK | native TLS 的校验结果 |

源文对这些层分别改变校验/处理结果；知识上保留边界，不提供接受任意证书或清空绑定的实现。

### iOS 四层证书模型

| 层 | 对象 | 技术含义 |
|---|---|---|
| 1 | NSURLSession / SecTrustEvaluate | 系统信任评估及 kSecTrustResultProceed |
| 2 | Alamofire ServerTrustManager | 网络框架的信任决策 |
| 3 | AFNetworking AFSecurityPolicy | 框架证书安全策略 |
| 4 | libcurl 的 SSL 验证回调 | 底层库的验证结果 |

源文在 libcurl 条目提及 LD_PRELOAD，本文只保留它是原文使用的历史加载机制名称，未核验其 iOS 适用性；不据此生成平台操作。原文把封装能力视为覆盖 Android 五层/iOS 四层的概括，也不等于具体版本已经全部命中。

## 观察与判断

实际设备状态、接口视图、检测结论和业务拒绝是四种不同证据。原代码中的返回替换、统一常量和异常模拟表达对这些关系的干预，不作为知识页的执行指令。

## 记录与报告

记录平台、检测层、接口、文件/包/字段、返回含义、触发时机及拒绝分支；证书部分按系统、框架、页面和 native 层注明已确认范围。原参考名称为 OWASP MSTG、Frida CodeShare、Objection wiki。
