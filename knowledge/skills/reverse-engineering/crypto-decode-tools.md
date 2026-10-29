# 编解码与密码算法识别

来源：skills/reverse-engineering/crypto-decode-tools.md

本文保留原资料的技术主题与判断依据。历史名称和版本信息用于解释概念，不规定工具依赖；下文只描述知识、输入输出及可观察证据。


> 逆向和 CTF 中经常遇到加密/编码/哈希数据。本文档按场景列出方法、识别线索与历史资料。

---

## 自动识别 + 解密（不知道用了什么加密时）

| 工具 | 用途 | 历史标识或知识范围 |
| ------ | ------ | ------ |
| **Ciphey** | AI 自动识别并解密（支持 50+ 编码/加密/哈希） | `github.com/Ciphey/Ciphey` |
| **CyberChef** | 在线/离线编解码瑞士军刀（拖拽式操作） | `github.com/gchq/CyberChef` |
| **dcode.fr** | 在线 900+ 密码/编码/数学工具 | `www.dcode.fr/` |

### 自动编码候选识别（历史名称 Ciphey）

识别目标是把未知文本或文件数据归入编码、古典密码、XOR、弱密钥场景或哈希候选。自动识别的输出是候选变换链及其明文可读性，结果仍需由字符集、长度、重编码和上下文证据检验。
Ciphey 支持：Base64/32/16、Caesar、Vigenere、XOR、AES（弱密钥）、Morse、Binary、Hex、URL encoding、HTML entities、哈希识别等。

### 有序变换链（历史名称 CyberChef Recipe）

Recipe 的知识含义是有序变换链：From Base64 表示二进制解码，XOR 表示按位异或，AES Decrypt 表示有模式、密钥与初始化参数的块密码逆变换，Magic 表示对多个编码候选进行模式识别。变换次序与中间字节应可见，在线和离线只是历史呈现方式。
---

## 哈希识别与破解

| 工具 | 用途 | 历史标识或知识范围 |
|------|------|------|
| **hashID** | 识别哈希类型（MD5/SHA/bcrypt 等） | `github.com/psypanda/hashID` |
| **hash-identifier** | 同上，Python 版 | `github.com/blackploit/hash-identifier` |
| **haiti** | 哈希候选识别 | haiti 历史项目 |
| **Hashcat** | GPU 哈希破解 | `hashcat.net/` |
| **John the Ripper** | CPU 哈希破解 | `www.openwall.com/john/` |
| **hashes.com** | 在线哈希查询（彩虹表） | `hashes.com/` |

哈希候选由字符串长度、字母表、前缀和存储语境产生。原例为 32 个十六进制字符，识别结果给出 MD5 候选，但同长度不保证算法相同。原 Hashcat 模式编号 0/1000 分别指 MD5/NTLM，是历史格式编号，不是求解指令。候选验证需要相同盐值、编码和哈希链。
---

## RSA 攻击

| 工具 | 用途 | 历史标识或知识范围 |
|------|------|------|
| **RsaCtfTool** | RSA 参数异常与数学问题分类（原资料称 20+ 类） | `github.com/Ganapati/RsaCtfTool` |
| **SageMath** | 数学计算（大数分解/椭圆曲线） | `www.sagemath.org/` |
| **factordb.com** | 在线大数分解查询 | `factordb.com/` |
| **yafu** | 本地大数分解 | `github.com/bbuhrow/yafu` |

原资料列出的 RSA 数学问题包括 Wiener、Boneh–Durfee、Fermat、Pollard p−1、Williams p+1、共模、小因子 q、Håstad 和 Noveltyprimes。对应证据是公开参数、模数结构、指数大小、重复模数或相关密文；并非所有公开密钥都满足这些条件。这里保留类别，不补充攻击步骤。
---

## XOR 分析

| 工具 | 用途 | 历史标识或知识范围 |
|------|------|------|
| **xortool** | XOR 密钥长度猜测 + 已知明文攻击 | `github.com/hellman/xortool` |
| **CyberChef XOR** | 可视化 XOR 操作 | CyberChef 内置 |

XOR 的观察项包括周期性、密钥长度假说、常见字节、已知明文片段及其对齐。原例考察长度 4 的重复密钥及明文字节 0x00。纯算法关系为 `K[i mod L] = C[i] XOR P[i]`；多个已知位置对同一密钥位的约束应相容。
---

## 古典密码

| 密码类型 | 工具 | 说明 |
|---------|------|------|
| Caesar | CyberChef / dcode.fr | 暴力 25 种偏移 |
| Vigenere | dcode.fr / Ciphey | 需要猜 key 长度 |
| Substitution | quipqiup.com | 频率分析自动求解 |
| Enigma | dcode.fr | 在线模拟器 |
| Rail Fence | dcode.fr / CyberChef | 栅栏密码 |
| Playfair | dcode.fr | 需要 key |
| Morse | CyberChef | 点划转文字 |
| Bacon | dcode.fr | 二进制隐写 |
| ROT13/47 | 字符映射 | 简单替换 |

---

## 编码识别与转换

| 编码 | 识别特征 | 解码方式 |
|------|---------|---------|
| Base64 | 末尾 `=` 或 `==`，字符集 A-Za-z0-9+/ | Base64 字节解码 |
| Base32 | 大写字母 + 2-7，末尾 `=` | 32 字符字母表、位分组与填充 |
| Base58 | 无 0/O/I/l，常见于 Bitcoin | 58 进制映射及具体字母表 |
| Hex | 只有 0-9a-f，长度为偶数 | 成对十六进制还原 |
| URL encoding | `%XX` 格式 | 百分号字节解码 |
| HTML entities | `&#XX;` 或 `&amp;` 格式 | 实体到字符的映射 |
| Unicode escape | `\uXXXX` 格式 | Unicode 转义解释 |
| JWT | `xxxxx.yyyyy.zzzzz`（三段 Base64URL） | 头、载荷与签名的编码/覆盖关系 |
| Brainfuck | 以 `><+-.,[]` 为指令 | 单元、指针、循环与 I/O 语义 |
| Ook! | `Ook.` `Ook!` `Ook?` 词元 | 词元对到相应指令语义的映射 |

---

## 逆向中的加密识别

### 通过常量识别算法

| 常量/特征 | 算法 |
|-----------|------|
| `0x67452301, 0xEFCDAB89, 0x98BADCFE, 0x10325476` | MD5 |
| `0x6A09E667, 0xBB67AE85, 0x3C6EF372` | SHA-256 |
| `0x63, 0x7C, 0x77, 0x7B` (S-Box 开头) | AES |
| `0x243F6A88` (π 的十六进制) | Blowfish |
| `0xB7E15163, 0x9E3779B9` | RC5/RC6/TEA |
| `0x61707865` ("expa") | ChaCha20/Salsa20 |
| `0xC6EF3720` | XTEA |

### 通过行为识别

| 行为特征 | 可能的算法 |
|---------|-----------|
| 256 字节查找表 + swap 操作 | RC4 |
| 16 字节块 + 多轮置换 | AES |
| Feistel 结构（左右交换） | DES/Blowfish/TEA |
| 大数乘法/模幂 | RSA |
| 椭圆曲线点运算 | ECDSA/ECDH |
| 固定 64 轮循环 | TEA/XTEA |
| 32 轮 + delta 常量 | XTEA |

---

## 自动化密码分析

| 工具 | 用途 | 历史标识或知识范围 |
|------|------|------|
| **FeatherDuster** | 自动化密码分析框架 | `github.com/nccgroup/featherduster` |
| **PkCrack** | ZIP 已知明文攻击 | `www.unix-ag.uni-kl.de/~conrad/krypto/pkcrack.html` |
| **bkcrack** | ZIP 已知明文攻击（现代版） | `github.com/kimci86/bkcrack` |
| **z3** | SMT 求解器（约束求解） | `github.com/Z3Prover/z3` |
| **angr** | 符号执行（自动求解输入） | `angr.io/` |

---

## 未知数据的分类依据

未知数据可以按相互独立的证据分类：全十六进制可能是编码或摘要；末尾等号与 Base64 字母表支持 Base64 假说；三段点分支持 JWT 结构；32/40/64 个十六进制字符分别与 MD5/SHA-1/SHA-256 的常见表示长度相符。

编码识别的结果是候选字节和层次；哈希识别的结果是算法与参数假说；RSA 的问题是参数数学结构；XOR 的问题是周期和密钥约束；自定义加密的核心是实现中的轮函数、数据流和比较方向。候选失败应回到原始字节与上下文，而不是固定切换某个工具。
---

## 历史资源与能力分工

| 资源 | 历史标识或知识范围 | 用途 |
|------|------|------|
| CyberChef | `gchq.github.io/CyberChef/` | 万能编解码 |
| dcode.fr | `www.dcode.fr/` | 900+ 密码工具 |
| quipqiup | `quipqiup.com/` | 替换密码自动求解 |
| factordb | `factordb.com/` | RSA 大数分解 |
| jwt.io | `jwt.io/` | JWT 解码/验证 |
| hashes.com | `hashes.com/` | 哈希反查 |
| crackstation | `crackstation.net/` | 在线哈希破解 |
