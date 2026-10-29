# 基础二进制模式：状态、变换与隐式控制流

来源：skills/reverse-engineering/patterns.md

## 自定义虚拟机（Custom VM Reversing）

### 结构与语义

虚拟寄存器、内存、IP、字节码、executeIns/runvm 分发器共同决定 VM。opcode 的编号只有在该 VM 内才有意义。原例 1=MUL、2=SUB、3=NOT、4=XOR 内存、5=MOV、7=JNZ、8=输出、10=输入。操作数类型、宽度、字节序和 IP 增量与名称同样重要。

### 常见状态模式

寄存器型与栈型 VM 的状态表示不同；虚拟内存访问与宿主内存访问也需分开。字节码提升后的表示应仍可追溯到原偏移、opcode 和 handler，才能从 VM 实现中识别真正算法。

### RVA-Based Opcode Dispatching

某些“opcode”实际为 handler 的 RVA。handler 完成运算后读取下一个 RVA 并转移；模块基址与 RVA 的关系决定真实目标。链表式控制流并不一定存在普通 switch。

### 大状态机（90K+ states）

原例通过 BFS 记录 `(state,path)`，按输入字符扩展转移，达到目标长度时收集路径。状态去重是否可行取决于路径长度和历史是否影响后续；不能只凭 state 编号丢掉有意义历史。

### 未知指令集与黑盒观察（hxp CTF 2017）

变长 ISA 的对齐可通过 6–11 位候选宽度与重复形态判断；最小程序的寄存器、内存和输出差异能够支持单指令语义。原发现的编码保留如下，它是案例 ISA，不是新指令集设计：

```text
000 xxxxxxxx  jmpz    001 xxxxxxxx  jmp     010 xxxxxxxx  call
011 xxxxxxxx  label   1000 xxxxxxx  loadram  1001 xxxxxxx  saveram
110 xxxxxxxx  loadi   11100 xxxxxx  shl      11101 xxxxxx  shr
111100 not    111101 and    111110 or    111111 setif
```

指令宽度、操作数字段与分支边界相互验证。缺少 XOR/ADD 原语时，原案例用 AND/OR/NOT 表达 XOR，并用全加器逐位传递 carry；这说明高层算法不一定在低层存在同名 opcode。

```text
# XOR from AND/OR/NOT:  XOR(a, b) = (a OR b) AND NOT(a AND b)
# ADD via full-adder chains using AND/OR/NOT for carry propagation
def xor_from_primitives(a, b):
    return (a | b) & ~(a & b)

def add_from_primitives(a, b, bits=32):
    carry = 0
    result = 0
    for i in range(bits):
        ai = (a >> i) & 1
        bi = (b >> i) & 1
        sum_bit = xor_from_primitives(xor_from_primitives(ai, bi), carry)
        carry = (ai & bi) | (carry & xor_from_primitives(ai, bi))
        result |= (sum_bit << i)
    return result
```

## 反调试（Anti-Debugging Techniques）

### 常见检查

IsDebuggerPresent、PTRACE_TRACEME、TracerPid、rdtsc/time、注册表可提供环境输入；返回值后的 test/cmp 与分支才决定后果。

### 分支与干预差异

原修改寄存器例子承载“检测值—条件测试—后继”的关系。改变检测值只能说明修改后的路径，不能代替原环境或真实算法证据。

### 动态链接层（LD_PRELOAD Hook）

动态符号解析可将库入口与实际函数连接起来；RTLD_NEXT 的概念是查找下一个同名实现。直接 syscall、静态链接和早期入口不一定经过该层。本页不保留库截获代码、编译或预加载命令。

### 指令改变（原 pwntools 示例）

立即返回、NOP、返回 0 或 1 会改变调用者条件；是否保留返回值与副作用需要单独判断，单一 ret 不保证得到指定值。不保留补丁调用或字节载荷。

## Nanomites

### Linux 信号

SIGTRAP/int3、SIGILL/ud2、SIGFPE/除零、SIGSEGV/空指针可作为自定义操作入口。父子进程的跟踪关系、信号与上下文改变形成隐式计算。

### Windows 调试事件

EXCEPTION_DEBUG_EVENT 和 WaitForDebugEvent 关联异常派发；原资料的 0x1337BABE、0xDEADC0DE 是案例标记。源在此混用 Linux PTRACE_POKETEXT 的名称，应按实际平台区分，不把它当 Windows API。

### 重建依据

fork/跟踪关系、父进程事件循环、寄存器值到操作的映射和写入事件能说明“谁解释谁”。真实算法可能在处理事件的父进程，而非最显眼的子进程中。

## 自修改代码（Self-Modifying Code）

### XOR 解密块

输入字符作为 XOR 字节作用于下一代码块，解密后控制流进入它。已知入口 opcode 或序言可约束密钥，但需要多个字节验证；偶然形成合法指令不证明正确。块大小、写入次序、权限和实际后继是关键。

## 已知明文 XOR（Known-Plaintext XOR）

重复密钥假说下，`K[i mod L]=C[i] XOR P[i]`。原例尝试长度 2–32，并用前缀、提示词和可打印性筛选。密钥位未知与真实值为 0 不能共用零哨兵；应记录已知状态与冲突。可读结果只是候选，需与真实校验相符。

### 位置索引变体

`C[i]=P[i] XOR K[i mod L] XOR i`，或使用 `low8(i)`。重复密钥初期匹配、后续错位，或者已知明文推出的“密钥”随位置变化，支持位置项假说。算法上先分离位置项，再合并同一周期位的约束；高位截断需与字节语义一致。

## 混合位宽加载（Mixed-Mode x86-64/x86）

0xCB/0xCA 与可能的 0x48 前缀提示远返回；32 位区中的 psubb/pxor/paddb 与计算跳转可解释 SIMD 变换。源例 retf 消耗 EIP 4 字节和 CS 2 字节，共 6 字节。跨模式状态包含内存、通用寄存器低位、EFLAGS 和 XMM；遗漏 SIMD 或 flags 会得到错误输出。若父进程修改子进程字节，写入关系也属于初始状态。

## LLVM 控制流平坦化

### 状态形态

while 循环、多个状态常量比较、更新 state 后返回 dispatcher 可取代原 if/else/循环。原样例状态常量只是标签，不是算法常量。

### 真实边恢复

状态变量的读写、每次 dispatcher 值与后继块建立状态转移图。动态路径提供经过的边，未观察边仍需静态可达性或约束依据。详见 OLLVM 专题。

## S-Box 与密钥流生成

### Fisher–Yates / Xorshift32

从 0–255 的置换表倒序选择 j 并交换，Xorshift32 的移位 13/17/5 与 32 位掩码构成原生成逻辑：

```text
def gen_sbox():
    sbox = list(range(256))
    state = SEED
    for i in range(255, -1, -1):
        state = ((state << 13) ^ state) & 0xffffffff
        state = ((state >> 17) ^ state) & 0xffffffff
        state = ((state << 5) ^ state) & 0xffffffff
        j = state % (i + 1) if i > 0 else 0
        sbox[i], sbox[j] = sbox[j], sbox[i]
    return sbox
```

### Xorshift64* 案例

移位 12/25/27、乘数 0x2545f4914f6cdd1d 与取最高字节是识别线索。原例将乘法结果反馈到 state，下列保留的是案例算法，不应未经核对替换为某标准实现：

```text
def gen_keystream():
    ks = []
    state = SEED_64
    mul = 0x2545f4914f6cdd1d
    for _ in range(256):
        state ^= (state >> 12)
        state ^= (state << 25)
        state ^= (state >> 27)
        state = (state * mul) & 0xffffffffffffffff
        ks.append((state >> 56) & 0xff)
    return ks
```

### 识别依据

Fisher–Yates 的倒数循环、PRNG 选位与 swap 区别于普通查表。0x9e3779b97f4a7c15 是另一个常见常量线索。seed 决定状态，但实际更新、掩码、输出选择同样不可省略；具体种子或密钥不保留。

## SECCOMP/BPF

### 过滤器到约束

A=sys_number、mem[N]=A 与 A=mem[N] 的读写、比较和跳转可表达系统调用参数校验。输入变量在源例为 14 个 32 位向量；可打印范围只在真实输入满足时成立。需要保留架构号、参数宽度、分支返回和有符号/无符号语义，而不是只复制求解器调用。

## 异常处理器混淆

### RtlInstallFunctionTableCallback

动态函数表和展开信息可能在异常时触发回调；回调继续登记新信息或修改代码，使普通调用图不完整。

### VEH

AddVectoredExceptionHandler 登记的 handler 可能在故意异常处解密并改变后续 PC。异常地址、handler 顺序、解密前后字节和返回上下文共同形成控制流。

## 内存转储

### 自身地址空间材料

/proc/self/maps 与 /proc/self/mem 读取可能用于自身内存处理或转储，不能仅由读取行为认定用途。堆数据可能附加在转储末尾，文件偏移与运行时地址需要映射。

### 已知序言

原已知字节 f3 0f 1e fa 55 48 89 e5 对应 endbr64 与常见帧序言，可与相同位置的密文字节 XOR 得到局部候选。序言并不保证存在于所有编译结果，局部候选需要其他位置交叉检验。

## 独立逐字节变换

单个输入位置改变只影响对应输出，常量输入产生常量输出，支持无跨字节耦合假说。有限 0–255 映射可以形成正表和候选逆表；逆表可能一对多，位置相关性和全局状态仍需排除。

## x86-64 易错点

### Sign Extension

0xffffffc7 是一个位模式：按有符号 32 位解释为 -57，按无符号为大正数。XOR 低字节取 0xc7；32 位加法按模 2^32；movsx/cdqe 的符号扩展与写 32 位寄存器的零扩展不同。

### 循环边界更新

首次跳入循环中段与后续迭代从循环顶部进入，会使状态更新使用旧 a 而非新 a。状态依赖、读取点与增量位置应以原指令为准，避免反编译重排造成一轮偏差。

## 成对状态变换（Custom Mangle）

输入每 2 字节参与变换并更新运行状态，静态 .rodata 保存目标。逆关系需要从末状态按相反顺序撤销操作；数据字节与状态的相互依赖决定是否可独立反转。

## 位置加减（Position-Based Transformation）

PascalCTF 2026 原例偶数位置正向加 i，逆向减 i；奇数位置正向减 i，逆向加 i。字符域、字节溢出和索引起点决定实际关系。

## 十六进制字符串比较

Spider's Curse 案例把输入转为 hex 再比较常量。目标位于编码域；成对十六进制还原、大小写、前导零与长度对应原字节，而不需要假设额外密码算法。

## 信号树（Signal-Based Binary Exploration）

SA_SIGINFO、sigaltstack、handler 解码嵌入数据和重新安装信号构成节点。Node 安装两个子信号，Leaf 输出并退出；信号安装、选择边、是否退出与回溯状态可形成 DFS 树。这里描述已有控制流，不保留信号发送和截获库代码。

## 多项环境判断（Malware Anti-Analysis）

原 Carrot 案例涉及 ptrace 返回 -1、sleep(150)、cpuinfo/hypervisor、VMware/VirtualBox、getpwuid 用户名、LD_PRELOAD、风扇与硬件数量、hostname。每项观察包括数据来源、比较条件和错误路径。远端可能再次检查收集到的环境字段，因此仅改变本地分支不证明远端接受；本页不包含伪造请求、格式串替换或服务交互。

## 多阶段加载器

间接调用、ptrace 检测、XOR 解码循环和可隐藏的 int3 可能逐层出现。阶段边界、解码后内存与下一入口比文件中第一层更接近真实逻辑。末阶段连续 mov 的 32 位立即数可能按小端拼为字符串；记录字宽和次序，避免按显示的十六进制顺序拼接。

## 时间侧信道

Clock Out 案例中匹配字符使验证时间更长。相同长度、相同初态、多次观测和环境噪声决定差分可信度；一次最长耗时可能只是网络或调度延迟。保留时间与前缀的依赖知识，不保留远端连接代码。

## 多线程诱饵与信号 MBA（ApoorvCTF 2026）

线程 1 为类似 AES 的诱饵与 ud2，线程 2 在 SIGSEGV 中进行真实 MBA，线程 3 清零结果，主线程使用 rdtsc 检测。pthread_create、handler 登记和 SHA-256 常量被当查表而非哈希是识别依据。

原 40 字节关系为：`t=((7*i+0x3f) mod 256) XOR rol8(i,3)`；查表索引 `i AND 7`，字节偏移 `((i>>3) AND 3)*8`，从 8 个 SHA-256 初值提取：`6a09e667, bb67ae85, 3c6ef372, a54ff53a, 510e527f, 9b05688c, 1f83d9ab, 5be0cd19`；偶/奇位置分别读取两个交错数组。输出是 t、查表字节与目标数组字节的 XOR。具体数据数组是样本输入，不是通用算法；原常量值可在密码识别页与原来源对应。时序与清零说明事后空内存不等于没有计算结果。

## INT3 与 core 状态（Pwn2Win 2016）

变换后的故障时点可以在 core 中留下中间结果。候选输入与快照状态的关系可作为局部 oracle，但 core 内容、进程身份、时点、长度和最终成功条件必须对应。源补丁字节写入与运行循环不保留。

## 信号链（Nuit du Hack 2016）

原案例 main 对自身产生 1024 次 SIGINT，每个 handler 校验一个字符并登记下一个 handler。登记发生与校验进展有依赖，因而登记事件可提供侧信道；它不是对任意 signal 调用成立的结论。

### printf 格式串 VM（SECCON 2017）

已有 VM 把输出字符计数的低 8 位通过 %hhn 写入内存；写地址来自参数指针，写值来自累计输出计数。对数字字段归一化可区分格式族，再建立“格式族—状态更新—目标地址”映射。源例约束涉及模 256 的和与 XOR，变量为 32 个 8 位输入并按真实需求限制可打印域。

关键是累计计数、覆盖顺序、地址别名与输入变量的对应；把每个格式片段当独立常量会丢掉先前输出贡献。不新增格式串或载荷，不保留求解器调用代码。

## 证据组织

报告保留格式与架构、真实数据来源、隐式边、状态宽度、常量与样本数据的区别、算法及逆向前提、反例、候选与最终接受关系。工具、命令和某个局部成功不能代替这些证据。
