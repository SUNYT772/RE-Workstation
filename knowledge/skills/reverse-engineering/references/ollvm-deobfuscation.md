# OLLVM 反混淆 / Obfuscator-LLVM Deobfuscation

来源：skills/reverse-engineering/references/ollvm-deobfuscation.md

本文保留原资料的技术主题与判断依据。历史名称和版本信息用于解释概念，不规定工具依赖；下文只描述知识、输入输出及可观察证据。


> 面向 APK .so、ELF 与控制流平坦化的知识参考。
> 变种和项目状态是原资料标为 2026 年的历史记录，本次未核验。
> 适用：Android NDK 加固、CTF 逆向、加壳 .so 分析、商业混淆器对抗。

---

## 0. 分析条件与方法类别

不同实现对应不同分析表示；以下历史项目名解释这些表示的覆盖范围，不规定依赖：

| 表示或问题类型 | 历史项目背景 | 技术能力与边界 |
|---|---|---|
| microcode、数据流与混合执行 | obpo-plugin、d810-ng | 真实后继连接；原 obpo 描述含函数字节向服务端传递的边界 |
| 规则与 SMT 等价性 | d810-ng、D-810 | MBA、谓词、常量和不同 dispatcher 状态模型 |
| Android native 的 IR | ollvm-breaker、Binary Ninja | 加固函数的控制流和调用关系 |
| x86/x64 的符号状态 | ollvm-unflattener、Miasm | Windows/Linux 覆盖；按调用关系 BFS 解释多层控制流 |
| 路径条件与动态符号状态 | angr、Triton | 可达性和约束；缺失环境与特定陷阱可能造成失败 |
| ARM64 的具体状态 | deollvm、Unicorn | 解码状态与真实边；不能自动覆盖其他架构 |
| Goron/Arkari 间接分支 | DeObfBR | 分支目标与数据不可变性假设的关系 |
| Tigress switch/indirect | 历史 UnflattenerSwitchCase / UnflattenerTigressIndirect | 跳表来源与间接目标表关系 |

> **方法差异：** microcode、SMT、混合执行和具体模拟覆盖不同语义层；成功与否由目标变种、架构、状态模型和行为证据决定。

---

## 1. 现代 OLLVM 变种生态（2026 社区调研）

OLLVM 早已不只是 2017 年那个原始仓库。以下是原资料记录的混淆器分支，变种识别关系到解释模型，因为不同变种的对抗手段差异很大：

### 1.1 混淆器分支谱系

| 变种 | 基准 LLVM | 相比原始 OLLVM 的新增特性 | 对抗要点 |
|------|----------|----------------------|---------|
| **Obfuscator** (原始) | 3.3~4.0 | sub + bcf + fla（三大基础 pass） | 基础状态、谓词和表达式模式 |
| **Hikari** | 6~8 | Anti Class Dump、Function Call Obfuscate、Function Wrapper、Indirect Branching、Split BB、String Encryption | 字符串明文状态与间接目标关系 |
| **Hikari-LLVM15** | 15~19 | + Anti Debugging、Anti Hook、Constant Encryption | 已闭源；Constant Encryption 增加静态分析难度 |
| **goron** | 7~10 | Indirect Branch/Call/GlobalVariable | 间接目标可能依赖数据不可变性假设；需原写入关系支持 |
| **Arkari** (komimoe/Hikari) | 14~latest | 基于 goron，持续维护 | 与 goron 的间接目标和数据状态关系相近 |
| **Pluto** | 14 | MBA Obfuscation、Random CF、Split BB、**Trap Angr**（专门坑 angr） | 特定路径/指令可能使符号状态模型膨胀或失败，需区分环境问题 |
| **Polaris** (原 Pluto) | 16 | Alias Access、Indirect Branch/Call、String Encryption、Merge Function、Linear MBA、Dirty Bytes Insertion、Function Splitting、Junk Insertion | 字符串、别名、间接目标与 MBA 等多层语义同时存在 |
| **O-MVLL** | open-obfuscator | Python 驱动 pass manager；Anti Hooking、Arithmetic(MBA)、BB Duplicate、CF Breaking、Function Outline、Indirect Branch/Call、Opaque Constants | 原资料中的 Android 加固与配置驱动 pass 例子 |
| **amice** (Rust) | Rust 实现 | 全套 + VM Flatten、Instruction Virtualization、Delayed Offset Loading、Parameter Aggregation | 含 VM 化，语义对象包含 handler 与虚拟状态 |
| **VMP 系** (SmallVmp/VMPilot/xVMP/VMPacker) | — | 指令虚拟化 | **不属于普通 OLLVM fla**；核心对象为 VM 指令和状态 |

### 1.2 关键判断线索

- **Trap Angr**（Pluto/Polaris）：如果 angr 跑着跑着爆炸或路径爆炸，怀疑目标用了 Trap Angr pass → 可从 microcode 与具体状态语义交叉定位
- **Goron/Arkari 间接跳转**：如果分发器用间接跳转（BR x8 而非 switch），分析相关数据段的可写性假设，间接跳转目标常变成可静态求解
- **Constant Encryption**（Hikari-LLVM15/Polaris/O-MVLL）：常量在运行期展开；明文依赖解码 stub 的输入、状态与运算，文件常量和使用时常量需要区分
- **VM Flatten**（amice）：控制流变成 VM dispatch loop，其 handler 表、虚拟寄存器和 opcode 语义不能由普通 fla 状态模型完全表达

---

## 2. OLLVM 混淆类型检测

OLLVM 三大核心 pass 的识别特征：

### 2.1 控制流平坦化 (Control Flow Flattening / `fla`)

**IDA 视图特征：**
- 函数入口先跳转到唯一的分发器（dispatcher）块
- 主逻辑被拆成多个基本块，每个块末尾跳回分发器
- 分发器通过**状态变量**（state variable）决定下一个要执行的块
- 巨大的 `switch` 结构，各 case 之间无逻辑关联

```text
Original:             OLLVM flattened:
  block_A               entry -> dispatcher
  block_B                 ↓
  block_C              state_machine:
                         switch(state):
                           0 → block_A
                           1 → block_B
                           2 → block_C
```
**变种形态（d810-ng 识别的多种 dispatcher）：**
- O-LLVM：switch / if-chain + 状态变量
- Tigress：`m_jtbl`（switch-case）或 `m_ijmp`（间接跳转，需 `goto_table_info` 配置）
- Hodur (PlugX)：嵌套 `while(1)` 状态机，`jnz state, #CONST`，**无 switch 分发器**
- Approov：`while(v8 != C)`，状态常量集中在 `0xF6000–0xF6FFF`

### 2.2 虚假控制流 (Bogus Control Flow / `bcf`)

- 每个真实分支间插入**不可达假分支**
- 假分支用**不透明谓词**保护（条件恒真/恒假，但静态分析无法直接证明）
- 大量死代码膨胀函数体积

```text
// 整数或对应无符号位宽语义下，x(x+1) 最低位为 0
// 是否被编译器证明取决于优化与表达；源级 signed overflow 需另判
if (x * (x + 1) % 2 == 0) {
    // 真实逻辑
} else {
    // 不可达垃圾代码
}
```
### 2.3 指令替换 (Instruction Substitution / `sub`) → MBA

- 简单算术/位运算替换为等价复杂表达式（MBA, Mixed Boolean-Arithmetic）

```text
a + b  →  (a ^ b) + 2*(a & b)
a ^ b  →  (a | b) - (a & b)
a - b  →  a + (~b) + 1
```
### 2.4 快速分类表

| 混淆类型 | 结构特征 | 语义目标 |
|---|---|---|
| fla | 大型 switch 或分派器 | 状态变量到真实后继的映射 |
| bcf | 不透明谓词、假分支与死代码 | 谓词恒定性及分支可达性 |
| sub/MBA | 复杂算术与位运算 | 相同位宽下的等价表达式 |
| 多种 pass 叠加 | CFG 与表达式同时膨胀 | 假边、真实边和表达式各自的证据与依赖 |

---

## 3. 历史项目所代表的语义能力

### 3.1 obpo-plugin — microcode 与混合执行

> obpo-project/obpo-plugin（历史标识：`github.com/obpo-project/obpo-plugin`）

基于 Hex-Rays **microcode** 的伪代码优化器，使用**数据流跟踪 + 程序切片 + 混合执行（concolic）**重建平坦化控制流。原资料包含效果评价；知识判断仍以等价性和可达路径证据为依据。

**关键特性：**
- 在 microcode 层操作，直接优化反编译输出（不是改 ASM）
- 支持 IDA 7.5.0 / 7.6.0 / 7.7.0 + Hex-Rays
- 架构：ARM, ARM64, x86, x86_64, PowerPC, PowerPC64, MIPS（7.6/7.5）
- **云插件**：目标函数二进制会上传到 obpo-server 处理（核心闭源，插件免费开源）
- 原服务描述的处理超时为 600s；服务运维限制不转为 agent 规则

**技术输入与结果：**
技术输入为目标函数的 microcode、CFG 和分发块；技术结果是数据流、程序切片与混合执行支持的真实边连接。嵌套平坦化可在外层简化后暴露新 dispatcher；可读性改善需要和原路径行为相互验证。安装、上传和菜单操作不属于此知识页。
**适用场景与限制：**
- ✅ 标准和嵌套 fla，效果好
- 历史架构包含目标函数二进制上传；本页仅记录这一数据流，不要求访问
- ⚠️ 服务器可能宕机，依赖作者维护
- ❌ 不能解决所有混淆（作者明确声明）

### 3.2 d810-ng — 规则与 SMT

> w00tzenheimer/d810-ng（历史标识：`github.com/w00tzenheimer/d810-ng`）

D-810 的现代维护/重构版（Next Generation）。原项目描述为本地、开源并集成 **Z3 SMT**；技术区别在规则、状态机与等价性表示。

**核心能力（按 d810-ng README 整理）：**

*指令级优化：*
| 类别 | 说明 |
|------|------|
| MBA simplification | `(a+b)-2*(a&b) => a^b`，Z3 验证的 DSL 规则 |
| Hacker's Delight | 位运算等价（来自 Hacker's Delight 一书） |
| O-LLVM patterns | Obfuscator-LLVM 专用 MBA 模式 |
| Constant folding | 22 条常量简化规则 |
| Predicate simplification | 不透明谓词去除（setz/setnz/lnot/smod） |
| Z3 rules | 模板匹配失败时用 SMT 求解 |
| Hodur-specific | PlugX (Hodur) 恶意软件的 MBA 模式 |

*控制流 Unflattener（按目标混淆分类）：*
| Unflattener | 目标 | 说明 |
|------------|------|------|
| `Unflattener` | O-LLVM | 标准 switch/if-chain + 状态变量 |
| `UnflattenerSwitchCase` | Tigress | Tigress switch-case 分发（`m_jtbl`） |
| `UnflattenerTigressIndirect` | Tigress | Tigress 间接跳转（`m_ijmp`），需 `goto_table_info` 配置 |
| `HodurUnflattener` | Hodur (PlugX) | 嵌套 `while(1)` + `jnz state, #CONST`，无 switch |
| `BadWhileLoop` | Approov | `while(v8 != C)`，状态常量在 0xF6000–0xF6FFF |
| `UnflattenerFakeJump` | 通用 | 去除恒真/恒假的条件跳转 |
| `SingleIterationLoopUnflattener` | 残留 | 清理 `INIT == CHECK` 且 `UPDATE != CHECK` 的单次循环 |
| `UnflattenControlFlowRule` (实验) | 通用 | 基于 path emulation 的 CFG unflattener |

**技术输入与结果：**
规则覆盖的观察目标包括 MBA 等价式、常量折叠、不透明谓词和分发器状态转移。规则匹配结果应有位宽与路径语义支持；SMT 可用于检验模板无法直接证明的表达式等价性。
**与 D-810 原型的历史差异：**
- 原版 D-810 已较少维护
- d810-ng 有 CI 测试、重构代码、新增 Tigress/Hodur/Approov 专用 unflattener
- 集成 Z3，模板匹配失败时回退到 SMT 求解，原资料据此评价覆盖改善，未提供统一成功率口径

### 3.3 ollvm-unflattener — Miasm 符号执行，纯脚本

> cdong1012/ollvm-unflattener（历史标识：`github.com/cdong1012/ollvm-unflattener`）

基于 **Miasm** 符号执行引擎，不依赖 IDA/BN，纯 Python 命令行。

**特性：**
- 用 Miasm 符号执行恢复原始控制流（区别于 MODeflattener 的纯静态方法）
- **BFS 多层处理**：自动跟随目标函数的调用，递归去混淆
- 支持 Windows/Linux x86/x64
- 输出去混淆后的新二进制

**技术输入与结果：**
方法核心是将机器指令提升为 IR，以符号状态恢复真实后继，再按调用关系进行 BFS 多层归并。输出可以表达新的控制流关系或去混淆表示；平台和架构边界是 x86/x64、Windows/Linux 的原资料覆盖范围。
**适用：** 无 IDA、目标 x86/x64、需要批量脚本化处理。

### 3.4 ollvm-breaker — Binary Ninja 实战

> amimo/ollvm-breaker（历史标识：`github.com/amimo/ollvm-breaker`）

使用 **Binary Ninja** 去平坦化，仓库自带 Android 加固样本 `libvdog.so` 作为测试用例，已修复 JNI_OnLoad、crazy::GetPackageName、prevent_attach_one 等函数。

**适用：** Binary Ninja 用户、Android .so 实战。

### 3.5 deollvm — ARM64 Unicorn

> GeT1t/deollvm（历史标识：`github.com/GeT1t/deollvm`）

基于 **Unicorn** 的 ARM64 OLLVM deflat。无 IDA 时处理 ARM64 .so 的相关方法。

### 3.6 DeObfBR — BR 混淆专项

> Mrack/DeObfBR（历史标识：`github.com/Mrack/DeObfBR`）

专门去除 **BR 混淆**（间接分支混淆，Goron/Arkari 风格）。

**数据不可变性假说：** 原资料把 Goron/Arkari 的部分间接目标简化与数据段只读联系起来。这里保留的是分析模型对数据是否变化的假设：目标表若在相关路径保持不变，间接值可更易求解；实际存在写入时不得忽略其效果。改变内存权限不等同于证明数据不变，也不是本页的执行要求。

### 3.7 angr — 符号执行通用框架

通用符号分析承载的是 CFG、函数边界、状态变量与路径条件。目标函数地址只用于限定分析对象；归一化结果需要与原始可达路径和副作用对照，不能由一次框架调用证明成功。
**⚠️ Pluto/Polaris 的 Trap Angr pass：** 这两个变种专门写了 trap 来坑 angr 符号执行。如果 angr 路径爆炸或异常，怀疑目标用了 Trap Angr → 可从 microcode 与具体状态语义交叉定位。

---

## 4. 场景与语义验证

### 4.1 变种与语义层的对应

分类依据包括标准 fla/bcf/sub、Pluto/Polaris 的符号执行陷阱、Goron/Arkari 的间接目标、Tigress switch/indirect、Hodur 无 switch 状态机和 amice VM handler。方法类别包括 microcode 数据流、程序切片、符号执行、具体状态模拟与虚拟机语义恢复。

分层依赖关系是：不透明谓词的简化可减少假边；真实状态转移支持去平坦化；MBA 简化改善表达式。原文常用 bcf→fla→sub 的经验顺序保留为依赖解释，不约束固定 agent 阶段。验证证据包括函数体积变化、CFG 从放射状转为链/树形、关键输入输出和状态序列的一致性。
### 4.2 Android NDK .so 脱密专项

Android NDK 编译的 .so 经 OLLVM 加固是 APK 逆向最常见的场景。

**分析对象：native .so**
Android NDK 的分析对象是 APK 中与目标 ABI 一致的 native .so；来源可为包内库或已有文件副本。文件归属、ABI 与样本版本决定后续地址和调用关系，不保留设备提取命令。
**变种证据**
结构线索：.text 异常大而函数少可能说明膨胀；巨大 switch 关联 fla，不可达分支关联 bcf，复杂算术关联 sub/MBA，BR x8 关联间接目标，while(1) 与 jnz state 关联 Hodur 风格。单一线索不是变种定论。
**分层语义**
Android .so 的分层知识同样是谓词可达性、真实 CFG 与 MBA 等价性。可观察结果分别为假边减少、dispatcher 状态映射恢复、复杂表达式化为更简单且等价的位向量式。
**状态序列验证**
动态状态证据包含模块加载基址、dispatcher 的模块内偏移、state 所处寄存器或栈槽、每次进入时的值和对应后继块。源示例假定 state 在 x8，但实际位置应由数据流证明；状态序列用于检验静态恢复的边，而不是修改状态制造路径。
### 4.3 CTF 场景的启发与故障定位

案例中的体积启发式及失败定位：

大函数可以作为混淆定位候选；原例按函数大小观察前 5 个，仅是启发式。CFG 与函数边界、体积、正常运行路径和异常位置共同说明候选是否确实被混淆。符号执行异常可能来自环境、未支持指令或 Trap Angr，不能仅由失败判定混淆器品牌。
---

## 5. MBA 表达式简化

### 5.1 常见 OLLVM MBA 模式

```text
# 这些等式是 OLLVM sub pass 生成表达式的化简目标
"(a | b) + (a & b)"        # → a + b
"(a | b) - (a & b)"        # → a ^ b
"(a ^ b) + 2*(a & b)"      # → a + b
"(a | b) & ~(a & b)"       # → a ^ b
"~(~a & ~b)"               # → a | b (De Morgan)
```
### 5.2 表达式表示与验证方式

| 工具 | 方式 | 适用 |
|------|------|------|
| **d810-ng MBA simplifier** | IDA 内批量，Z3 验证 | 反编译 IR 中的规则简化 |
| **SiMBA** （表达式简化研究项目） | 命令行/库 | 纯表达式化简，批量处理 |
| **Arybo** | 符号位向量 | 大量 MBA 表达式 |
| **Z3 直接求解** | SMT | 最通用，模板匹配都失败时 |

纯表达式目标是把 `(a|b)+(a&b)` 与 `(a^b)+2*(a&b)` 等式归并为 `a+b`。验证对象是相同位宽下的语义等价，而不是字符串形式相似；溢出与有符号解释应在证据中明确。
---

## 6. 完整案例的证据结构

原整套脚本承载的技术材料为文件/架构画像、OLLVM 变种假说、MBA/谓词/分发器规则结果、嵌套 dispatcher、目标架构覆盖、去混淆前后表示与行为差异。保留这些材料即可复核结论；安装、云服务调用和固定工具优先级不保留。
---

## 7. 常见失败原因与核对依据

| 问题 | 可能原因 | 核对依据 |
|------|------|---------|
| angr 路径爆炸/异常退出 | Pluto/Polaris 的 **Trap Angr** pass | 比较 microcode 推理与具体状态语义 |
| obpo-plugin 联不上 | 服务器自费维护，可能宕机 | 区分服务可用性与本地语义恢复结果 |
| Goron/Arkari 间接跳转 deflat 失败 | 分发器用 BR x8 而非 switch | 核对相关数据的不可变性与间接分支目标 |
| d810-ng 处理后函数仍乱 | OLLVM 自定义了 pass 参数/seed | 不透明谓词是否遮蔽真实边，以及状态映射是否覆盖实际后继 |
| 嵌套 fla（多层平坦化）一次没清干净 | obpo/d810-ng 单次只清一层 | 外层恢复后新暴露的 dispatcher 与内层状态关系 |
| ARM64 .so 用 deflat 报错 | 老 deflat 脚本只支持 x86 | 目标架构、指令支持与状态模型覆盖是否一致 |
| Hikari 字符串看不到 | String Encryption pass | 对照解密 stub 的输入、状态和解密后字符串 |
| amice 目标 deflat 完全无效 | 含 VM Flatten / Instruction Virtualization | handler、虚拟寄存器与内存语义超出普通 fla 模型 |
| Hodur(PugX) 样本没有 switch 分发器 | 嵌套 while(1) + jnz state | 匹配无 switch 的 Hodur 状态模型 |
| Approov 样本状态常量看不出规律 | 常量集中在 0xF6000–0xF6FFF | 匹配 Approov 的状态常量和 while 结构 |
| 敏感样本误用 obpo | 二进制上传云服务 | 关注样本字节是否离开本地这一数据边界 |
| Frida hook OLLVM 函数卡死 | 状态变量被改导致无限循环 | 核对 dispatcher 状态是否被观察过程改变以及循环是否有界 |

---

## 8. 历史项目的表示覆盖

| 历史名称 | 表示层 | 保留的技术范围 |
|---|---|---|
| obpo-plugin | microcode + concolic | 程序切片、数据流与分发边；原服务架构含外部处理边界 |
| ollvm-breaker | Binary Ninja IR | Android .so 的控制流 |
| ollvm-unflattener | Miasm 符号状态 | x86/x64 与 BFS 多层关系 |
| d810-ng | microcode + SMT | MBA、谓词和多种 dispatcher |
| DeObfBR | 间接分支 | Goron/Arkari 的目标表达式 |
| IDA_Ollvm-unflattener | Miasm 的插件表示 | 同类语义在另一界面中的封装 |
| deollvm | 具体 CPU 状态 | ARM64 的分发边 |
| angr | 符号执行 | 通用路径约束与环境/陷阱局限 |
| SiMBA | MBA 表达式 | 位向量等价化简 |
| Triton | 动态符号状态与污点 | 当前轨迹的数据依赖与条件 |

原 Stars、价格和活跃度不承载算法或状态知识，知识页不据此排序或规定依赖。

---

## 9. 历史研究与项目线索

**混淆器（用于理解对抗目标）：**
- obfuscator-llvm/obfuscator（历史标识：`github.com/obfuscator-llvm/obfuscator`） — 原始 OLLVM
- HikariObfuscator/Hikari（历史标识：`github.com/HikariObfuscator/Hikari`） — Hikari
- komimoe/Hikari（历史标识：`github.com/komimoe/Hikari`） — Arkari (基于 goron, LLVM 14+)
- amimo/goron（历史标识：`github.com/amimo/goron`） — goron
- bluesadi/Pluto（历史标识：`github.com/bluesadi/Pluto`） — Pluto
- za233/Polaris-Obfuscator（历史标识：`github.com/za233/Polaris-Obfuscator`） — Polaris (原 Pluto)
- open-obfuscator/o-mvll（历史标识：`github.com/open-obfuscator/o-mvll`） — O-MVLL
- fuqiuluo/amice（历史标识：`github.com/fuqiuluo/amice`） — Rust 实现 OLLVM passes
- lich4/awesome-ollvm（历史标识：`github.com/lich4/awesome-ollvm`） — **变种生态总览（变种资料集合）**

**反混淆工具：**
- obpo-project/obpo-plugin（历史标识：`github.com/obpo-project/obpo-plugin`） — 云端 microcode 研究项目
- w00tzenheimer/d810-ng（历史标识：`github.com/w00tzenheimer/d810-ng`） — 本地规则体系
- cdong1012/ollvm-unflattener（历史标识：`github.com/cdong1012/ollvm-unflattener`） — Miasm 纯脚本
- amimo/ollvm-breaker（历史标识：`github.com/amimo/ollvm-breaker`） — Binary Ninja
- GeT1t/deollvm（历史标识：`github.com/GeT1t/deollvm`） — ARM64 Unicorn
- Mrack/DeObfBR（历史标识：`github.com/Mrack/DeObfBR`） — BR 混淆专项
- maskelihileci/IDA_Ollvm-unflattener（历史标识：`github.com/maskelihileci/IDA_Ollvm-unflattener`） — IDA 插件版
- angr（历史标识：`angr.io/`） — 符号执行框架
- SiMBA（历史标识：`github.com/tech-srl/simba`） — MBA 化简

**学术/博客：**
- Quarkslab: Deobfuscation: Recovering an OLLVM-protected program（历史标识：`blog.quarkslab.com/deobfuscation-recovering-an-ollvm-protected-program.html`） — deflat 经典原理
- MODeflattener（历史标识：`github.com/mrT4ntr4/MODeflattener`） — 静态 deflat（ollvm-unflattener 的对照）

> 关联文档：反分析机制（模块内 anti-analysis.md） (反调试/反分析总表)、高阶语义恢复（模块内 tools-advanced.md） (高级工具集)、ELF 结构（模块内 elf-analysis.md） (ELF 文件分析)、AI 辅助逆向（同目录 ai-assisted-re.md） (AI 辅助逆向)
