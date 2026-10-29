# 栈、堆与内核内存损坏知识

保留已知漏洞的保护条件、数据布局、ABI、对象生命周期与稳定性解释。原利用模板、工具调用和固定 agent 阶段转为技术目标、约束及可观察结果，不复制运行 payload。

## 阅读导航

| 知识页 | 内容 |
| --- | --- |
| [内存损坏与环境一致性](overview.md) | 漏洞分类、NX/PIE/canary/RELRO、版本与保护、两类场景和稳定性 |
| [栈、ABI 与控制流条件](references/stack-pwn.md) | 覆盖偏移、ret2libc/ret2csu、one_gadget、符号基址、对齐和 I/O |
| [分配器、链表与对象生命周期](references/heap-pwn.md) | glibc 版本、tcache/fastbin/unsorted/large、safe-linking、House 分类、FILE/退出回调 |
| [内核对象、保护与返回状态](references/kernel-pwn.md) | 镜像、slab 对象、KASLR、SMEP/SMAP/KPTI、凭据与 helper 机制、历史案例 |

## 内容组织与历史限制

一份来源对应一份正文，技术细节和故障依据保留在原子主题页。源中若干“全版本”“一次足够”等概括以及 glibc 版本分界、slab 大小和固定 gadget 偏移有适用范围风险，正文保留原说法背景并注明条件，未将其提升为普遍规律。

来源范围（纯文本）：skills/pwn-chain/SKILL.md；skills/pwn-chain/references/stack-pwn.md；skills/pwn-chain/references/heap-pwn.md；skills/pwn-chain/references/kernel-pwn.md。
