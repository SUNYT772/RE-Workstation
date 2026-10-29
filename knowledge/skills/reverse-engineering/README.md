# 逆向基础与高阶方法

本模块是已有资料的知识转写。来源只覆盖 `skills/reverse-engineering`；知识页保留格式、算法、运行时、反分析、故障线索和案例经验，不构成工具调用、安装或固定 agent 执行流程。

## 基础与技术对象

- [基础原则](principles.md)：目标边界、证据、比较方向、PIE、诱饵与报告结构。
- [ELF 与 AArch64](elf-analysis.md)：文件布局、运行时映射、压缩与自修改识别。
- [编解码与密码识别](crypto-decode-tools.md)：编码特征、常量、轮结构和数学问题。
- [Go 专题](go-reverse.md)：pclntab、moduledata、Garble、CGo。
- [内核驱动](kernel-driver-reverse.md)：WDM/KMDF、IOCTL、LKM、对象结构与 Rootkit 证据。

## 表示、状态与反分析

- [静态表示与格式](tools.md)
- [动态状态与路径约束](tools-dynamic.md)
- [高阶语义恢复与差异比较](tools-advanced.md)
- [反分析机制](anti-analysis.md)
- [基础二进制模式](patterns.md)

## 语言、平台与案例

- [解释型语言和字节码](languages.md)
- [编译语言与运行时](languages-compiled.md)
- [平台和框架案例](languages-platforms.md)
- [操作系统、固件和总线](platforms.md)
- [硬件与指令集](platforms-hardware.md)
- [算法案例一](patterns-ctf.md)
- [算法案例二](patterns-ctf-2.md)
- [算法案例三](patterns-ctf-3.md)
- [经验笔记](field-notes.md)
- [参考资源的知识分工](awesome-re-resources.md)

## 深入参考

- [参考页导航](references/README.md)
- [AI 语义恢复与验证](references/ai-assisted-re.md)
- [OLLVM、MBA 与控制流恢复](references/ollvm-deobfuscation.md)

## 阅读约定

各页以来源路径、范围、分主题正文和证据说明组织。来源路径是相对原工作站根目录的纯文本。历史工具名用于解释原有术语与方法类别；指标、年代和案例不表示本次已运行或联网核验。跨模块关联只保留文字线索；本模块内的相对链接在整理完成时统一检查。
