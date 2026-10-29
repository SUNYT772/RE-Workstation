# 硬件信号、特权状态与指令集

来源：skills/reverse-engineering/platforms-hardware.md

## HD44780 LCD GPIO Reconstruction（32C3 2015）

原案例从 Raspberry Pi GPIO 记录恢复 LCD 文本。信号映射涉及 RS、时钟/使能线与 D4–D7 的 4 位数据；1→0 边沿采样并合并两次半字节为一字节。RS=1 表示数据写入，RS=0 表示设置光标、清屏等命令。采样的先后半字节、时间排序、缺失边沿与初始化模式都会影响结果。

原资料称 DRAM 的地址实际关联显示 DDRAM；多行寻址不连续。其列出的区间为行0 0x00–0x27、行1 0x40–0x67、行2 0x14–0x3B、行3 0x54–0x7B；这些是地址线索，具体可视列数与行映射取决于模块，不能直接当四条互不重叠的物理行。

```text
state = 空显示、cursor 与待配对半字节
按时间读取 GPIO：
  若出现目标下降沿：提取 D4–D7
  两个半字节齐备后形成 byte
  RS=1：按 DDRAM→可视位置映射写字符并更新 cursor
  RS=0：更新命令状态、地址或清屏结果
```

最活跃引脚可作为时钟候选，RS 可从命令/数据阶段的相关性推断；它们是启发式，仍需完整字节和合法命令序列检验。

## RISC-V Advanced

### Custom Extensions

Zbb/Zbc/Zbs 中 clz/ctz/cpop 为前导零/尾零/置位计数，orc.b/rev8 为按字节操作，andn/orn/xnor 为带取反逻辑，clmul/clmulh/clmulr 为无进位乘法，bset/bclr/binv/bext 为单个位操作。Zk* 密码指令如 aes32esi/aes32dsmi、sha256sig0/sha512sum0、sm3p0/sm4ed 分别关联 AES、SHA、SM3/SM4。未知指令可能是未开启扩展而非垃圾字节。

### Privileged Modes

M 为固件/启动层最高特权，S 为内核层，U 为应用层。mstatus/sstatus 表达权限和中断状态，mtvec/stvec 为陷阱入口，mepc/sepc 为异常返回位置，mcause/scause 为原因，satp 关联地址转换。异常入口、返回 PC 与页表语义共同决定真实执行地址。

### Debugging 的状态目标

硬件或系统模拟观察的核心材料是架构/扩展、寄存器、CSR、内存、陷阱与特权级。OpenOCD、GDB、QEMU 是原历史资源名；端口、附着和启动命令不保留。

## ARM64 / AArch64

### ABI 与执行状态

x0–x7 关联参数（x0 常为返回），x8 关联间接结果地址；x9–x15 为临时值，x19–x28 被调用者保存，x29 为 FP，x30 为 LR，sp 要求相应 ABI 的 16 字节对齐，xzr 读零、写丢弃。LR 不一定天然位于栈，是否保存由函数行为和编译决定。

指令固定 4 字节，ADRP+ADD 可形成页相对地址，亦存在 ADR 等 PC 相对形式；不能把原“没有 RIP 相对”误解为没有 PC 相对寻址。NOP 的指令字为 0xD503201F；BR/BLR 操作数为寄存器，BLR 还设置返回关系。

### 常见模式

STP x29/x30 到递减 sp 与 MOV FP 建帧；LDP 与递增 sp 恢复后 RET。原例栈帧大小 0x30。ADRP 加 0x8 得到页内地址；跳转表可能以 LDRB 读偏移、SXTB 符号扩展并加到基址，再 BR 进入目标。

### 返回链语义与边界

原 AArch64 ROP 材料的知识重心是 LDP 从栈恢复多个寄存器和 RET 的后继，固定宽度与 LR 使可用片段不同于 x86。原例中可见 x19/x20/x29/x30 与 sp 变化，这些属于已有指令片段的状态效果；本页不复制调用框架、命令字符串或构造链。

### 跨架构条件

ELF 的 AArch64 标记、raw blob 的模式/基址、目标库与系统调用语义构成解释条件。原 qemu-aarch64-static、radare2/Ghidra、交叉反汇编和 Unicorn 只是历史表示名称，不规定工具链。原案例线索为 Google CTF 2016 Forced Puns、Insomni’hack 2018 onecall。

## MIPS64 Cavium OCTEON CP2（SEC-T 2017）

dmtc2/dmfc2 是与 CP2 传送 64 位数据的指令，可能驱动 AES/SHA256 硬件，而非普通寄存器搬运。selector 范围 0x100–0x40FF 是原线索，需要型号手册对应。

原寄存器记录：AES 参数 quadword 0–3 为 0x0104–0x0107；SHA256 中间状态为 0x400E–0x4012，控制/结果记录为 0x404F。参数写入次序、读回时点和宽度解释硬件状态，具体密钥字节不保留。

## EFM32 MMIO AES（SEC-T 2017）

原 Cortex-M flat 映像加载语境为小端 ARMv7-M、基址 0x1000、Thumb 状态。AES 外设基址 0x400E0000：CTRL +0x000、CMD +0x004，KEYLA/LB/LC/LD 为 +0x010/+0x014/+0x018/+0x01C。控制位关联使能/解密，命令关联启动/停止，参数字写入关联密钥装载。

原样本将两组值 XOR 后作为 AES 参数，并对内嵌块作 ECB 逆变换。保留数据来源、字序、合成关系和模式，移除具体密码材料与密码库调用代码。MMIO 地址应与真实芯片资料核对，不能套给任意 Cortex-M。

## MBR / Bootloader（Square CTF 2017）

BIOS 常把 MBR 放在 0x7c00；16 位实模式、段寄存器与之后的 32 位保护模式切换决定地址解释。磁盘偏移、加载地址、PC 和当前模式不能混为一个数。原条件分支改变样例仅说明比较与后继的依赖，不保留补丁程序。

观察产物包括入口附近指令、寄存器、原始字节、模式切换状态和磁盘到内存的对应；源码级调试是否可得取决于符号，不能仅由连接调试接口推定。

## 证据与报告

信号映射、采样边沿、位宽、端序、特权级、异常与状态、MMIO/CP2 寄存器表、参数来源和输出之间保持可追溯关系。原资料的示例地址、阈值、接口与设备代际局限均保留在对应主题中。
