# 跨版本符号对应：输入与记录格式

来源：skills/binary-diff/references/prompt-template.md

## 适用范围与概念

原 Prompt 模板及 API 脚本转写为独立的数据契约。任务对象是同一候选函数的两版反汇编与伪代码，以及需要在目标版定位的旧版符号。下面的 YAML 是知识记录格式，不是工具调用或 agent 指令。

## 核心原理与技术细节

### 标准比对 Prompt 的信息结构

参考函数拥有可信符号，目标函数缺少部分或全部符号。比较需要回答每个目标名称在目标函数中是否有可证实引用，以及引用类型、指令、实体和偏移是什么。输出仅收录证据支持的目标符号；“未找到”可以表示空结果，而不是必须填满所有类别。

### 变量填充说明

| 原字段名 | 信息来源 | 保留含义 |
| --- | --- | --- |
| disasm_for_reference | 旧版参考函数 | 含符号反汇编与地址 |
| procedure_for_reference | 旧版参考函数 | 含符号伪代码 |
| disasm_code | 新版候选函数 | 无符号或部分符号反汇编 |
| procedure | 新版候选函数 | 对应伪代码 |
| symbol_name_list | 参考代码中有意义名称 | 需要在新版定位的符号范围 |

原自动名 `sub_`、`loc_` 通常不被视为待迁移的语义名称；它们仍可能是目标函数地址的标识，不能因此从指令证据中删除。

### 五类对应记录

```yaml
found_vcall:
  - insn_va: '0x180777700'
    insn_disasm: 'call [rax+68h]'
    vfunc_offset: '0x68'
    func_name: ILoopMode_OnLoopActivate
found_call:
  - insn_va: '0x180888800'
    insn_disasm: 'call sub_180999900'
    func_name: CLoopMode_RegisterEventMapInternal
found_funcptr:
  - insn_va: '0x180666600'
    insn_disasm: 'lea rdx, sub_15BC910'
    funcptr_name: CLoopMode_OnClientPollNetworking
found_gv:
  - insn_va: '0x180444400'
    insn_disasm: 'mov rcx, cs:qword_180666600'
    gv_name: g_pNetworkMessages
found_struct_offset:
  - insn_va: '0x1801BA12A'
    insn_disasm: 'mov rcx, [r14+58h]'
    offset: '0x58'
    size: 8
    struct_name: CResourceService
    member_name: m_pEntitySystem
```

例中地址和名称仅用于说明原格式，不表示可复用偏移或已核验真实目标。

### 定位约束

insn_va 是目标版引用指令位置，insn_disasm 是该指令文本。虚调用记录必须定位具有位移的指令，包括间接调用或虚函数指针读取；vfunc_offset 是虚表槽位移。函数指针记录对应实际装载/引用目标地址的指令，全局变量记录对应实际装载/引用变量的指令。普通结构体偏移需要成员名、结构体名、位移及访问宽度，虚函数指针不归入该类。

### 批量脚本骨架所承载的算法关系

原脚本把五项输入组合成单函数比对内容，得到文本，再提取 YAML 数据并按五种类别区分名称与注释。知识上可保留为“输入身份—对应证据—格式解析—类别分派—实体校验”的数据关系，不保留网络请求、Authorization、模型选择或 API 调用。

原例能容忍纯 YAML、yaml 围栏或通用围栏中的返回文本，说明展示外壳与数据本体不同。解析成功不等于语义正确；空 YAML 可能解析为空值，空结果与异常结果不应混同。

### 引用地址与目标实体

原 apply_results 骨架在 renames 列表中保留 insn_va，并带 call_target、funcptr_target、gv 等类别提示；注释也强调需要解析真实目标。这不是可直接把引用地址重命名的完整实现。直接调用名应归实际调用函数，函数指针名应归指针目标，全局名应归变量实体；虚调用与成员描述可以归引用点注释。这里保留原骨架的未完成性质，避免将待解析地址误当最终实体。

### API 配置建议的知识转写

原 default/fallback/alternative 分别表示不同规模与难度的解释资源；temperature=0、60 秒等待等是历史实现参数，不是对应正确性的保证。端点、密钥、请求头、调用代码和供应商配置已移除。保留的目标是同样输入下可比较的输出、完整上下文、明确空结果与失败状态。

## 观察项与判断依据

有意义名称与新版真实指令、位移和调用关系应一致。任何只存在于参考版、没有目标版证据的符号都属于未匹配项；常量、字符串和结构模式只能支撑候选，不能自动填补缺失关系。

## 故障定位与经验

输出带额外解释可能破坏结构解析；直接使用引用地址会污染命名；虚表槽与普通成员混淆会污染类型解释；过长函数导致上下文截断会制造似是而非的空结果。缓存与输入身份关联，可以识别重复解释与版本不一致。

## 证据与报告结构

保留参考/目标身份、输入代码范围、请求符号范围、五类对应、真实实体解析状态、验证依据和未匹配原因。协议字段完整保留，不以“批量迁移成功”替代字段级证据。

## 相关知识与维护说明

锚点和迭代原理见 [符号迁移原理与证据](../overview.md)，入口见 [README](../README.md)。
