# [种子] APK Frida 绕过 OkHttp SSL Pinning

来源：skills/field-journal/seed-008_apk-okhttp-ssl-pin-bypass.md

来源性质：**[种子] 预置案例素材，非已验证的真实实战成果。** 文中结果、耗时与修复为原素材中的条件化叙述，不表示本次或真实目标已完成验证。

本页为原案例/种子材料的知识转写。保留环境、观察、失败、修复与验证条件；工具调用、安装与载荷改为技术关系。

## 场景分类
APK 逆向 / 移动安全测试

## 目标概述
对一个使用 OkHttp + 自定义 CertificatePinner 的 Android 应用，用 Frida 动态绕过证书校验，让 Burp 能拿到明文流量。

## 观察、决策与验证

1. 原环境具有动态观察通道，目标身份与进程名需要对应
   
原设备连接观察区分 Android 进程列表和动态观察侧的目标身份，包名统一为〈Android 应用包〉。安装、端口转发和进程查询调用不保留；连接可达不等于证书校验路径已经观察成功。

2. 原代理观察出现证书错误；错误可能处于信任链、主机名或 Pinning，需结合调用路径区分
3. APK 静态结构的关键类线索为 `CertificatePinner` 或 `checkServerTrusted`
4. 确认是 OkHttp 自带 `CertificatePinner` 还是自定义 `X509TrustManager`
5. 运行时关键校验点与静态假设的对应
6. 观察时机早晚影响启动期校验覆盖，不保留注入命令
7. 原验证结果：代理可见 HTTPS 解密内容

## 失败路径与诊断

| 问题 | 原因 | 解决方案 | 耗时 |
|------|------|---------|------|
| Frida 启动报错 `unable to connect to remote frida-server` | server 未启动或端口被占 | 传输端口、服务状态与占用情况是原修复观察项 | 10min |
| Hook 不生效 | App 启动太快，Frida 注入晚了 | 原记录对比早期与进程启动后观察的时机差异 | 15min |
| Hook 后部分请求仍 SSL 错误 | 应用同时用了 OkHttp 与原生 HttpsURLConnection | 确认 X509TrustManager.checkServerTrusted 与 HostnameVerifier.verify 是额外独立路径 | 20min |
| 反检测：App 检测到 Frida 后退出 | App 自检 frida-server 端口 / `/data/local/tmp/re.frida.server` | 原记录比较了不同运行时组件位置与自检特征；此处不保留替代注入方案 | 30min+ |
| ProGuard 混淆后类名找不到 | 类名变成 `a.b.c` 短名 | 在 jadx 里用 `Find Usages` 反查谁实例化 OkHttpClient.Builder | 25min |

## 能力与格式知识


- objection 的原 Pinning 描述体现常见校验接口集合；80% 是未给样本范围的源经验值，不作覆盖保证。
- frida-multiple-unpinning（源归属 WithSecureLabs）列出 OkHttp 3/4、Retrofit、HttpsURLConnection、Conscrypt、Cordova；这是历史接口集合，具体版本与自定义校验仍是条件。
- MEDUSA 在原素材中表示模块化动态观察；模块覆盖、载入时机和实际网络结果决定观察是否完整，不规定依赖。

## 技术机制与证据

校验输入与接口关系：


原脚本定位 OkHttp 3/4 CertificatePinner 的 hostname / peer-certificate 参数、Conscrypt TrustManagerImpl.verifyChain 的证书链/主机/客户端身份/OCSP/SCT，以及 HostnameVerifier 的名称校验。三者分处 Pinning、信任链、主机名边界；原示例覆盖不完整且含空 catch，不应据“无错误”证明全部请求覆盖。本页保留类、方法、输入与证据，不复制返回值替换代码。


源一键模式的技术目标：


原 objection 示例将运行时观察与 Android 应用身份关联；知识目标是记录证书错误的具体层与变化前后请求结果，不保留一键调用。


## 可迁移经验


- OkHttp 3/4、Conscrypt、自定义 TrustManager、Flutter/boringssl 的校验位置并不相同；失败可来自接口覆盖、签名校验、原生层或进程时机。
- 证书校验变化、请求可见与业务响应是不同结果；知识记录保留对应版本、校验路径和失败证据。

## 算法与结构

**通用绕过流程**：


证书 Pin、Hostname 与 TrustManager 的错误并不等价；静态类/调用关系与实际网络路径需要相互印证。OkHttp 3/4、Conscrypt、自定义 TrustManager 和 Flutter/BoringSSL 是独立覆盖分支，libflutter.so 的 ssl_verify_peer_cert 是原文历史观察线索，不形成绕过脚本或安装依赖。


## 环境信息
- Kali / Windows + adb + frida-tools 16.x
- 目标 Android: 8-14（不同版本 TrustManagerImpl 路径不同）
- 注入方式: USB 调试 + frida-server / 或 zygisk-frida 隐藏

## 来源与脱敏说明
本条目为种子数据，基于公开技术模式编写，不涉及真实目标。包名 `com.target.app` 为占位符。
