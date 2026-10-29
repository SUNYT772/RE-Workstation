# 操作系统、固件与总线的逆向知识

来源：skills/reverse-engineering/platforms.md

## macOS / iOS

### Mach-O Binary Format

Mach-O 的 load commands 决定 dyld 的加载关系。LC_MAIN 关联入口，LC_UNIXTHREAD 是另一类历史入口状态，LC_LOAD_DYLIB 关联动态库依赖，LC_CODE_SIGNATURE 关联签名数据。Universal/fat 文件可含多个架构切片，偏移需要按当前切片解释。

__TEXT 常为代码、__DATA 为全局状态、__LINKEDIT 为符号/链接相关材料；__text、__cstring、__objc_methname 分别关联指令、C 字符串、ObjC 选择子。__DATA_CONST.__got 与 __DATA.__la_symbol_ptr 分别表达 GOT 与延迟符号指针。load command、虚拟地址、文件偏移和 ASLR slide 应保持对应。

### Code Signing & Entitlements

签名是否存在、校验是否通过、签名覆盖区域和 entitlements 是不同属性。com.apple.security.app-sandbox 与 com.apple.security.network.client 是原能力字段例子。代码修改会影响签名有效性；原 ad-hoc 签名是历史本地测试背景，不提供移除/重签命令，也不把“能加载”当作所有安全属性相同。

### Objective-C Runtime

@interface、@protocol、方法签名与类型来自运行时元数据。objc_msgSend 的 receiver、selector 与实现地址形成动态分发关系；原 rdi=self/rsi=selector 只对应特定 x86-64 ABI。selector 字符串与实现交叉引用能够定位方法，类和继承也参与目标选择。

method_exchangeImplementations、class_replaceMethod 是 swizzling 线索，需要运行时实现指针与磁盘元数据对照。class-dump、dsdump、ObjC 段分析是历史材料表示，不保留方法实例化或工具调用代码。

### Swift Binary Reversing

__swift5_types、__swift5_proto、其他 __swift5_*、swift_* 与符号解名能揭示类型与协议。Value Witness Table（VWT）关联类型操作，Protocol Witness Table（PWT）关联协议分发。swift_allocObject、swift_release、swift_bridgeObjectRetain、swift_once 分别提示分配、引用减计数、ObjC/Swift 桥接持有和惰性初始化。

原字符串模型包括 16 字节存储中的最多 15 字节小串、指针/长度/flags 的大串；具体编码以目标 ABI 为准。Array 可关联 ContiguousArrayStorage 的头与元素，Dictionary 可表现为开放寻址哈希表。元数据解析与真实字段访问需要相互验证。

### iOS App Analysis

IPA 中 Payload/*.app、Mach-O 与资源归属应区分。LC_ENCRYPTION_INFO 的 cryptid 表示相应加密状态，文件中加密区域与运行时明文区域不必一致。frida-ios-dump、Clutch、bfdecrypt 是原历史资料名，本页不包含设备连接或解密调用。

越狱检测可能涉及 Cydia/Sileo、/private/var/lib/apt、/etc/apt、/bin/sh、/usr/bin/ssh、substrate/substitute，以及沙箱下 fork 行为。路径存在、访问权限、API 返回值和实际分支需分开解释；单项命中不足以说明整个系统状态。

### dyld / Dynamic Linking

dyld 的实际加载库、插入库、依赖与 shared cache 能解释真实 API 归属；hardened runtime 与 SIP 会影响相关环境机制。原缓存路径包含 arm64e，说明架构与系统版本关系。这里保留动态链接和代码来源知识，不保留注入环境变量或运行命令。

## Embedded / IoT Firmware

### Firmware Extraction：来源与边界

容器魔数、版本字符串、文件系统标记、压缩数据与分区边界构成镜像层次。hsqs 提示 SquashFS，UBI# 提示 UBI，递归封装可能形成多个容器层。偏移、长度和原镜像哈希连接提取组件与来源。

物理接口的知识属性：UART 常见 GND/TX/RX/VCC 与波特率（原例常见 115200）；JTAG 与 CPU 调试信号、10/14/20 针连接有关；SPI 常见 8 针 SOIC 闪存（Winbond/Macronix）；eMMC 常见测试点与存储块。原 USB-UART、OpenOCD/J-Link/Bus Pirate、JTAGulator、flashrom/CH341A、eMMC reader 是历史接口载体，不提供接线、写入或采集命令。

### Firmware Unpacking：格式族

SquashFS 的压缩族可能为 xz/lzma/lzo/gzip；JFFS2、UBI/UBIFS、CPIO/initramfs、DTB/DTS 是不同层。zImage、uImage、vmlinux 关联内核封装或符号映像。容器识别、文件系统重建、设备树语义与内核地址不是同一问题，不能只用“解压成功”概括全部。

### Architecture-Specific Notes

ARM、Thumb 的模式与函数指针最低位有关；ARM 常为 4 字节，Thumb 包含 16/32 位编码，原“Thumb 全为 2 字节”是简化。AArch64 的调用约定与固定指令宽度见[硬件知识](platforms-hardware.md)。

MIPS 的端序、指令集版本、PIC 的 gp/GOT、lui+addiu 形成常量与分支延迟槽影响控制流。延迟槽是否执行还需区分 branch-likely 等变体，不能绝对说任何后继指令都执行。RISC-V 需区分扩展、2/4 字节混合与无 flags 分支语义，见[格式知识](tools.md)。

### RTOS

FreeRTOS 的 xTaskCreate、任务名 IDLE/Tmr Svc、xQueueSend/Receive、vTaskDelay、xSemaphoreTake 分别关联任务、队列、时间与同步。Zephyr 的 k_thread_create、k_msgq_put/get 与 CONFIG_* 提供线程、队列和配置线索。

裸机常有位于 0 或 STM32 例 0x08000000 的中断向量、持续输入—处理—输出主循环与 MMIO。地址依芯片和映像布局，外设寄存器应与对应芯片材料一致。

## Kernel Driver Reversing

### Linux Kernel Modules

.ko 是可重定位 ELF，init_module/cleanup_module、.modinfo、导出符号和 file_operations 决定接口。alloc_chrdev_region、cdev_init 与设备对象关联，unlocked_ioctl 的 cmd 分发、copy_from_user/copy_to_user 与用户缓冲区形成数据链。

静态相对地址可独立解释，但运行时仍需模块真实加载地址和重定位，原“base address 无关”不能用于运行时定位。vmlinux、模块符号、内核版本和转储时点应一致。详见[驱动专题](kernel-driver-reverse.md)。

### eBPF Programs

原模型含 r0–r10 共 11 个 64 位寄存器：r0 返回、r1–r5 参数、r10 帧指针。基本指令槽为 8 字节，某些立即数使用两个槽，不能一律一个 opcode 等于一个槽。translated 与 JIT 后主机指令是不同表示。

bpf_map_lookup_elem/update_elem 关联 map，bpf_probe_read 关联内存读取，bpf_trace_printk 关联诊断输出。map 类型、键值宽度、附着点和 helper 副作用都需保留。

### Windows Kernel Drivers

.sys 常为 PE，DriverEntry 的 DRIVER_OBJECT/UNICODE_STRING、IoCreateDevice、IRP_MJ_DEVICE_CONTROL、MmMapIoSpace、ObReferenceObjectByHandle、ZwCreateFile/ReadFile 分别指向设备、请求分发、MMIO、对象引用和内核文件操作。

## Automotive / CAN Bus

经典 CAN 的标识符为 11 或 29 位，DLC 与数据长度相关，载荷通常最多 8 字节；这不覆盖 CAN FD。原例总线速率 500000 是样本配置，不能通用化。时间戳、ID、DLC、数据及其变化关系支持消息解释。

UDS 原服务号：0x27 Security Access（seed/key），0x2E Write Data By Identifier，0x31 Routine Control。seed/key 算法、重放时序与 ECU 固件/UDS/KWP2000 是已有案例主题；接受条件需由状态与响应证据支持。本页不保留接口配置、重放、请求帧或发送命令。

## 证据组织

按容器、切片、模块、地址、架构、runtime/OS、接口字段与时间状态组织报告。签名、加密、授权判断、文件系统与指令语义各自保留证据，避免用一个界面结果替代跨层判断。
