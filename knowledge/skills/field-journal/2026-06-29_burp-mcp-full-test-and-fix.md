# 2026-06-29 HTTP 扩展：请求服务上下文与异步收尾

来源：skills/field-journal/2026-06-29_burp-mcp-full-test-and-fix.md

## 案例问题与环境

原 burp-mcp-full 扩展声明 63 项能力，源记录进行运行可用性比较并报告三个功能缺陷和一个桥接竞争条件。运行目标统一为〈授权测试服务〉，本页没有编译、加载、发送请求或连接 MCP。工具调用代码与构建脚本改为协议、对象和状态机知识。

环境背景为 Windows 11 10.0.26200、JDK 21.0.11+10、Burp Suite Professional 2026.4.2、montoya-api 2025.5、gson 2.11.0、nanohttpd 2.3.1。版本是原记录，不构成安装依赖或最新版本结论。

## 观察、决策与结果

Java dispatch、能力清单与桥接描述三处的 63 项一致性，只能证明静态名称覆盖，不能证明运行可用。源五批覆盖只读/编解码/查询、网络/编辑器、七种 Intruder 变体、范围/配置/规则/处理器/问题/比较，以及抓取和历史清空，代表不同副作用与对象前提。原七类变体名称为 attack、async、wordlist、pitchfork、cluster_bomb、battering_ram、with_options，保留为源接口分类。

原记录声称修复后回归通过；本知识页保留该历史结论及对应故障，而不重新执行或把它扩展到其他版本。

## 失败路径与修复机制

| 故障 | 根因 | 原修复与验证知识 | 原耗时 |
|---|---|---|---|
| scan 的 request_count 恒为 0 | AuditConfiguration 没有接收种子 URL，任务未有初始请求 | URL 语义、HttpService 和种子请求需被 Audit 状态实际接收，配置存在不等于已有输入 | 2 小时，含验证 |
| Intruder 提示缺少 HttpService | 原始报文字节未带服务对象 | host、port、TLS 与 raw request 关联，使消费者得到完整目标上下文 | 20 分钟 |
| 上游代理字段缺失导致 NPE | null 后直接字符串转换 | 参数存在性、类型和清晰错误是不同边界 | 5 分钟 |
| 快速四请求中第四个无响应 | stdin 关闭就退出，未完成 HTTP 任务被终止 | pending 与 stdinClosed 共同决定生命周期；原有 mock 验证 | 1 小时 |
| HTTP_CODE=000 | 本地环境的客户端被沙箱限制 | 本地传输失败不能当目标服务不存在；不同实现的观察作对照 | 5 分钟 |
| Audit 包位置推断错 | 根据文档猜测未核对实际 JAR | 实际版本为 scanner.audit.Audit，实际类型信息比猜测可靠 | 30 分钟 |
| 中文文本替换失败 | UTF-8 BOM 与显示编码错层 | BOM 识别、实际文本与终端显示需要区分 | 10 分钟 |

## 技术机制与证据

### HTTP 请求与服务对象

HttpRequest 的 raw 表示与 HttpService 的 host、port、TLS 是两层对象。原 Repeater 接受无 service 重载，而 Intruder 要求附带服务上下文，说明同一种报文被不同消费者使用时前提不同。Host 头可有端口，源示例缺失时回退为无 service 请求；它以 raw 中是否出现 https 或 :443 推测 TLS，属于启发式而非可靠通用 URI 解析。原正则未完整覆盖 IPv6 等输入，此处不把示例推广为全部解析器。

### 扫描种子与状态

URL 包含 protocol、host、显式/默认 port、path 与 query。原空路径按根路径解释，query 追加到 path，http/https 对应不同默认端口。构造请求、服务对象存在与 Audit 真正接收到种子是三个状态；request_count 从零变化是源修复的关键观察。原请求构造/网络调用不复制。

### 桥接竞争条件

状态模型只有两个量：未完成任务数 pending 与输入已关闭标志 stdinClosed。接收一项工作时计数增加，成功或失败收尾都减少；关闭输入只说明不会再有工作，并不说明已有工作结束。完成条件是“输入已关闭且 pending 为零”。这是一段纯生命周期算法，适用于异步资源收尾，不规定线程、工具或进程退出调用。

### 请求结果分类

源测试记录结构包括可解析 JSON、非 JSON 原文片段、传输错误与超时。30 秒是原超时示例，结果类别彼此不同。Content-Length 的字节长度、响应 data/end 及 error/timeout 都影响解释；不能把无响应当成有效空结果。这里只保留状态和字段，不保留 HTTP 客户端代码。

## 版本与能力知识

montoya-api 2025.5 的 Audit 位于 burp.api.montoya.scanner.audit；版本对象的 major/minor/build 已发生移除，源记录以 buildNumber/edition/toString 获取版本信息。http.sendRequest 的流量不进入 proxy history，是源观察，不能把历史为空解释成未发送。IDA 的实例端口可能递增，而此 Burp 服务可由属性/环境设置端口；端口角色和实际绑定需要分别记录，不使用固定地址连接。

## 可迁移经验与报告

描述表一致、编译成功、服务加载、请求实际可用和功能结果是不同证据层。原场景的关键迁移点为消费者对象前提、异步结束条件、输入缺失处理、版本 API 与真实构建依赖一致性。报告记录每项能力的输入前提、实际结果/错误、修复根因、回归证据和未覆盖版本。

原维护表、索引同步与安装登记属于平台运行元数据，未保留为 agent 强执行要求；原更新了哪些材料只作为历史背景，不宣称本次修改这些范围外文件。
