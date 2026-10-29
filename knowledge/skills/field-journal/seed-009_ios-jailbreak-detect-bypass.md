# [种子] iOS 越狱检测绕过 + 抓包

来源：skills/field-journal/seed-009_ios-jailbreak-detect-bypass.md

来源性质：**[种子] 预置案例素材，非已验证的真实实战成果。** 文中结果、耗时与修复为原素材中的条件化叙述，不表示本次或真实目标已完成验证。

本页为原案例/种子材料的知识转写。保留环境、观察、失败、修复与验证条件；工具调用、安装与载荷改为技术关系。

## 场景分类
iOS 逆向 / 移动安全测试

## 目标概述
某 iOS 应用在越狱设备上启动即闪退或显示"环境异常"，需要绕过越狱检测才能进一步分析其 HTTP 请求。

## 观察、决策与验证

1. 原场景为越狱设备及动态观察环境，Dopamine / palera1n / unc0ver 是历史背景，安装源不保留
2. IPA 身份、签名/安装状态与可观察进程是不同前提，不保留安装步骤
3. 启动 App → 闪退或弹"环境异常"
4. open、stat、access、fork 的调用与启动退出时序构成检测线索
5. 常见命中：探测 `/Applications/Cydia.app`、`/private/var/lib/apt`、`/usr/sbin/sshd`、`fork()` 是否成功、`/etc/apt`
6. 原记录比较检测分支变化与启动状态，工具调用不保留
7. NSURLSession 流量、系统证书与用户信任设置影响后续网络可见性

## 失败路径与诊断

| 问题 | 原因 | 解决方案 | 耗时 |
|------|------|---------|------|
| objection 绕过后还是闪退 | App 用了 SSL Pinning + 越狱检测双重 | 两类检测分别影响运行环境与 TLS，可独立导致失败 | 15min |
| App 在启动前就检测，hook 来不及 | 越狱检测在 `+load` 或 `__attribute__((constructor))` 里 | 观察窗口需要覆盖 +load / constructor 发生时刻 | 20min |
| Hook stat 之后 App 卡住 | stat 被 hook 后部分系统调用也被影响 | 调用者归属能区分应用检测与系统本身依赖，缩小观察范围后原卡顿改善 | 30min |
| Frida-server 启动后 App 仍能检测到 | App 检测了 27042 端口与 frida 字符串 | 端口、组件名和字符串都是源记录发现的自检输入，未保留修改方案 | 25min |
| mitmproxy 装证书后仍 SSL 错误 | iOS 14+ 系统证书需要在"通用 → 关于本机 → 证书信任设置"再开一道开关 | 原修复确认了证书存在与系统信任开关是两项状态 | 10min |

## 能力与格式知识


- objection 的原模块集合涵盖 jailbreak、sslpin、clipboard 和 Keychain 观察；这些分别对应环境检测、证书校验和数据存储边界。
- r2frida 表示运行时反汇编、寄存器状态与动态观察的结合；源“更强”描述没有可比测量，不作为依赖或优越性结论。
- Hopper、IDA 与 Ghidra 在原素材中代表 Mach-O 静态结构与反编译视角，解释结果仍需版本、架构和动态证据。
- dumpdecrypted 与 frida-ios-dump 是源材料的脱壳历史比较；加密标记、映像身份和可分析代码范围才是产物判断依据。

## 技术机制与证据

越狱线索与接口机制：


NSFileManager 的 fileExistsAtPath、Cydia/MobileSubstrate/apt/sshd 等路径和 fork 的结果，分别反映文件存在性与进程能力判断。原拦截器改变返回结果以影响检测分支；知识页保留调用来源、参数、结果与控制流关系，不保留路径隐藏或 fork 替换代码。


已解密 Mach-O 产物与静态分析：


脱壳样本输出为应用 bundle 与已解密 Mach-O，后续静态分析需要维持原应用身份和架构。原文把 Mach-O 交给 jadx 的说法与文件格式不符；保留产物概念并标记该处原流程混淆，不提供调用。


## 可迁移经验


- iOS 的脱壳、越狱检测、SSL Pinning、Keychain 与 +load 时机各有独立前提；Android 的包/运行时知识不能直接代替这些平台关系。
- +load 等早期初始化时机可以解释观察为何晚于检测分支；检测函数变化与完整启动、请求结果仍需分开判断。

## 算法与结构

**iOS 安全测试速查**：


环境、脱壳结果、类层级、运行时观察、越狱/证书两类检测、证书信任设置和静态控制流是不同证据层。原八步速查改为这些互相验证的关系，不规定工具或顺序。


## 环境信息
- 越狱设备: iPhone X（iOS 16.5）+ Dopamine 1.1.7
- 主机: macOS 13+ / Kali（mitmproxy + frida-tools）
- frida-server-ios: 16.x

## 来源与脱敏说明
本条目为种子数据，基于公开技术模式编写，不涉及真实目标。Bundle ID `com.target.app` 为占位符。
