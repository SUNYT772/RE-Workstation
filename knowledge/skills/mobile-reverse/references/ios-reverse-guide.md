# iOS 逆向专项：包、运行时与保护

来源：skills/mobile-reverse/references/ios-reverse-guide.md

## 主题与范围

IPA 来源与加密状态、Mach-O、Objective-C、Swift、越狱检测及其他保护。所有命令与 Hook 模板转成字段、查找模型与观察点。

## 知识正文

### IPA 获取与解密

App Store 下载样本与设备上已安装应用是原文两类来源。已安装 .app 的位置关系到包与运行中二进制是否一致。App Store 二进制的加密状态与是否 FAT/多架构是不同属性，不能仅由包来源推断具体结构。frida-ios-dump、Clutch、dumpdecrypted 是原文中恢复运行时可读代码的历史名称；不包含账户、设备访问、解密调用或库注入步骤。

### Mach-O 分析

| 对象 | 保留的技术信息 |
|---|---|
| Load Commands / crypt 信息 | 加密状态及其范围 |
| 动态库依赖 | 二进制关联的库 |
| 头部 | 文件类型、架构等结构信息 |
| 内存页 | 页范围与映射信息 |
| FAT 架构切片 | 多架构集合与单独 arm64 切片的对应 |
| 导出/全部符号 | 可见符号范围不同 |
| Swift 修饰符号 | 还原模块、类型、方法和签名的线索 |
| ObjC 类与方法声明 | 类结构和 selector，不等于完整方法源码 |

源文历史名称 otool、jtool2、lipo、nm、swift-demangle、class-dump 分别对应上述视角，不作为依赖。

### Objective-C 消息传递模型

objc_msgSend 的 self、SEL/selector 与其后参数体现动态派发。源文模型保留为：类的方法缓存；类的方法列表；逐级父类；resolveInstanceMethod/resolveClassMethod 动态解析；forwardingTargetForSelector；methodSignatureForSelector 与 forwardInvocation 的消息转发。该列表用于解释可能的派发路径，不是 agent 执行阶段。

### ObjC 方法观察

“-”表示实例方法，“+”表示类方法；selector 与 implementation 关联到真正实现。原模板的 native 参数语义是 args[0] 为 self，args[1] 为 selector，args[2+] 为业务参数；对象参数和整数参数不能采用同一种解释。NSString 与 UTF8String 的关系是 ObjC 对象到文本的桥接，不保留具体调用代码。

### Swift 名称修饰

原示例符号为 $s10ModuleName5ClassC6method3argSi_tF，用于说明 $s 前缀、长度编码的模块名与类名、类型标记、方法名、参数名、返回/参数类型信息。该串是源材料的示意，未核验其长度与完整 Swift 语法，不将其解码为已验证函数。swift-demangle/Hopper 是历史还原概念；符号可读性不等于控制流已理解。

### 越狱检测类别

| 类别 | 原文完整观察项 |
|---|---|
| 文件系统 | /Applications/Cydia.app；/var/lib/apt/；/bin/bash；/usr/sbin/sshd；NSFileManager.fileExistsAtPath |
| 沙箱行为 | fork 是否成功；system 调用的结果 |
| Dyld 加载 | _dyld_get_image_count 的映像数量 |
| Scheme | cydia://；UIApplication.canOpenURL |
| 进程状态 | sysctl 的 CTL_KERN/KERN_PROC/KERN_PROC_PID；kinfo_proc.p_flag.P_TRACED |

### 统一模板的观察含义与局限

文件路径、文件可见性结果、fork 返回及加载映像数量是独立输入。源模板将大于 200 的映像数量限制为 200，这是示例阈值，不是“正常 iOS”边界。文件检测的源代码在返回回调中引用进入参数而未展示跨回调保存，本文不复制该缺口或声称其可运行；知识上保留“路径与本次返回需要对应”这一证据契约。

### 关键防护清单

| 防护 | 技术目标和观察结果 |
|---|---|
| App Store 加密 | 代码的加密状态与运行时可读表示 |
| SSL Pinning | 系统及框架信任决策；SSL Kill Switch 2 为历史名称 |
| 越狱检测 | 文件、沙箱、Scheme、加载映像与状态字段 |
| 反调试 PT_DENY_ATTACH | 启动/附加时机和拒绝附加的检测分支 |
| 完整性校验 | 原文称 MAC 检查/代码签名验证；校验输入及结果 |
| 反注入 | Mach-O __RESTRICT 段及加载约束；不复制移除操作 |
| Swift 混淆 | 修饰符号恢复与业务语义之间的关系 |
| 截图保护 | 原文列 UIScreen.mainScreen.snapshotViewAfterScreenUpdates；需核对实际接收者与调用链，单个名称不证明保护有效 |

## 观察与判断

包加密、FAT 架构、名称混淆和运行时可见性需分别描述。固定阈值、示意符号及 API 名称只是原材料线索，不能代替版本和行为证据；本文不借转写新增技术或替原代码填入实现。

## 记录与报告

记录 IPA/二进制身份、加密状态、切片架构、依赖与符号、类/selector/implementation 关系、消息派发路径、检测输入、结果与未核验条目。原参考名称为 OWASP MSTG、frida-ios-dump、The iPhone Wiki。
