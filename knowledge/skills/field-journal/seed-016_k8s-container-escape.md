# [种子] 容器逃逸 → 拿宿主机 root（cap_sys_admin / 特权容器 / docker.sock）

来源：skills/field-journal/seed-016_k8s-container-escape.md

来源性质：**[种子] 预置案例素材，非已验证的真实实战成果。** 文中结果、耗时与修复为原素材中的条件化叙述，不表示本次或真实目标已完成验证。

原案例/种子材料的知识转写：保留完整问题、观察、失败原因、修复条件与验证；不保留执行命令、安装与载荷。

## 场景分类
渗透测试 / 云原生 / 容器安全

## 目标概述
拿到一个容器内的 shell（通过应用漏洞 / 暴露的 Jenkins / 或 K8s 上的 RCE），需要从容器逃逸到宿主机，进而横向控制整个 K8s 集群。

## 观察、决策与验证

1. 进容器后第一时间踩点
   
容器 uid、CapEff、socket 权限、挂载、cgroup/runtime、环境身份和 ServiceAccount 挂载是独立证据。容器 root 不等于宿主 root；token 实际值和查询命令不保留。

2. 按检测结果选逃逸路径：

   **路径 A：特权容器（`--privileged`）**
   
特权容器可能扩大设备和挂载能力，但实际宿主访问仍受 user namespace、LSM、设备和运行时配置影响。原挂载/chroot 代码改为容器与宿主文件系统/身份边界。


   **路径 B：cap_sys_admin / cap_dac_read_search**
   
cap_sys_admin、cap_dac_read_search 与 cgroup/release_agent 涉及不同权限与版本前提。原 CVE-2022-0492 属于具体历史边界，不能由 capability 名称单独断言逃逸成功。


   **路径 C：挂了 docker.sock**
   
docker.sock 的权限决定是否能控制宿主 runtime；套接字被挂入与当前身份可读写是不同事实，不保留创建容器或挂载宿主代码。


   **路径 D：K8s SA token 有过权限**
   
ServiceAccount 的身份、命名空间、RBAC 动词/资源范围与 Pod 安全准入共同决定实际能力。具备 create pod 不自动意味着允许 hostPID/hostNetwork/hostPath 或 privileged；原 API 调用与部署配置不保留。


   **路径 E：kernel exploit（Dirty Pipe / Dirty COW / OverlayFS）**
   
Dirty Pipe、Dirty COW、OverlayFS 等内核类风险需要架构、补丁、配置与 namespace 前提；只看版本字符串不足以证明可用。


3. 逃出来后，在宿主机上找下一跳
   - kubelet 凭据 (/var/lib/kubelet)
   - container runtime socket (containerd / dockerd)
   - 其他 pod 的 token
   - hostNetwork → 直连集群所有 service IP
4. 原报告的后续影响假设需逐资源和身份证据支持，不能从宿主可达自动推出整个集群

## 失败路径与诊断

| 问题 | 原因 | 解决方案 | 耗时 |
|------|------|---------|------|
| 容器身份非 root 且 CapEff 为空 | 应用层加固较好 | 原路径前提不成立，setuid、内核与外部服务属于不同权限边界，不新增替代步骤 | 数小时 |
| 看到 docker.sock 但读不了 | sock 是 root:root 660 | socket 所有者、组、模式与当前 uid/gid 决定访问；存在不等于可用 | 30min |
| 起了特权 pod 但镜像下载失败 | 内网集群，docker registry 内部 | 镜像来源、注册表可达性与准入是独立失败前提，原部署替代方式不保留 | 20min |
| K8s SA token 没权限 | 默认 SA 通常是 default/restricted | 默认 SA 权限与其他工作负载身份隔离需分别观察，不保留跨 Pod 凭证获取 | 1h |
| chroot 后没有常用工具 | 宿主机是极简发行版 | 宿主文件系统可见、namespace、系统伪文件系统和程序依赖是不同条件，不保留挂载操作 | 30min |
| 集群有 PodSecurity Standards | restricted 策略禁了 hostPath / privileged | 不同命名空间准入、资源类型与身份策略需分别说明，不保留替代部署路径 | 数小时 |

## 能力与格式知识


- deepce、botb 与 cdk 在原素材中代表容器权限/隔离线索的归纳；脚本或模块存在不证明某条逃逸路径成立。
- kdigger 表示结构化容器/Kubernetes 环境观察；peirates 表示集群身份与资源关系，kube-hunter 表示集群暴露线索。
- 原产品历史描述涉及不同云环境，实际覆盖依赖版本、身份权限、挂载、runtime 与内核条件，不规定工具依赖。

## 技术机制与证据

一键自检：


原 deepce 例承担归纳容器特权、挂载、socket 和身份配置的目标，检测到候选路径不等于实际逃逸；下载与执行代码已移除。


用 K8s SA token 起特权 pod 逃逸：


原 SelfSubjectRulesReview 与 Pod 配置表达身份可用权限和期望工作负载的差异。apiVersion/kind、namespace、securityContext.privileged、hostPID、hostNetwork、volumeMounts 和 hostPath 是字段语义，不保留部署 YAML、token 或创建请求。


CVE-2022-0492 利用（cap_sys_admin + 不带 user namespace）：


源 release_agent 例涉及 cgroup 状态、回调与宿主上下文的关系，依赖特定内核/namespace/权限组合；不保留写入或触发操作。


## 可迁移经验


- privileged、cap_sys_admin、docker.sock、宿主挂载和内核缺陷具有不同前提；容器内可见对象与可写/可控能力不能混为一谈。
- 宿主影响和集群控制是不同结果：节点文件或进程权限不能直接证明集群身份、RBAC 或控制面权限。

## 算法与结构

**容器逃逸 5 路径速查**：


原五种路径对应特权、capability、runtime socket、SA/RBAC、内核缺陷五类边界。历史标识包括 CVE-2022-0492、CVE-2022-0847、CVE-2022-2588、CVE-2023-0386，保留为漏洞类别，不拼接命令链。


**逃出来后必看**：


kubelet 数据、容器运行状态、主机网络、进程树和启动参数可能包含集群身份/资源关联。报告记录来源及可访问范围，秘密值脱敏，不保留跨 Pod 凭证提取或横向命令。


## 环境信息
- 攻击位置: 容器内（原已有容器会话条件）
- 目标: K8s 1.24+ / Docker 20+ / containerd 1.6+
- 内核: 视目标而定，关注 CVE-2022-0492 / CVE-2022-0847 / CVE-2023-0386 时间窗

## 来源与脱敏说明
本条目为种子数据，基于公开容器/K8s 安全研究编写，不涉及任何真实集群。
