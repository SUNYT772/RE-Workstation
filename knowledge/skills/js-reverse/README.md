# 前端 JavaScript 逆向知识导航

本模块按问题提供知识：请求与脚本关联、运行时证据、浏览器环境依赖、本地复现、首处分歧定位及语义恢复。各页可独立阅读，章节名称表达分析对象，不规定 agent 阶段或执行顺序。

来源范围：skills/js-reverse；来源路径均为相对原工作站根的纯文本。

| 知识页 | 适合查阅的问题 |
|---|---|
| [总体知识](overview.md) | 请求签名、运行时采样、证据到复现的关系 |
| [请求与源码观察入口](references/automation-entry.md) | 页面、请求、initiator 与候选脚本如何关联 |
| [观察范围与定位参数](references/tool-defaults.md) | 分页、压缩源码、源码片段、URL 特征和栈帧 |
| [任务输入字段](references/task-input-template.md) | 描述目标和已有线索所需的字段 |
| [证据记录模板](references/mcp-task-template.md) | Observe/Capture/Rebuild/Patch/Output 的记录含义 |
| [证据产物](references/task-artifacts.md) | 请求样例、调用栈、运行时值、补丁记录 |
| [本地复现](references/local-rebuild.md) | 入口、调用顺序和浏览器依赖 |
| [环境差异修正](references/env-patching.md) | 最小因果单元和 first divergence |
| [Node 环境复现](references/node-env-rebuild.md) | 目标脚本、shim、首个异常与页面证据 |
| [插桩观察](references/instrumentation.md) | 请求断点、源码位置、调用栈和局部变量 |
| [AST 去混淆](references/ast-deobfuscation.md) | 何时值得进行语义清理 |
| [无进展时的证据回溯](references/fallbacks.md) | 断点、源码推测、环境修正和去混淆的诊断 |
| [报告结构](references/output-contract.md) | 参数位置、参与函数、证据、复现状态和缺口 |

本模块只提供排版与知识；原有客户端注册、服务端口、工具调用表、安装与强制启动提示不作为知识依赖。
