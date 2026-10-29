# 编译语言与运行时结构

来源：skills/reverse-engineering/languages-compiled.md

## Go Binary Reversing

### Recognition

go.buildid、runtime.gopanic/runtime.*、main.main、GOROOT/GOPATH 和 /usr/local/go/src 等路径是线索。静态链接使小程序也可能达 MB 级，但大小不是单项判据。main.main 是业务入口线索，实际系统入口还包括 runtime 初始化。

### Symbol Recovery

strip 后仍可存在函数名、地址、类型、接口和源路径。原 GoReSym JSON 的 UserFunctions、Start、FullName 代表地址—名称映射；golang-loader 与 redress 的历史能力包括字符串引用、接口表、类型、包和源树。是否可恢复取决于元数据版本和结构，不保留固定优先级。

### Go Memory Layout

在典型 64 位语境下，string 为 `{ptr,len}`（16 字节），slice 为 `{ptr,len,cap}`（24 字节），interface 为类型/itab 与 data 两个指针（16 字节）。map 与 channel 在原资料中分别指向 runtime.hmap/hchan；布局可能随版本演进。字符串通常不以 NUL 结束，长度为字节数；不能以 C 字符串扫描替代。

### Goroutine and Concurrency

runtime.newproc/newproc1 关联创建；chansend1、chanrecv1、selectgo、closechan 分别关联发送、接收、选择和关闭。goroutine 栈、调度关系和阻塞通道决定执行顺序，不能只看宿主线程栈。

### Common Go Patterns

Defer 注册与退出调用常按 LIFO，可能包含清理和结果清零；较新编译器也可内联处理 defer。多返回值中的 error 与 nil 比较要依据实际 ABI，源里“error 总在 rax”的注释不宜套用。concatstrings、fmt.Sprintf 与 %s%d/%x 常量说明格式构造；crypto/aes、crypto/cipher、crypto/sha256、encoding/hex/base64、net/http、net.Dial、bufio.NewReader、os.Open、io.ReadAll/ReadFile 分别提供能力线索。

### 综合结构与 embed.FS

架构、符号、main.main、GoString 双字段、比较、密码调用与嵌入资源构成完整证据。Go 1.16+ 的 embed.FS 可以留下 ZIP 的 PK 或 PNG 等原始文件签名；数据签名仍需关联实际引用与边界。原“恢复 90%+ 名称”是历史经验，不代表任意样本。

### UUID 与身份绑定案例（BSidesSF 2026 see-two）

构建信息中的链接变量赋值可使 UUID 进入固定长度 backing array；字节串、指针与长度之间的关系是二进制数据知识。原案例中 mTLS 证明连接方持有凭据，而应用 UUID 与证书是否绑定是另一层身份约束。相关客户端表、文件存储和注册响应是原叙述中的业务对象。本页只保留身份绑定差异、内嵌值来源和数据长度，不复制 UUID 修改、注册、API 枚举、文件下载或凭据。

## Rust Binary Reversing

### Recognition

core::panicking、/rustc/<commit>/library 路径、.rustc、Rust 模块路径与较大静态产物是线索。_ZN 是旧式符号形态，不能把所有 Rust 都认定为同一 mangling；实际构建可能使用不同方案。

### Symbol Demangling

符号解析可恢复 crate、模块、类型与函数名，rustfilt/c++filt 是原资料历史名称。解析成功仍不证明泛型实例等价或函数边界完整，名称需与地址关联。

### Common Rust Patterns

Option/Result 逻辑上有变体标签与载荷，但 niche optimization 可消除显式标签，0/1 含义和布局不能从源伪结构泛化。Vec 和 String 常有指针、容量和长度三字段，&str 是指针与长度；具体字段顺序不是稳定 ABI。String::from 与 from_utf8 可提示分配和 UTF-8 检查。

iter/map/filter/collect 可能融合为带内联闭包的紧密循环；单态化为不同类型生成相似函数。panic、unwrap 错误、文件路径和行号可揭示语义与失败条件，但 panic 路径不一定是业务正常路径。

### 表示与规模

函数大小分布、类型相关重复实例、panic 位置与调用图支持静态归类。cargo-bloat 和社区 Rust helper 是原历史资源名，不保留安装或菜单说明。

## Swift Binary Reversing

__swift5_*、swift_* runtime 与 mangled 名称提示 Swift；解析出的类/方法/参数可支持语义命名。swift_allocObject、swift_release、swift_once 关联对象生命周期与初始化；协议 witness table 关联动态分发。原小字符串示例为最多 15 字节内联，大字符串为堆存储，但长度与标志需依据目标 ABI。参见[平台知识](platforms.md)。

## Kotlin / JVM

### JVM Bytecode

kotlin.Metadata、kotlin/jvm/internal、ClassName$Companion、data class 的 copy/componentN/toString、Intrinsics.checkNotNull 是语言线索。when 可形成 tableswitch/lookupswitch，sealed 类可能形成 instanceof 链。

协程常见 ContinuationImpl、invokeSuspend 与 label；每个 suspend 点对应状态，恢复点携带跨挂起局部值。异步执行顺序由状态和恢复调用决定，不是简单按反编译文本顺序。

### Kotlin/Native

konan、kotlin.native、InitRuntime、DeinitRuntime、CreateStablePointer 与 LLVM native 产物不同于 JVM 字节码。原文把其内存管理概括为 ARC；这一描述具有版本风险，不应覆盖所有 Kotlin/Native 版本。具体机制以产物的分配、释放和运行时结构为准。

## D Language（CSAW CTF 2016）

_D 前缀与数字长度编码（例 _D4mainQaFNaNbNfZv）、Phobos 的 _D3std、std.string/std.conv.to、GC 分配以及编译期模板 enc!(...) 可解释大量相似函数。原案例约 500 个函数，各层按循环字符与该字符序列长度 XOR；逆序撤销层时，位置周期与层次次序都需保留。算法 key 是每层参数，不保留具体密钥材料。

### Haskell STG Closures（hxp 2017、Codegate 2018）

GHC 的 STG 表示包含惰性求值、闭包与 thunk。libHSbase/libHSrts、hs_main、GHC 字符串和 Z-encoding（如 zd 表示点、zi 表示美元符）是识别线索。原寄存器例 rbx=R1、r14=R2 属于特定 GHC 平台约定。

闭包首 qword 指向 info table/代码相关结构；info table 可包含 closure 类型、布局与 SRT，具体相邻布局依构建选项。hsdecomp 是恢复伪 Haskell 和模式匹配的历史资源。原同版本 Main_main_info 参考构建案例说明已知求值器与隐藏闭包的 ABI 关系，本页不提供替换入口或执行隐藏闭包代码。

### Haskell CMM 与递归串（N1CTF 2018）

CMM/C-- 可保留 thunk、闭包与递归结构。原串为 `f(n)=s1+f(n−1)+s2+f(n−1)+s3`；完整展开呈指数增长，而长度 `L(n)=2*L(n−1)+|s1|+|s2|+|s3|` 可缓存。按段累计边界确定 offset 所属，再递归进入相应子段，无需实体化整个串。

```text
char_at(n,k)：
  n=0 时在 s0 中取值
  否则依次考虑 s1、左 f(n-1)、s2、右 f(n-1)、s3
  若 k 在某段长度内：取该段或递归；否则 k 减去段长度
```

原代码未完整处理右子段后的 s3 与边界；此处按原递归定义补齐知识表示，保留这一遗漏风险。

## .NET / C#

PE CLR header、mscoree.dll、mscorlib/System.Private.CoreLib 支持托管假说。IL、元数据、反编译 C#、混淆后的名字与源码近似需区分。ConfuserEx、SmartAssembly、Babel、.NET Reactor 是原混淆族；dnSpyEx/ILSpy/de4dot/dnlib 是历史表示载体，不规定依赖。

关键判断以 IL 和真实控制流为证据，避免生成 C# 重编译失真。IL2CPP/NativeAOT 主要为 native 产物，不能默认有 CLR/完整 IL。原 Sharp* 分析、Unity 和 .NET 专题只保留为相关领域文字；XOR/Base64/AES-CBC 案例见[格式知识](tools.md)。

## C++

### vtable Reconstruction

对象中的 vptr、表中槽位、构造/析构与间接调用支持多态恢复。64 位示例的槽位偏移 0x18 对应 3 个指针宽度，但表的地址点、RTTI 和析构槽依 ABI；源文混合 typeinfo 与方法表的排列不可照搬为统一布局。

### RTTI

type_info、类名和继承关联可支持层次恢复；仅按“数字加大写”的字符串模式并不能识别全部或唯一类型。表引用、对象偏移与真实调用关系需一致。

### Standard Library Patterns

libstdc++ 的 std::string 可含 ptr、size 与 cap/buf[16] 联合，短串常为 15 字符；vector 为 begin/end/capacity_end；map 常为含左右/父/颜色/键值的红黑树；unordered_map 常含 bucket_array、size 与负载因子。实现、架构与优化会改变具体布局。

## 证据与故障定位

报告保留语言与版本线索、运行时初始化、真实 ABI、元数据、对象与并发状态、间接调用、泛型/闭包的归属、算法与反例，以及源码近似和机器语义之间的差异。
