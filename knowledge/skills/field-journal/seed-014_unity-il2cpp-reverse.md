# [种子] Unity IL2CPP 游戏逆向 → 还原元数据 + 修改逻辑

来源：skills/field-journal/seed-014_unity-il2cpp-reverse.md

来源性质：**[种子] 预置案例素材，非已验证的真实实战成果。** 文中结果、耗时与修复为原素材中的条件化叙述，不表示本次或真实目标已完成验证。

原案例/种子材料的知识转写：保留完整问题、观察、失败原因、修复条件与验证；不保留执行命令、安装与载荷。

## 场景分类
游戏安全 / 移动逆向

## 目标概述
一个 Unity 打包的 Android 游戏（IL2CPP 模式），游戏内购或核心算法在 C# 编写但已编译成 native。需要还原方法名、定位关键逻辑、改动 / hook 实现修改。

## 观察、决策与验证

1. 拆包 APK，确认是 IL2CPP
   
APK 中 ABI 对应目录的 libil2cpp.so 与 global-metadata.dat 是原 IL2CPP 识别线索。库和元数据的架构、版本、内容身份应匹配；目录存在不证明元数据未加密。

2. 用 **Il2CppDumper** 还原元数据
   
原恢复输出为 DummyDll、script.json、il2cpp.h、dump.cs，分别承载伪托管程序集、地址映射、类型声明和可读方法声明。所有产物需关联同一次输入，避免名称与 native 地址错配。

3. 把 IDA 的 IL2CPP 脚本（`ida_with_struct.py`）跑一遍
   - libil2cpp.so、类型声明与 script.json 的同源性决定方法映射是否可信
   - IDA 现在能看到 C# 方法名、签名、字符串
4. 在 dump.cs 里按业务关键字搜（`AddCoin` / `OnPurchase` / `Verify` / `IsVip` / `CheckSign`）
5. 拿到关键方法的偏移 → IDA 跳过去看反汇编 / 反编译
6. 选择修改方式：
   - 静态变化关注返回约定、文件身份与业务状态，不保留补丁字节
   - 运行时变化关注方法 ABI、真实调用参数与业务结果
7. 重打包验证 / 注入验证

## 失败路径与诊断

| 问题 | 原因 | 解决方案 | 耗时 |
|------|------|---------|------|
| Il2CppDumper 报 metadata 版本不支持 | 新版 Unity 改了 metadata 格式 | 元数据格式版本与解析器支持范围不匹配，需要以格式能力说明失败 | 30min |
| global-metadata.dat 加密了 | 用了 AntiCheatToolkit / 自定义加密 | 找游戏初始化时的解密函数（通常在 il2cpp_init 周围）→ Frida 在 mmap 后 dump | 2h |
| dump.cs 看到方法名但 IDA 没匹配 | script.json 与 so 不一致 | 同一次输入的映射与库身份需要一致，陈旧缓存会造成假失配 | 20min |
| Frida hook IL2CPP 方法报错 | IL2CPP 方法不是标准 Java/ObjC，要算 method offset | IL2CPP 方法 ABI/偏移与 Java/ObjC 不同，需要核对真实方法结构 | 1h |
| Patch 后游戏闪退 | 校验文件 hash 或 anti-tamper | 文件完整性、运行时变化和 anti-tamper 是不同失败条件，不保留替代修改方案 | 2h |
| 重打包后启动崩溃 | apksigner v2 签名不能改字节后再签 | 签名方案、打包内容与验证状态需要一致；原因不能单由 v2 签名推断 | 30min |

## 能力与格式知识


- Il2CppDumper 在原素材中表示元数据与代码映射恢复；Il2CppInspectorRedux 表示输出面向 IDA/Ghidra/Binary Ninja 的类型/分析信息，适配取决于 Unity 与 metadata 版本。
- frida-il2cpp-bridge 表示 IL2CPP 类型和方法的运行时关系，不保留“强N倍”或事实标准之类未量化排名。
- dnSpy/dnSpyEx 所见 DummyDll 是重建的伪 .NET assembly，和实际原生方法体需区别。
- UnityCheat、GameGuardian 是源中辅助观察的历史名称，不构成依赖；实际逻辑仍以元数据、原生代码和运行状态对应为据。

## 技术机制与证据

frida-il2cpp-bridge hook 示例：


运行时示例把 Assembly-CSharp 的类型、方法、实例/静态签名及调用参数关联。原 AddCoin、VerifyReceipt 的数值/返回修改改为业务字段与方法结果的对应，不复制改币或付费校验绕过代码。


原编译/注入链只是源描述到运行时观察的承载方式；输入为类型/方法映射与观察逻辑，输出为调用参数、结果和业务状态，不保留安装、编译或注入调用。


IDA 静态 patch：


静态例关注 ARM64 返回约定与业务判定，修改后的文件完整性、签名和运行状态可能与原版不同；原补丁指令和打包/签名步骤已改为验证条件。


## 可迁移经验


- 完整解释需要贯通 metadata 版本、代码注册/元数据注册关系、DummyDll、原生地址与动态方法状态；仅恢复名称不能声称完整还原。
- 加密或变形 metadata、地址映射错误、运行时初始化与版本不匹配属于不同失败路径，产物身份和条件需保留。

## 算法与结构

**IL2CPP 标准流程**：


IL2CPP 识别、元数据、native 映射、业务方法、静态/运行时状态和实际业务场景组成证据关系，不规定固定七步或某个库。


**加密 metadata 处理**：


加密元数据在初始化与文件/内存读取之间可能转为可解释结构，il2cpp_init、open/fopen、mmap/read 是原关联线索。文件与内存的版本、长度和映射一致性决定恢复是否可信，不复制截获或转储脚本。


## 环境信息
- Windows / macOS（运行 Il2CppDumper 用），目标设备 Android arm64
- IDA Pro 7.7+ 或 Ghidra 11+
- frida-tools 16.x, frida-il2cpp-bridge 0.9+
- Unity 版本: 2019.x - 2022.x（不同版本 metadata 格式略异）

## 来源与脱敏说明
本条目为种子数据，基于公开技术模式编写，不涉及任何真实游戏。包名 `〈游戏应用包〉` 为占位符。
