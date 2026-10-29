# Go 二进制逆向指南

来源：skills/reverse-engineering/go-reverse.md

本文保留原资料的技术主题与判断依据。历史名称和版本信息用于解释概念，不规定工具依赖；下文只描述知识、输入输出及可观察证据。


> Go 编译的二进制有独特的挑战：静态链接导致体积巨大、函数数量上万、字符串格式特殊、符号 strip 后恢复困难。
> 本文档覆盖运行时结构、恢复依据和场景知识。

---

## Go 二进制的特征识别

Go 编译产物的判断依据：

Go 二进制常包含 runtime.*、go.buildid、GOROOT/GOPATH 字符串；静态链接 runtime 会增大体积。原资料以 C Hello World 约 20KB、Go 约 2MB 作数量级示例，实际大小取决于版本、平台和构建选项。
常见特征：
- 包含 `runtime.` 前缀的大量函数
- 包含 `go.buildid` section
- 包含 `GOROOT`、`GOPATH` 路径字符串
- 函数数量 5000-50000+（包含整个 runtime 和标准库）

---

## 历史项目与技术能力

### 符号恢复

| 工具 | 用途 | 历史标识或知识范围 |
|------|------|------|
| **GoReSym** | Mandiant 出品，解析 Go 符号信息（pclntab/moduledata） | `github.com/mandiant/GoReSym` |
| **GoResolver** | Volexity 出品，用 CFG 相似度自动去混淆 Garble 二进制 | `github.com/volexity/GoResolver` |
| **redress** | 分析 stripped Go 二进制，恢复类型/接口/包结构 | `github.com/goretk/redress` |
| **GoStringUngarbler** | Google 出品，专门恢复 Garble 混淆的字符串 | `github.com/mandiant/GoStringUngarbler` |

### IDA 插件

| 工具 | 用途 | 历史标识或知识范围 |
|------|------|------|
| **go_parser** | IDA 插件，解析 moduledata/pclntab/类型信息 | `github.com/0xjiayu/go_parser` |
| **IDAGolangHelper** | IDA 脚本集，解析 Go 类型信息 | `github.com/sibears/IDAGolangHelper` |
| **AlphaGolang** | SentinelLabs 的 IDAPython 脚本集 | `github.com/SentineLabs/AlphaGolang` |
| **IDA 9.2+ 原生支持** | Hex-Rays 官方 Go 反编译改进 | `hex-rays.com/blog/stop-guessing-and-start-going` |

### Ghidra 插件

| 工具 | 用途 | 历史标识或知识范围 |
|------|------|------|
| **Ghidra + GoReSym 输出** | GoReSym 历史能力： 导出符号后导入 Ghidra | 符号与地址的对应 |
| **golang_loader_assist** | Ghidra Go 加载辅助 | 社区脚本 |

### 独立分析工具

| 工具 | 用途 | 历史标识或知识范围 |
|------|------|------|
| **gore** | Go 逆向工程库（redress 的底层） | `github.com/goretk/gore` |
| **garble** | Go 混淆工具（了解它才能对抗它） | `github.com/burrowers/garble` |

---

## Go 二进制的关键结构

### pclntab (PC Line Table)

Go 二进制中最重要的结构，包含：
- 所有函数名和地址映射
- 源文件路径
- 行号信息
- 栈帧大小

即使 strip 了符号，pclntab 通常仍然存在（Go runtime 依赖它）。

pclntab 可以通过魔数、布局、函数地址范围和 moduledata 引用识别。原资料列出的候选值为 0xFFFFFFF0、0xFFFFFFFB，分别附有 Go 1.16+、Go 1.18+ 标签；这些版本对应关系存在待核对风险，此处保留原记录并标记，不把它用作解析规则。版本判定应与字段布局、端序和构建信息一致。
### moduledata

包含：
- pclntab 指针
- 类型信息表
- itab（接口表）
- 全局变量信息

### 字符串格式

Go 字符串不是 C 风格的 null-terminated，而是 `(pointer, length)` 结构：

```text
C 字符串:   "hello\0"
Go 字符串:  struct { ptr *byte; len int } → ptr 指向 "hello"（无 \0）
```
这导致 IDA/Ghidra 默认的字符串识别会漏掉大量 Go 字符串。

**解决方案**：
- 以指针与长度识别 Go 字符串
- 字符串列表保留地址和长度
- runtime 字符串相关引用与使用点相互定位

---

## 不同构建场景的恢复依据

### 场景 1：未 strip 的 Go 二进制

未 strip 时，函数名、类型、源文件路径和包结构能够直接参与归类；runtime.* 与标准库占据大量函数，业务定位可依据 main.main、main.*、第三方包与调用关系。符号信息的价值在于地址与语义的对应，不在于某种导出或导入命令。
### 场景 2：strip 后的 Go 二进制

strip 后常仍保留 runtime 所依赖的 pclntab。可恢复材料包括函数名/范围、源路径、包结构、类型和接口。某一解析方式失败可能是版本或结构差异，不等于材料不存在；moduledata 引用、函数地址合理性和包调用关系提供交叉证据。
### 场景 3：Garble 混淆的 Go 二进制

Garble 可随机化函数名（如 main.main 变为随机标识）、加密字符串、移除源文件路径并混淆包名。恢复线索包括 CFG 签名的标准库相似性、字符串解码模式、runtime 参数和返回值，以及同版本参考构建中的 runtime 结构。历史名称 GoResolver、GoStringUngarbler、Frida/dlv 与 binary-diff 分别承载这些思路，不定义依赖。
### 场景 4：CGo 混合编译

CGo 的知识边界是 Go 与 C 两套调用约定、栈和类型表示的衔接。_cgo_*、_cgo_topofstack、crosscall2 等桥接函数可以标记边界；Go 侧的 runtime 元数据和 C 侧的常规结构证据需分别解释，再通过参数与返回值相连。
---

## 符号与结构化输出

可观察产物包括函数名与地址表、类型、接口、包结构、源文件完整路径、Go 构建版本、CFG 归属候选和字符串解码结果。跨版本符号迁移还需记录匹配置信度；生成脚本或二进制的文件名不是技术结论。
---

## 静态分析表示的完整性

分析表示需要架构、pclntab、moduledata、类型、Go 字符串以及 runtime/业务归属保持一致。源流程中加载、插件、菜单和过滤操作转为这些结果：可解释的函数边界、已标注的字符串长度、类型恢复、main.main 入口及其业务调用关系。
---

## 常见陷阱

| 陷阱 | 说明 | 解决 |
|------|------|------|
| 函数太多看不过来 | Go 静态链接导致 5000-50000 函数 | 按包名区分 runtime、标准库与业务逻辑 |
| 字符串识别不全 | Go 字符串不是 null-terminated | 结合长度字段和交叉引用恢复 |
| 反编译结果难读 | Go 的 defer/goroutine/interface 让伪代码复杂 | defer、goroutine、interface 与真实运行状态的对应 |
| Garble 混淆 | 函数名/字符串全部随机化 | CFG 匹配与字符串语义恢复 |
| 版本差异 | 不同 Go 版本的 pclntab 格式不同 | 元数据布局与目标 Go 版本一致性；原项目覆盖范围为历史描述 |
| CGo 边界 | Go 和 C 代码混合 | 识别 _cgo_* 函数作为分界线 |

---

## 与相关方法的关系

| 技术问题 | 对应知识对象 |
|------|--------|
| IDA 深度分析 Go 二进制 | Go 元数据与反编译结构对照 |
| Ghidra 分析（免费） | 元数据恢复出的函数、类型与静态表示对应 |
| 快速侦察 | 字符串与文件结构画像 |
| 动态 Hook | Go runtime 调用、goroutine 和参数/返回状态 |
| 跨版本对比 | 旧版有符号信息与新版结构对照 |
| Garble 去混淆 | CFG 匹配与字符串语义恢复 |
