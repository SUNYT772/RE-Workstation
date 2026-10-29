# 跨领域分析地图

此页将原完整手册中采集、结构、行为、验证与归档的知识关系重新编排。各环节可按问题选择和往返，不是固定的 agent 阶段。

## 对象与材料

应用可能来自已安装版本、商店版本、厂商渠道或分包集合。记录版本、渠道、签名与分包关系，避免把不同材料混作同一目标。APK 的 Manifest、DEX、原生库、资源和热更新包代表不同代码与数据层。

流量的设备侧原始包、代理可见请求和协议解码结果有不同证据范围。TLS加密、证书固定、缓存、重试、压缩与多路复用可能解释观察差异；没有看到流量不等同没有联网。

## Android 结构与定位线索

Manifest 提供权限、Application、Activity、Service、Receiver与组件入口。`lib/`区分架构与Native模块；`assets/`可能含配置、加密资源、热更新；`res/raw/`可能含二进制；其他资源与游戏引擎关联。

| 问题 | 原文关键词与结构线索 |
| --- | --- |
| 密码与签名 | encrypt、decrypt、sign、key、MD5、AES、native、JNI |
| 网络与协议 | HTTP/HTTPS、api、v1/v2、OkHttp、Retrofit、WebSocket、Socket、CocosWebSocket |
| 身份与配置 | appId、appKey、appSecret、token、secret |
| 初始化与派生 | getDI0、getDA0、initKey、genKeySeed |
| 壳与检测 | DEX异常、壳库、映射扫描、调试状态与环境字段 |

这些字符串是候选入口，结合交叉引用和数据流确定语义。DEX极小、缺失或头异常，与已知库和加密资源组合起来才有较强保护线索。

## Native 语义恢复

先辨认 ELF/PE/Mach-O、位数、指令集、字节序和模块边界。静态JNI名称与`JNI_OnLoad`动态注册对应不同入口恢复问题。`dlopen/dlsym`提示动态依赖；socket/connect/send/recv提示网络；AES/SHA/HMAC/EVP类符号提示密码相关函数。

沿参数来源→变换→调用→返回或持久化追踪。256字节查找表可能是S-box，IV、分组长度、轮结构与常量需要互证；`srand(time)`等提示弱种子，不能只凭函数名确认可预测性。PBKDF2关注salt与轮次，HKDF关注HMAC派生输入和上下文。

标注保留原名称、入口地址、函数签名、参数含义与判断依据；重命名不是证明。完整常量与壳指纹见知识库相应页。

## 框架指纹与研究层次

| 框架 | 线索 | 知识关注点 |
| --- | --- | --- |
| Cocos | libcocos.so、main.js/jsc | 引擎、脚本/字节码与Native边界 |
| Unity IL2CPP | libil2cpp.so、global-metadata.dat | 元数据、类型/方法映射、原生实现 |
| Unity Mono / .NET | 托管程序集 | IL、类型系统、事件、反射和运行时 |
| Unreal | libUE4.so、pak | 原生引擎与资源关系 |
| React Native | index.android.bundle、Hermes | JS包、字节码、Native桥 |
| Flutter | libflutter.so、libapp.so | 引擎与应用快照/原生实现区分 |
| Xamarin / MAUI | Mono程序集 | 托管与平台互操作 |

框架线索可组合；混合应用可能同时包含多种运行时。

## 动态证据与观测语义

函数入口可以解释参数与调用上下文，出口可以解释返回值和副作用；线程、时间、对象生命周期与可重入性影响归因。原生指针、字符串和字节数组的长度、编码、可访问范围分别记录。符号查找可能来自模块导出、动态解析或相对偏移，不能混用绝对地址与模块偏移。

Android 的ptrace、TracerPid、maps、进程、线程、端口、加载命名空间和Java环境字段，Windows的PEB调试标志和NtGlobalFlag，属于不同检测面。观测方法可能改变目标状态，保留基线与对照。

Windows行为证据按文件（CreateFileW/WriteFile）、注册表（RegOpenKeyEx）、网络（socket/send/recv/InternetConnectW/HttpSendRequestW）、进程和内存关联。内存区域、磁盘映像、修补前后字节与模块装载地址分开记录。

## 算法与请求验证

原模板提供参数排序、排除字段、应用标识、密钥派生、摘要/HMAC、XOR和字符码数组等关系。比对同一输入的原始字节、规范化串、算法输出、序列化与最终请求。签名正确性、身份有效性、业务状态是不同判断。

HTTP 200/201与业务成功字段共同说明结果；HTTP成功但sign error/token invalid仍可能失败；401/403可能涉及身份缺失、过期或策略，不宜只归因签名。时间戳、设备与会话、服务端状态、样本版本应纳入差异解释。

## 结果沉淀

一次分析可留下对象标识、核心问题、关键发现、接口表、算法关系、时间线、失败路径与下一步证据。经验页区分单次案例与可迁移规律；知识归档保留来源和适用条件，避免把个案数据固化为通用规则。

## 原集成故障中的通用经验

插件文件存在与运行期类可发现是不同事实；相同归档文件哈希也不能证明加载布局正确。结构化消息通道混入banner或普通日志会破坏协议，标准输出与诊断输出的角色应区分。重复配置、陈旧进程与客户端/服务端状态不一致会造成不同故障表象。这些是历史工程经验，不构成此知识包的运行依赖。

来源：原 `RE-TOOLCHAIN.md`、`TOOLS.md`、总入口和路由矩阵（根文档原件保留在本地历史快照）。
