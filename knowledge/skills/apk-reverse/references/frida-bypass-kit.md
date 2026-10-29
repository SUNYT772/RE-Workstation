# 通用检测框架的观察点

来源：skills/apk-reverse/references/frida-bypass-kit.md

## 主题与范围

FridaBypassKit 是源文记载于 2025 年的 Android 检测对抗框架名称。其四类能力转写为检测对象与可观察结果；“通用、开箱即用”是原项目描述，不作为适用性保证。

## 知识正文

### Root 检测

File.exists 对 su 二进制等路径的可见性、Runtime.exec 中 Root 检查意图、PackageManager 对 Magisk/SuperSU 相关包的可见性，以及系统属性，分别构成本地 Root 判定输入。原方案改变这些输入的呈现，知识上需要区分实际设备状态与应用收到的视图。

### SSL Pinning

TrustManagerImpl.verifyChain 与 checkTrustedRecursive 涉及证书链验证。源文以返回链或空链等变化表达校验被改变，覆盖 OkHttp、Retrofit 与自定义实现的意图；实际方法签名、返回类型和应用自定义绑定仍需证据，不能将“返回空链”当作各框架的通用正确行为。

### 模拟器检测

TelephonyManager 的设备/电话/运营商信息及 Build 属性是环境指纹。源方案伪造这些值说明判定依赖输入，但设备信息在不同接口间的一致性同样关系到结论。本文不复制假号码或固定设备身份。

### 反调试

Debug.isDebuggerConnected 是 Java 调试连接状态信号。调试状态变化与进程崩溃之间的相关性只能产生候选解释；崩溃本身不证明命中了反调试。

### 场景与诊断关系

| 现象 | 原文对应的候选原因 | 需要的证据 |
|---|---|---|
| Root 设备拒绝运行 | Root Detection | 实际文件/包/属性读取与判定分支 |
| HTTPS 看不到明文 | SSL Pinning 或信任路径问题 | 网络是否发生、TLS 错误及绑定位置 |
| 模拟器拒绝运行 | Emulator Detection | Build/Telephony 输入与拒绝分支 |
| 运行时附加后崩溃 | Debug Detection 等 | 崩溃栈、时序和实际检测读取 |

源文“先组合全部能力再调整”在此保留为多类检测可能叠加的知识；不要求固定组合或改变任何返回值。

### 历史参考名称

httptoolkit/frida-interception-and-unpinning 关注 HTTPS 进程内观察与绑定；0xCD4/SSL-bypass 和 incogbyte/ssl-bypass 关注常见证书绑定；Zero3141/Frida-OkHttp-Bypass 关注 CertificatePinner。名称用于辨识历史来源，不提供下载、安装、服务启动或注入操作。

## 观察与判断

隐藏一个信号不等于设备未 Root、不是模拟器或没有调试；改变证书校验也不等于原始通信安全。区分底层事实、接口返回和业务判定。

## 记录与报告

保留现象、检测接口、输入类型、返回含义、判定位置和证据；说明原项目的通用性主张尚需按具体应用验证。
