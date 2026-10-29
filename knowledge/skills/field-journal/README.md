# 经验材料库

本模块包含 30 份来源材料，按来源性质分开阅读；正文保留问题、观察、决策、失败路径、原修复/验证与可迁移经验。工具调用、安装和执行协议均转为知识解释。

## 原项目记录（按来源自述）：7 篇

这些页面转述原记录，不能等同于本次重新运行验证。

- [2026-05-15 Android 模拟器崩溃与 Fragment 生命周期](2026-05-15-cellular-pro-mumu-ksad-fragment-fix.md)
- [2026-05-15 lumine v0.9.1：Go TLS 分片代理逆向](2026-05-15_lumine-go-reverse.md)
- [2026-05-16 博客系统：Swagger、Mass Assignment 与前端静态证据](2026-05-16_pentest-personalblog-fun-mass-assignment.md)
- [2026-05-17 Vue SPA 与 Spring Boot：配置与凭证暴露](2026-05-17_pentest-vue-spa-actuator-leak.md)
- [2026-05-25 Access 保护范围、兄弟子域与会话 Cookie](2026-05-25_pentest-cf-access-sibling-subdomain-cookie-poisoning.md)
- [2026-05-26 Web 网关：限速身份与匿名信息](2026-05-26_pentest-newapi-rate-limit-bypass.md)
- [2026-06-29 HTTP 扩展：请求服务上下文与异步收尾](2026-06-29_burp-mcp-full-test-and-fix.md)

## [种子] 预置案例素材：17 篇

以下属于方法素材，非已验证的真实实战成果。结果与耗时仅按原素材条件理解。

- [[种子] ELF 自解压加载器逆向](seed-001_elf-packed-loader.md)
- [[种子] Go 恶意软件逆向（stripped + Garble）](seed-002_go-malware-stripped.md)
- [[种子] Web API 未授权访问 + IDOR](seed-003_web-api-auth-bypass.md)
- [[种子] JS 签名逆向（Webpack + AES + 时间戳）](seed-004_js-sign-webpack.md)
- [[种子] [2026-03] AD CS ESC1 证书模板滥用 → 域管权限](seed-005_ad-certipy-esc1.md)
- [[种子] [2026-02] SSRF → 云元数据 → AK/SK → OSS 全量数据](seed-006_ssrf-cloud-metadata.md)
- [[种子] [2026-04] NTLM Relay + Coercer → 域管权限（无需密码）](seed-007_ntlm-relay-coercer.md)
- [[种子] APK Frida 绕过 OkHttp SSL Pinning](seed-008_apk-okhttp-ssl-pin-bypass.md)
- [[种子] iOS 越狱检测绕过 + 抓包](seed-009_ios-jailbreak-detect-bypass.md)
- [[种子] CTF Pwn — x64 栈溢出 + ROP 链调用 system](seed-010_ctf-pwn-rop-x64.md)
- [[种子] PCAP 自定义二进制协议逆向](seed-011_pcap-protocol-reverse.md)
- [[种子] Log4Shell（CVE-2021-44228）JNDI 注入打 RCE](seed-012_log4shell-jndi-rce.md)
- [[种子] Kerberoasting → 离线破解 → DA](seed-013_kerberoasting-spn.md)
- [[种子] Unity IL2CPP 游戏逆向 → 还原元数据 + 修改逻辑](seed-014_unity-il2cpp-reverse.md)
- [[种子] IoT 路由器固件提取 + UART 串口拿 root](seed-015_iot-firmware-uart.md)
- [[种子] 容器逃逸 → 拿宿主机 root（cap_sys_admin / 特权容器 / docker.sock）](seed-016_k8s-container-escape.md)
- [[种子] XXE 盲注 OOB → 外带 /etc/passwd 与内网探测](seed-017_xxe-oob-exfil.md)

## 索引、模板与知识支持：6 份

- [经验材料分类索引](_index.md)
- [经验材料的知识记录结构](_template.md)
- [经验材料的脱敏与语义保留](anonymization.md)
- [授权语境与技术证据的区别](precedent-auth.md)
- [逆向工程先例的技术知识](precedent-reverse.md)
- [安全评估先例的协议、权限与证据知识](precedent-pentest.md)

## 来源与证据

每页标注原目录相对来源。原统计、错码和证据不足处明确说明；工具名只作历史概念，跨域主题以文字关联。源内容未修改，未运行源脚本、部署或连接外部服务。
