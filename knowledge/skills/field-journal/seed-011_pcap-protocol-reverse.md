# [种子] PCAP 自定义二进制协议逆向

来源：skills/field-journal/seed-011_pcap-protocol-reverse.md

来源性质：**[种子] 预置案例素材，非已验证的真实实战成果。** 文中结果、耗时与修复为原素材中的条件化叙述，不表示本次或真实目标已完成验证。

本页为原案例/种子材料的知识转写。保留环境、观察、失败、修复与验证条件；工具调用、安装与载荷改为技术关系。

## 场景分类
抓包分析 / 协议逆向

## 目标概述
某 IoT 设备/桌面客户端走 TCP 自定义二进制协议（非 HTTP），抓到一段 PCAP，需要还原帧结构、字段含义、加密层（如有），并写一个本地 client/server 复现。

## 观察、决策与验证

1. Wireshark 打开 PCAP，先做基础统计
   - `Statistics → Conversations` 看 IP/端口对
   - `Statistics → I/O Graphs` 看数据节奏
2. 找出真正的应用层流（剥掉 TLS 之类标准层）
3. 在某条 TCP 流上 → `Follow → TCP Stream` → 切到 RAW 模式 → 导出
4. 二进制级观察：每帧前几字节是不是固定 magic / 长度字段？
   
原二进制视图关注帧起始字节、长度与重复片段；输入是重组后的字节流，输出是边界和字段假设，不保留文件查看命令。

5. 用 hex 模式找规律：固定头、长度、TLV、CRC
6. 写 Python 解析器（struct + scapy）一帧一帧解
7. 同步从二进制反编译反查协议字段（IDA / Ghidra 看 send/recv 周围 struct）
8. 验证：原客户端/服务端对照结果与预期响应一致，文档不运行网络服务

## 失败路径与诊断

| 问题 | 原因 | 解决方案 | 耗时 |
|------|------|---------|------|
| Wireshark 不识别协议，只显示 "Data" | 是私有协议，没有解析器 | 写 Wireshark Lua dissector 或 直接 Python 离线分析 | 30min |
| 看起来无规律，每帧都不同 | 有压缩或加密层 | 用熵分析（熵数值）判断是否加密；找 nonce/IV 字段 | 1h |
| 长度字段算不对 | 长度可能是 little-endian / big-endian / 含/不含自身 | 找几条不同长度的帧，列方程组解出来 | 40min |
| TLS 抓到但解不了 | 客户端不留 SSLKEYLOGFILE | 原记录以 TLS 读写边界的明文与 PCAP 密文对照，未保留插桩调用 | 1.5h |
| 数据正确但服务端不响应 | 协议带递增的 seq / nonce，重放被拒 | 搞清楚 seq 计算方式（通常前一帧的 hash 或递增计数器） | 50min |

## 能力与格式知识


- Wireshark Lua Dissector 在源素材中把字段解析映射为抓包视图；原“小于100行”仅描述该例规模，不是一般复杂度保证。
- scapy 的 Packet 类型描述字段结构；Kaitai Struct 用 YAML 描述字节序、长度和重复关系，可对应 Python/Java/C++/JS 等解析实现。
- NetworkMiner 在原素材中代表事后流量的文件重组与凭据线索，与交互式包字段观察关注面不同，不作工具排名。
- 熵描述字节分布随机性；原大于7.5的判断也可能来自压缩，不能独立证明加密。

## 技术机制与证据

scapy 自定义协议示例（TLV）：


原 scapy 结构为 magic=AB CD（2 字节）、version（1）、type（1）、length（uint16，大端）、seq（uint32）、payload、crc（uint16），示例 payload 长度写为 length−8。magic/头是否计入 length 需要用多帧方程确认；该偏移不是可直接套用的协议事实。原 dport=9527 只作会话样例，报告统一为〈协议服务端口〉，不保留解析调用。


Kaitai Struct YAML（原素材的声明式结构表示）：


Kaitai 的声明结构以 meta.endian=be 与 seq 字段顺序表达相同的帧假设：magic、u1/u1、u2、u4、length−8 的 payload 和 u2 CRC。声明式结构能生成不同语言 parser，但本页只保留字段、类型与长度关系，不部署生成器。


熵分析：


原熵图与熵值对应局部和总体随机性观察。高熵可来自加密或压缩，不能由大于 7.5 的数值单独断言加密；nonce/IV、解码前后结构及发送端逻辑可补充解释。


## 可迁移经验


- 自定义协议解释可从会话与方向、消息边界、字段假设及样本对照逐层展开；TCP 分段不能直接当应用消息边界。
- Kaitai 类声明式结构能够记录字节序、长度依赖与 TLV 关系；字段名是分析假设，跨样本一致性和错误样本决定解释可信度。

## 算法与结构

**自定义协议逆向 4 步法**：


节奏/会话、帧界、字段和编码层构成原四类证据。magic、长度、终止符、TLV、校验、nonce 与 send/recv 附近结构应交叉验证；不把 TCP segment 或 PSH 标记自动当作应用帧边界。


**找帧长度的小技巧**：

把同流的所有 PSH 包导出 → 看每个 TCP segment 的总长度，看长度字段（位置 i, i+1, i+2 都试）是否能推出 segment 长度。


## 环境信息
- Kali / Ubuntu，Wireshark 4.x, Python 3.10+, scapy 2.5
- 目标协议: 自定义 TCP 二进制（含 TLV / 长度前缀）
- 加密层: 视情况而定（常见 AES-CTR / ChaCha20）

## 来源与脱敏说明
本条目为种子数据，基于公开协议逆向方法编写，不涉及真实产品。
