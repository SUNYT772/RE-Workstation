# Android 反调试识别

## Native 检测与观察点

| 检测机制 | 代码或运行线索 | 分析要点 |
| --- | --- | --- |
| ptrace | PTRACE_TRACEME、返回值和错误状态 | 是否把已跟踪视为异常；判断发生时间 |
| TracerPid | `/proc/self/status` | 比较正常与调试状态下读取值及后续分支 |
| 内存映射扫描 | `/proc/self/maps` 的模块与字符串 | 区分真实库、映射名称、内嵌库与误报 |
| 本地端口探测 | 连接已知端口，如历史默认27042 | 端口命中只是线索，需关联进程与分支 |
| Linker 命名空间 | 已加载库及动态加载痕迹 | 加载方式和时机是否改变检测面 |
| 进程枚举 | `/proc` 和进程名 | 进程名、路径和行为交叉核对 |
| 代码完整性 | libc入口字节与磁盘映像差异 | 区分预期重定位、运行期修改和第三方插桩 |
| 线程名 | pthread_getname_np、特征线程 | 线程名称会随实现与版本变化 |

## Java 检测与相关行为

`Debug.isDebuggerConnected()` 对应调试连接；`FLAG_SECURE` 对应截图/录屏窗口标志；`Build.TAGS` 中 test-keys、ADB_ENABLED、已知框架类加载属于环境识别线索，不能把任意单项直接等同特定框架存在。

## 壳与关联文件

libshell.so / com.mx.shell（芒果），libjiagu.so（360），libtup.so（腾讯历史线索），libnesec.so（网易），libDexHelper.so（梆梆），libexec.so（爱加密）。结合 [完整壳特征](../signatures/packers.md) 交叉判断。

## 观察位置与时机

内嵌运行组件、启动期观察、运行中附加、内核观测在可见进程、模块映射、线程、早期初始化与权限方面各有不同。原材料的优先级属于历史样本经验，不能普遍保证“完全不可见”。选择分析位置时关注能否覆盖检测发生前后、是否扰动目标行为、证据是否可独立对照。

来源：原 `knowledge/anti-debug/android.md`。保留识别机制与分析比较，不包含具体工具调用。
