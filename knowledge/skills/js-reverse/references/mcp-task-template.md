# 证据记录模板

来源：skills/js-reverse/references/mcp-task-template.md

## 主题与范围

原 Observe、Capture、Rebuild、Patch、Output 名称在此表示记录分区，不是客户端调用模板或固定 agent 阶段。

## 知识正文

| 分区 | 完整保留的字段 |
|---|---|
| Observe：观察 | 页面；目标请求；initiator；可疑脚本 |
| Capture：采样 | 采样方式；命中位置；关键入参；关键返回 |
| Rebuild：复现材料 | 入口函数；依赖脚本；环境缺口 |
| Patch：环境差异 | first divergence；本次补丁；复测结果 |
| Output：产出状态 | 是否拿到参数；是否可稳定复现；剩余风险 |

## 观察与判断

输入、输出与同一次请求事件对应时，记录才能支撑因果归因；环境补丁需对应一个可解释的分歧。

## 记录与报告

各字段可直接作为报告栏目，空缺处写明未知原因，不以阶段完成标记代替技术证据。
