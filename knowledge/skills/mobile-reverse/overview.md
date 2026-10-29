# Android 与 iOS 逆向的知识框架

来源：skills/mobile-reverse/SKILL.md

## 主题与范围

APK/IPA 结构与安全、运行时动态观察、SSL Pinning、Root/越狱及反调试、AES/RSA/HMAC 相关算法与密钥来源关系、OWASP MASTG 评估，以及非 Root/非越狱环境的观察边界。以下按知识对象编排，不规定四阶段执行。

## 知识正文

### 信息与包身份

Android 原材料的获取来源包含应用商店、镜像站和已安装包；来源身份及样本一致性与后续判断相关。Manifest 中权限、导出组件、Intent Filter、backup 标志和签名构成包的外部边界；硬编码 API Key/Token/Secret 的位置属于静态风险线索；360、腾讯、梆梆、爱加密是源文加固识别的历史类别。

iOS 的来源包括 App Store、设备及配置器导出。App Store 二进制的加密状态关系到能否直接解释代码；Info.plist 的 ATS、URL Scheme 和 Queries Schemes 关系到网络策略与应用间跳转。ObjC 类声明、Swift/ObjC 混淆是不同语义恢复对象。

### 静态分析对象

Android 的 Java 视图、DEX/JAR 表示、smali/Dalvik 字节码、资源与 .so 相互补充。iOS 的 Mach-O、ObjC 头信息、Swift 修饰符号、调试符号和动态库依赖解释代码构成。单个 .so/Mach-O 的控制流、架构和调用约定不能由包层信息代替。

原文中的 JADX-GUI、apktool、dex2jar/JD-GUI、smali/baksmali 分别对应 Android 不同表示；Ghidra/Hopper、radare2/Cutter 对应 native 语义或构成；class-dump、swift-demangle、dsymutil、otool、jtool2 分别对应 ObjC、Swift、调试符号、依赖与 Mach-O 信息。名称不是依赖要求。

### 动态分析对象

设备进程、函数调用、参数和返回、私有方法与对象状态是运行时证据。Frida 在原文中表示通用插桩，Objection 表示封装后的交互视角；各自承载的知识包括检测分支、证书校验、Android Keystore/iOS Keychain、应用目录及 SQLite。改变返回值后的行为与原始观察结果需要分开记载。

### 非 Root/越狱环境

Gadget 材料说明运行时组件可随应用自身加载；Android 的 .so 与 iOS 的 .dylib、应用重签名和加载时机是关键概念。非 Root/非越狱不是“无需修改样本”的同义词：样本签名与二进制已可能不同，观察结果需要说明这种差异。此处不保留注入、部署或设备安装过程。

### 网络分析

HTTP/HTTPS 请求和响应、PCAP、代理信任、用户/系统证书、SSL Pinning、WebSocket 与 gRPC 分别提供不同层次证据。Burp Suite、mitmproxy、Wireshark 是源文的历史观察工具；Magisk/MoveCert、Xposed、SSL Kill Switch 2 是证书或检测对抗的历史名称，均不成为固定依赖。代理看不到明文需要区分路由、信任链和绑定逻辑。

### 常见检测判据

Android RootBeer.isRooted 及其细分项、Magisk/su 文件或包、frida-server 进程与 /proc/self/maps 都可能参与环境判断。Android 反调试涉及 ptrace/TracerPid、/proc/self/status 和 isDebuggerConnected；iOS 涉及 PT_DENY_ATTACH 与 sysctl 的 CTL_KERN/KERN_PROC/KERN_PROC_PID。SSL Pinning 可能在系统信任、网络框架及 native 层，不是单点问题。

### 移动端密码学字段

Android Cipher.getInstance 的 String 描述算法；Cipher.init 的 int 模式与 java.security.Key 表示计算模式及密钥来源。源标题写“获取密钥+算法”，但 getInstance 本身只提供算法选择；密钥关联来自初始化边界。

iOS libcommonCrypto.dylib 中 CCCrypt 的源例观察参数位置为：第 0 项操作、第 1 项算法、第 3 项密钥地址、第 4 项长度。记录字段含义和长度关系，不复制内存密钥。源页还将 AES/RSA/HMAC 列为目标，未给 RSA/HMAC 的具体提取算法，不在转写中补造实现。

### 能力分布与参考

源工具表中 Android 侧包括 Java 反编译和 APK 重建；iOS 侧包括 ObjC 类、IPA 加密状态及 Mach-O；两端共同包括 native 反编译、动态观察、自动 SAST/DAST 和 HTTP 分析。MobSF 表示自动评估视角。参考概念保留 OWASP MSTG/MASTG，细节见本模块三份参考页。

## 观察与判断

加密状态、混淆与不可反编译不是同一问题；可见一个方法不等于可解释完整业务；证书失败、设备检测和进程崩溃需要实际定位；改签名或随包加载组件可能改变观察条件。

## 记录与报告

记录包及二进制身份、平台/架构、入口、静态位置、动态参数/返回、密码学字段来源、网络证据及检测层；说明样本是否改变、复现是否稳定和仍未解释的保护。输出是知识与证据结构，不是工具调用或执行阶段打勾。
