# APK 逆向：结构、逻辑与证据

来源：skills/apk-reverse/SKILL.md

## 主题与范围

覆盖 Java 业务逻辑、登录、签名、风控、证书校验、Root 检测、AndroidManifest.xml、smali、重打包、Java/native 运行时观察及 APK 中 .so 的分析归因。

## 知识正文

### APK 的互补视图

Java 反编译视图适合理解包、类、方法与高层业务；资源及 smali 视图保留 Manifest、资源和字节码层结构。两者分别表达可读语义与实际构造。Java 反编译局部失败仍可能有可用源码，不能据此断定逻辑不可分析。

原有 decode 材料的可观察产物包括包名 package、Java 文件数 java_files、smali 目录数 smali_dirs、原生库数 so_files；Windows 材料还包含 xml_files。各视图状态需要独立记录。

### 构成与入口

AndroidManifest.xml 定义 package、application、activity、service、receiver、provider 和权限；主启动 activity 与 Application 的初始化逻辑解释业务从何处开始。lib/ 中 .so 的存在及 ABI 目录帮助判断是否有 native 边界。

Java 阅读对象包括 MainActivity、Application、登录/网络/加密/风控相关类和第三方 SDK 初始化类。源文检索词完整保留为 login、sign、encrypt、cipher、token、root、certificate、trust、okhttp、retrofit、webview；它们是线索，不自动构成风险证据。

### Smali 与资源层确认

Java 结果不完整、混淆较重或需解释字节码行为时，smali*/、res/values/strings.xml 与 Manifest 提供交叉证据。android:exported、调试标记、Root 检测返回分支、登录验证分支和证书校验分支是源文的重点观察对象。记录中应指出具体类/方法/分支及语义，不能只写“已 patch”。

### 重建、对齐、签名与设备状态

重建 APK、ZIP 对齐、签名、验签和设备接受是不同结果。可重建不等于可被设备接受；产物的签名身份也不等于原始包签名身份。unsigned、aligned、signed 三类产物及对应状态可用于区分故障位置。原文调试密钥的生成、默认口令及安装调用在此不保留，签名身份与验证结论仍作为报告字段。

### 运行时观察

登录函数、OkHttp/Retrofit/WebView 边界、javax.crypto、MessageDigest、Root 检测及 SSL pinning 是源文列出的目标。入参与返回值对照说明实际转换；构造函数、方法重载及调用时机决定样例是否属于目标逻辑。源文“先观察值再考虑修改”的经验在此体现为：区分原始观察结果与改变行为后的结果，避免将二者混为证据。

### Native .so 归因

Java 仅是 JNI 包装、核心签名不在 Java、System.loadLibrary() 后关键逻辑消失、证书校验或风控位于 .so 时，应将问题描述到 native 边界。lib/arm64-v8a/ 与 lib/armeabi-v7a/ 区分常见 ABI。导出符号、字符串与快速构成信息适合初步归因；反编译、函数命名、类型恢复适合深入语义分析，不绑定特定产品。

### 历史能力名称

jadx 对应 Java 反编译和符号检索；apktool 对应资源/smali 结构与重建；Frida 对应运行时函数观察；adb 对应设备、日志与文件上下文；zipalign/apksigner 对应对齐与签名结果。IDA、Ghidra、radare2 在原文中表示 native 分析视角。版本清单、本机路径和安装体系属于运行环境材料，不是本知识页的能力前提。

## 观察与判断

未明确 Manifest、主入口与业务对象就改变字节码或运行时返回值，容易失去原始行为依据。反编译缺失可由 smali、资源、运行时及 native 证据补充；核心已落在 .so 时，继续只读 Java 不能解释算法。

设备与进程状态需要明确区分：包标识、进程名、设备身份、启动前观察与附加到现有进程的时机不同，可能解释观测缺失。细节见对应知识页。

## 记录与报告

保留入口组件与关键类、逻辑所属 Java/smali/.so、登录/签名/Root/SSL/WebView/JNI 的已确认敏感点。涉及修改时说明对象和语义变化；涉及运行时观察时说明类、方法、重载或导出函数、输入、返回和结果。可复核证据包括源码位置、字节码位置、日志、截图与报告，不以工具调用或阶段打勾替代结论。
