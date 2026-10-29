# [种子] Kerberoasting → 离线破解 → DA

来源：skills/field-journal/seed-013_kerberoasting-spn.md

来源性质：**[种子] 预置案例素材，非已验证的真实实战成果。** 文中结果、耗时与修复为原素材中的条件化叙述，不表示本次或真实目标已完成验证。

原案例/种子材料的知识转写：保留完整问题、观察、失败原因、修复条件与验证；不保留执行命令、安装与载荷。

## 场景分类
渗透测试 / AD 攻击

## 目标概述
有一个普通域用户凭据，目标域内存在配置 SPN 的服务账户，通过 Kerberoasting 拿到 TGS 离线爆破，破出明文密码后查 BloodHound 路径直通 DA。

## 观察、决策与验证

1. 域内立足（任意普通用户，无需本地管理员）
2. 枚举 SPN
   
SPN 关联服务名、服务账户与 Kerberos 服务票据。原普通域用户是查询身份，输出是 SPN/主体/票据类型关系；域、账号、密码、域控与导出文件角色化，不保留请求调用。

3. 看哪些账户配了 SPN（通常是 SQL Server / IIS / 自定义服务账户）
4. 离线破解
   
原离线强度评估输入是票据相关加密材料，口令强度、候选分布、规则与计算资源影响耗时。该观察不等于一定恢复明文，不保留字典/规则/设备调用。

5. 破出某 svc 账户密码 → BloodHound 查这个账户的可达路径
6. 原记录将 Domain Admins / Server Operators / Backup Operators 等高影响组与敏感权限关联；实际复制权限需核实，组名不能直接推出 DCSync
7. RDP/WinRM 可达身份、关键主机与可见凭证是不同边界；原后续凭证获取与域影响只保留为关系，不保留操作

## 失败路径与诊断

| 问题 | 原因 | 解决方案 | 耗时 |
|------|------|---------|------|
| GetUserSPNs 无返回 | 当前用户没有读 SPN 权限 | 目录读取权限、域控定位和预认证状态分别核对；源“任何普通域用户都可以”的概括不足以排除策略差异 | 20min |
| 破解几小时无果 | 密码强度高 | 原耗时受口令强度、候选分布、规则和计算资源影响；替换调用不保留 | 数小时 |
| 拿到密码登录失败 | 凭据已过期或大小写敏感 | 凭据时效、大小写和服务协议的实际认证结果需要对照 | 10min |
| BloodHound 没数据 | 数据采集时少了 GPO/ACL | 缺少 GPO/ACL 会使图数据不完整，完整类别与压缩输出是不同能力概念 | 30min |
| AS-REP Roasting 没找到目标 | 设了 "Do not require Kerberos preauth" 的账户少 | AS-REP 与预认证策略是独立类别，不能用 TGS 结果代替 | 15min |

## 能力与格式知识


- impacket-GetUserSPNs 与 PowerView 在原素材中承载 SPN/目录属性和票据关系观察，平台差异不构成固定选择。
- netexec（nxc）与旧名 CrackMapExec 是历史背景，源模块 spider_plus、lsassy、ntds 分别关联共享/文件、身份材料与目录数据，不能把模块存在当成影响已验证。
- BloodHound Community Edition 表达目录关系图与权限路径；“更快”缺少对照测量，图边仍需版本、ACL 与实际主体条件。
- OneRuleToRuleThemAll 代表密码候选规则比较；bloodyAD 在源材料中代表低权限主体与目录 ACL 的关系，不保留调用或通用成功断言。

## 技术机制与证据

完整 Kerberoasting 流程：


完整例包括身份有效性、TGS、AS-REP、离线评估与目录图。每项有独立前提：SPN、预认证策略、加密类型、凭据生命周期和图数据覆盖；得到某个账户后，真实 ACL 与资源权限决定影响，不能由账户名推出域管。


如果 svc 账户能访问 DC 上 SeBackupPrivilege：


SeBackupPrivilege 关联备份读取权限，与 DCSync 复制权限不是同一概念。原 NTDS.dit 例表达敏感目录数据影响，实际权限和承载身份是必要证据，不复制导出方式。


## 可迁移经验


- Kerberoasting 的目录读权限、SPN、票据加密类型、服务身份和离线候选验证是不同条件；获得票据不等于已恢复可用凭据。
- Kerberoasting、AS-REP、DCSync、DCShadow、Constrained Delegation、Resource-Based Constrained Delegation、ESC1–ESC15 属于源列出的不同 AD 主题，不能以共同目录名称混同前提或影响。

## 算法与结构

**域内立足后 30 分钟标准动作**：


原“域内三十分钟标准动作”把共享、Kerberos、目录图和离线评估拼为固定链；本页改为身份、服务、图权限、密码强度及已知状态的相互印证，不规定时间或工具阶段。


**AD Kerberos hashcat mode 速查**：

| 模式 | 用途 |
|------|------|
| 13100 | Kerberos TGS-Rep (Kerberoasting) |
| 18200 | Kerberos AS-Rep (AS-REP Roasting) |
| 5500  | NetNTLMv1 |
| 5600  | NetNTLMv2 (Responder 抓到的) |
| 19600 | Kerberos TGS-Rep (AES128) |
| 19700 | Kerberos TGS-Rep (AES256) |


## 环境信息
- Kali 2026.x，impacket 0.12+, netexec 1.x, hashcat 6.2+
- 目标 AD: Windows Server 2019/2022, 域功能级别 2016+
- 攻击位置: 域内任意立足点（普通域用户）

## 来源与脱敏说明
本条目为种子数据，基于公开 AD 攻击技术模式编写，不涉及真实目标域。
