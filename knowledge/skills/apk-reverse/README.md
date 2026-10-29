# Android APK 逆向知识导航

本模块保留 APK 结构、Java/smali/native 归因、JNI、运行时观察、证书与设备检测、DEX、跨平台框架和安全评估知识。页面按主题查阅，不规定 agent 阶段或工具依赖。

来源范围：skills/apk-reverse；来源路径为相对原工作站根的纯文本。

| 知识页 | 内容 |
|---|---|
| [总体知识](overview.md) | 包结构、逻辑定位、重建产物与报告 |
| [Android 高级逆向](references/android-advanced.md) | JNI、内存、证书、Root、加固、React Native/Flutter |
| [APK 安全检查](references/apk-security-checklist.md) | 静态、动态、网络、存储、认证授权、代码保护 |
| [通用检测框架观察点](references/frida-bypass-kit.md) | Root、证书绑定、模拟器、反调试的判据 |
| [运行时观察场景](references/frida-cookbook.md) | 方法、密码学、网络、存储、类加载与辅助算法 |
| [Windows 解包产物知识](scripts/decode.ps1.md) | 双视图、摘要字段、部分失败判定 |
| [Shell 解包产物知识](scripts/decode.sh.md) | 路径命名、计数与状态差异 |
| [Windows 设备与进程上下文](scripts/frida-run.ps1.md) | 设备、进程、spawn/attach、暂停状态 |
| [Shell 设备与进程上下文](scripts/frida-run.sh.md) | 连接上下文与目标身份 |
| [Manifest 摘要语义](scripts/manifest-summary.ps1.md) | XML 命名空间、权限、组件与主入口判据 |
| [Windows 重建产物知识](scripts/rebuild-sign-install.ps1.md) | unsigned/aligned/signed、验签与设备结果 |
| [Shell 重建产物知识](scripts/rebuild-sign-install.sh.md) | 签名状态、覆盖语义与来源差别 |

scripts 子目录中的 .md 是原脚本的知识转写，不包含可执行脚本。debug.keystore 及凭据不进入 clean 副本。历史工具、项目名仅用于识别技术背景；安装、自举、调用参数和平台注册信息已移除。
