# 分配器、链表与对象生命周期

来源：skills/pwn-chain/references/heap-pwn.md

## 适用范围与概念

保留 glibc 分配器的缓存、链表、版本差异、safe-linking、bin 类别、House 名称和无 hook 场景的对象知识。原菜单交互和利用模板被转写为内存状态与约束，不保留 payload 或工具调用。

## 核心原理与技术细节

### glibc 版本差异

原简表将≤2.26 归为无 tcache、2.27 归为 tcache 引入、2.29 归为 unsorted unlink 加固、2.31 归为 tcache key 检查、2.32 归为 safe-linking、2.34 归为移除 malloc/free hooks、2.35+ 归为进一步加固。此处保留原分组作为材料背景，尤其 tcache 引入边界等不能未经核对用作准确版本结论；发行版回补和实际实现也可能改变行为。

`__free_hook`/`__malloc_hook` 的可用性、fd 编码和 unlink 检查，比单纯发行版标签更直接影响解释。原“FILE 路径仍可用”并非对所有构建的能力承诺。

### tcache poisoning：原 2.27–2.31 场景

tcache 是每线程按 size class 维护的单链缓存，fd 连接后续对象。旧版 double-free 检查范围、链表头与遍历检测存在差异，原材料以此解释为何同样重复释放在不同版本结果不同。失效对象中链字段仍可改变时，后续分配的返回位置可能受影响，但这一结论还依赖大小类别、检查和可控写范围。

### tcache 模板的内存结构

原例希望关联三类状态：tcache 容量满后的其他 bin 状态、unsorted 残留的 main_arena 指针、失效对象中 fd 与后续返回位置。padding 邻居用于解释为什么对象没有合并到 top。原 main_arena+96 和 `0x3ebca0` 是特定 glibc 2.27 amd64 例，不能跨版本套用。

源注释声称“大于 tcache 范围、>0x408”，实际代码使用 0x80 并通过数量填满缓存，注释与代码并不完全一致；本页保留这项风险，不把它整理成可直接运行的完整算法。原 hook 指向与执行触发链被转写为“特定版本的回调目标与对象内容有关”，不复制写入配方。

### safe-linking：编码关系

原文给出的纯算法为：

```text
PROTECT_PTR(position, pointer) = (position >> 12) XOR pointer
```

position 是保存链指针字段的位置，pointer 是被表示的指针；二者角色不能颠倒。位移 12 将位置的页级信息混入链值，堆位置未知时不能把原始指针直接当编码值。`heap_base = observed_heap_address AND NOT 0xfff` 是原页对齐计算例，只有观测值确实属于相应堆映射时才有意义。

原 safe-linking 绕过代码不复制，保留字段位置、编码、解读与地址身份的知识关系。unsorted 或 tcache 残留可能提供不同类型地址，不能将任意地址都当成 heap base。

### fastbin attack：传统场景

fastbin 也是单链结构，大小类别与 chunk 元数据一致性影响返回。tcache 优先时，fastbin 是否参与取决于缓存状态。原双重释放例表现为 A→B→A 的循环，这说明对象身份重复；伪造位置还需有相符 size 字段。原 0x60 请求、0x70 chunk 和地址+8 的 size 位置是源中特定布局例，不能扩展为所有架构通用偏移。

### unsorted bin attack：旧版场景

原材料描述 unlink 对 fd/bk 的一致性依赖，以及 main_arena 相关指针写入的副作用；2.29 的 `bck->fd == victim` 检查用于说明旧式假设为何失效。global_max_fast 的含义关联哪些大小走 fastbin，修改该值与普通数据覆盖的影响不同。原 main_arena+88、target-0x10 等写入配方不保留为触发代码，指针一致性和版本条件保留。

### large bin attack

large bin 多出 fd_nextsize/bk_nextsize，除普通双向链外还表达大小关系。原 2.32 大小检查、global_max_fast、`_IO_list_all` 与 House of Husk 的关联，说明元数据一致性与全局对象后果需要共同解释；本页不把原版本概括当作无条件可用性结论。

### House of XXX 速查

| 历史名称 | 原版本标签 | 原核心知识主题 |
| --- | --- | --- |
| Force | ≤2.28 | top chunk 大小影响后续分配位置 |
| Lore | 原称全版本 | small bin 链与返回对象一致性 |
| Orange | 2.23–2.30 | unsorted、`_IO_list_all` 与 `_IO_flush_all_lockp` |
| Roman | 2.23–2.26 | 地址低位不确定性、fastbin 与 malloc hook |
| Einherjar | 原称全版本 | prev_size、PREV_INUSE 与向后合并 |
| Botcake | 2.27+ | tcache/unsorted 组合与重复释放检查 |
| Husk | 2.27+ | printf 的 `__printf_function_table` 等表 |
| Cat | 2.34+ | `_IO_wfile_seekoff` 与虚表关系 |
| Apple | 2.34+ | `_IO_wfile_jumps`、上下文恢复关系 |

这些名称标记源文已有概念，没有新增攻击路径。原“全版本”或“2.27+”不能替代具体实现、保护和对象可控性证据。

### 原通用四步的知识关系

堆地址解释位置相关编码，库地址解释 arena 及符号偏移，对象或回调控制解释潜在控制流后果，最后实际观察说明前置假设是否成立。原 base 换算为观测 arena 地址减同版 arena 偏移。原 setcontext+53 与 heap 中 ROP 的示例属于特定版本，不复制为执行链或固定偏移。

### libc 2.34+：FILE 结构背景

`_IO_2_1_stdout_`、`_IO_2_1_stderr_`、`_IO_file_xsputn`、`_IO_OVERFLOW` 与虚表是原 stdio 路径中的对象/调用标记。FILE 对象的字段、实际分派和输出/刷新/异常路径决定能否发生间接调用。源“任何 puts/printf/abort/exit 都冲刷 stderr”的概括并非所有路径的保证；实际触发条件需要目标证据。

保留 FILE 字段与虚表语义，不复制伪造 vtable、函数地址与字符串排列。不存在 hook 并不自动意味着某条 FILE 路径必定可控。

### Exit handlers 与 tls_dtor_list

`__run_exit_handlers` 遍历 `__exit_funcs` 中的回调对象，`__call_tls_dtors` 关联线程局部析构链。函数、参数、链节点和退出时机是不同字段；指针保护/PTR_DEMANGLE 使内存值不一定是明文函数地址。源例提到 TLS `fs:[0x30]` guard，这是特定布局背景，不作为跨架构固定位置。

tls_dtor_list 与通用退出链结构相似不等于每个退出路径都执行它；原“更通用”的评价保留为历史意见，不提供伪造节点或回调 payload。

### pwndbg/GEF 堆观察项

arena 内 chunk 清单、tcache/fastbin/unsorted/small/large 分类、单个 chunk、疑似伪对象位置和可视化布局，对应不同观察粒度。原命令不保留，但这些对象和地址/大小/链接字段都需要在证据中可定位。

### heap menu 模板的协议结构

原菜单含 choice、idx、size、data 四类字段及 add/free/show/edit 四类操作。提示符、索引生命周期、请求长度与返回数据长度共同解释一次交互状态；本地和远端身份不同。知识页保留这种协议与对象关系，不复制网络连接或菜单操作代码。

## 观察项与判断依据

原每 size class 容量 7、用户请求加 0x10 头再按 0x10 对齐，以及下一 chunk 的 prev_size 与可用区关系，都是源中的 amd64 常见布局简化。它们需结合目标配置与实际 chunk 元数据，不能作为全架构常量。缓存已满、fastbin/unsorted 分流和合并是不同状态，不能只凭释放次数判断。

## 故障定位与经验

不同 libc 会改变链检查与 hook 可用性；fork 模型可能改变 brk/mmap 和对象位置；padding 邻居影响合并；意外 arena 指针残留会污染 show 输出。safe-linking 常见错误是把目标地址当作 position；另一个风险是把库残留地址误当堆地址。源模板还有 edit 未给出实现等不完整点，本页不将其补成运行程序。

## 证据与报告结构

记录 libc/build、arena 与缓存类别、请求/实际 chunk 大小、链字段和编码、对象分配/释放/重用时间线、邻居及合并状态、可观察返回数据、版本限制和源简化风险。地址和偏移仅作为对应构建的证据，不移植成通用写入配方。

## 相关知识与维护说明

栈状态见 [栈与 ABI](stack-pwn.md)，slab 与内核保护见 [内核知识](kernel-pwn.md)。原算法公式保留，工具安装、调用代码和完整 payload 不随知识副本保留。
