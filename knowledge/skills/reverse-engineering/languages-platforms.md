# 平台与框架案例：数据结构、接口与状态

来源：skills/reverse-engineering/languages-platforms.md

## Rust serde_json Schema Recovery

serde Visitor、字段字面量与 visit_str/visit_u64/visit_bool/visit_seq 可恢复字符串、数值、布尔与数组类型。访问者调用层次决定嵌套 schema，反序列化错误与期望字段也提供线索。源案例的字段集合包括 pascal、CTF、crab、I_、cr4bs、crabby、l0v3_、r3vv1ng_，嵌套类型与原顺序能携带信息；“拼接键得到结果”是样本属性，不是通用 JSON 规则。

## Android JNI RegisterNatives（HTB WonderSMS）

Java 的 native 声明与 System.loadLibrary 关联 `.so`。标准导出名不存在时，可能经动态登记，也可能存在其他加载机制；JNI_OnLoad 是常见锚点但不保证任意库都导出它。

JNINativeMethod 的三字段为 name、signature、fnPtr，登记同时关联 class 与 count。原 processMessage 的参数是 SmsMessage、返回 ProcessedMessage；name/signature/class 与真实函数指针之间的对应比符号字符串搜索更有意义。x86_64 与 arm64-v8a 是可对照的 ABI 产物，不规定某一架构工具优先。

## Android DEX 运行时变化（Google CTF 2017）

原 native 代码读取 /proc/self/maps 定位 DEX、改变权限、对指定字节区 XOR，导致包内静态 DEX 与实际执行不同。原示例区域长 144 字节；offset、长度、变换来源与最终字节是恢复证据，具体 XOR 参数不保留为密钥。DEX checksum 与 SHA-1 和内容一致性也需保留。

原文概括“仅 Dalvik/API<21，不适用于 ART”属于该案例实现边界，不能泛化为所有 runtime 字节码变化的定理。

### Java 与 native 校验边界（Codegate 2018）

包名、类名、方法签名与 native 符号或动态注册构成绑定。Java 层随机数、PIN、root 条件和 native 返回的实际计算属于不同层；如果 native 不重复这些条件，两层接受条件就可能不一致。保留边界和数据流，不复制创建替代工程、直接调用 native 或跳过校验的代码。

## Firebase Cloud Functions（BSidesSF 2026）

原 vinyl-drop/doremi 案例涉及登录后 UID、数值与时间戳字段，客户端 QR/购买界面、AppCheck、用户认证与服务端业务检查是不同层。google-services.json、依赖声明和函数调用点是识别线索。

客户端可构造参数这一事实不证明服务端必然接受任意值；可信结论要有 UID 来源、字段绑定、服务器验证与响应证据。原调用代码与配置值不保留，仅记录客户端边界与字段语义。

## Verilog / Hardware（srdnlenCTF 2026）

.v/.sv、always @(posedge clk)、case 状态机与移位寄存器历史是 Rev Juice 案例结构。隐藏 product 8 的条件依赖 COINS_HISTORY 抽头，而非只依赖当前余额。

原周期模型：投币 3、选择成功 7、选择失败 5、有币取消 4、零币取消 2。原抽头约束为 H[0]=1、H[7]=4、H[28]=H[33]=H[38]=6、H[63]=H[73]=2、H[80]=9，另有 `(H[19]+H[21]+H[56]+H[69]) mod 32=0`。抽头随每个时钟移动；动作的内部周期也计入历史，必须以真正状态转移解释。原操作序列属于样本答案形式，不扩展为新序列。

## 前缀哈希（Nullcon 2026）

不常见的 MD2 或自定义哈希也可以由调用名、输入前缀和输出对照识别；语言封装不改变逐前缀约束。原 2 秒超时只是防挂起的实现经验，不保留进程调用。完整关系见[算法案例二](patterns-ctf-2.md)。

## Ruby / Perl Polyglot（BearCatCTF 2026）

Ruby 的 =begin…=end 与 Perl 的 =begin…=cut/POD 有不同注释边界，因此同一文件可执行两套校验。原长度 50 的字符集约束涉及可打印域排除 ^、每字符仅一次，以及 `XOR(v,(v−16) mod 257)` 的模 257 原根性质；排序逆序计数表施加排列约束。

字符集合与排列是两类约束，应取交集后验证原检查。按原逆序表恢复时，索引定义是“左侧大于当前位置的元素数量”，不能与右侧较小计数混同。原代码的插入方向和索引需要正向计数验证，最终哈希/解密只用于核对已获候选，不保留密钥字面量。

## Electron 与 native（RootAccess2026）

resources/app.asar、Electron 文件和 package.json 表示封装；JS 中 child_process.spawn、ffi-napi 或 .node/.so/ELF/DLL 引用可指向 vault、auth、crypto 真实实现。包装层的验证流程与 native 的输入格式应关联，数据可能位于 .data/.rodata。

原示意变换使用 XOR 常量 0x42/0x7E、移位 3 或 i mod 8、循环参数字节。但代码只有右移，未包含循环移位的回卷项，因此不能把它视为一般可逆 rotate；恢复需要实际汇编和位宽。本页保留原示意的局限，不复制可运行解码脚本或 ASAR 安装命令。

## Node.js npm 内省（RootAccess2026）

混淆 dist、RC4 字符串、平坦化和死代码可能遮蔽导出对象；属性名、隐藏方法、方法返回片段、算法常量与拼接顺序是原观察对象。模块加载可能执行初始化和副作用，因此运行时发现的导出值与静态源码分别标注。原 require、直接调用、网络函数和输出代码不保留。

## Android 证书固定（h1702ctf 2017）

OkHttp CertificatePinner 属于传输层对端验证；本地 JNI 方法提供的 HTTP header 名和值属于应用凭据来源。两者边界不同。原 hName/hVal 方法仅作为数据来源名保留，不包含直接调用或凭据值，也不能由类中有方法推定服务端校验失效。

## Android TracerPid / su / 系统属性

原 ARM native 顺序检查 /proc/self/status 的 TracerPid、su 存在性和 __system_property_get 的自定义属性；这些分支共同决定后续寄存器值。静态 CFG 能表达各分支需要的值和可达性，但具体“正常值”应来自真实数据流，而非统一常量。

## Android 日志中的密码状态（HackIT 2017）

源安全消息应用记录 Curve25519 基础 agreement、逐消息 ephemeral/shared 值、message ID 和 shift counter。AES-CBC 的 IV 与短期共享值、密钥派生与基础 agreement 和累计移位相关。消息归属、计数顺序、编码、派生函数、参数长度与填充都需要记录；日志有相关字样不表示单独一个值就足够。具体密码材料和日志采集命令不保留。

## JNI 签名与 Smali 数据流（HackIT 2017）

原 .data 中 XOR 封装参数在使用前展开，说明磁盘表示和使用时表示不同；Smali 构造 POST 时哪些参数进入签名、顺序与编码是另一条数据链。完整证据应连接 native 输出、签名输入与请求字段；不保留密钥转储、参数改签、重打包、安装或请求代码。

## IBM AS/400 SAVF / EBCDIC（EKOPARTY 2017）

cp500 表示 IBM EBCDIC International。字节先按正确编码解释，再观察字符分布、大小写和下划线。源叙述 take-2-skip-2 的目标是每 4 个解码字符保留前 2 个；其示例只取每组首字符，与叙述不一致，应保留为需核对之处。过滤大写与下划线只适用于案例字符域，不能删除全部其他字节后宣称恢复完整内容。

## Intel SGX（Pwn2Win 2017）

enclave .so 仍含可分析的 x86-64 指令；ECALL 表以编号关联函数指针与 enclave 接口。远程证明、度量、ECDH P-256、CMAC-AES-128 派生与 AES-128-GCM 是原协议链。

原“仅由 enclave measurement 决定会话密钥”的说法不足：会话还依赖双方密钥材料与具体协议；原短 CMAC 示例没有完整表达 SGX 派生，不能作为规范或可用实现。知识页保留协议角色、函数表、共享值、派生与认证加密关系，不保留远端证明交互、密钥生成或连接代码。

## 证据与报告

按包/模块来源、语言层与 native 边界、绑定字段、运行时变化、原始数据与派生状态、服务端判断、时钟或计数条件、候选与反例组织。原材料中的简化、歧义和代码错误单独标记，避免将示意程序当真实协议规范。
