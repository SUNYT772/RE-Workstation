# 加密常量库

> 搜索到可疑字节序列时，在此对照匹配

## AES

```
S-box (标准):
63 7c 77 7b f2 6b 6f c5 30 01 67 2b fe d7 ab 76
ca 82 c9 7d fa 59 47 f0 ad d4 a2 af 9c a4 72 c0
b7 fd 93 26 36 3f f7 cc 34 a5 e5 f1 71 d8 31 15
04 c7 23 c3 18 96 05 9a 07 12 80 e2 eb 27 b2 75
09 83 2c 1a 1b 6e 5a a0 52 3b d6 b3 29 e3 2f 84
53 d1 00 ed 20 fc b1 5b 6a cb be 39 4a 4c 58 cf
d0 ef aa fb 43 4d 33 85 45 f9 02 7f 50 3c 9f a8
51 a3 40 8f 92 9d 38 f5 bc b6 da 21 10 ff f3 d2
cd 0c 13 ec 5f 97 44 17 c4 a7 7e 3d 64 5d 19 73
60 81 4f dc 22 2a 90 88 46 ee b8 14 de 5e 0b db
e0 32 3a 0a 49 06 24 5c c2 d3 ac 62 91 95 e4 79
e7 c8 37 6d 8d d5 4e a9 6c 56 f4 ea 65 7a ae 08
ba 78 25 2e 1c a6 b4 c6 e8 dd 74 1f 4b bd 8b 8a
70 3e b5 66 48 03 f6 0e 61 35 57 b9 86 c1 1d 9e
e1 f8 98 11 69 d9 8e 94 9b 1e 87 e9 ce 55 28 df
8c a1 89 0d bf e6 42 68 41 99 2d 0f b0 54 bb 16

Inv S-box (逆向):
52 09 6a d5 30 36 a5 38 bf 40 a3 9e 81 f3 d7 fb
7c e3 39 82 9b 2f ff 87 34 8e 43 44 c4 de e9 cb
54 7b 94 32 a6 c2 23 3d ee 4c 95 0b 42 fa c3 4e
08 2e a1 66 28 d9 24 b2 76 5b a2 49 6d 8b d1 25
72 f8 f6 64 86 68 98 16 d4 a4 5c cc 5d 65 b6 92
6c 70 48 50 fd ed b9 da 5e 15 46 57 a7 8d 9d 84
90 d8 ab 00 8c bc d3 0a f7 e4 58 05 b8 b3 45 06
d0 2c 1e 8f ca 3f 0f 02 c1 af bd 03 01 13 8a 6b
3a 91 11 41 4f 67 dc ea 97 f2 cf ce f0 b4 e6 73
96 ac 74 22 e7 ad 35 85 e2 f9 37 e8 1c 75 df 6e
47 f1 1a 71 1d 29 c5 89 6f b7 62 0e aa 18 be 1b
fc 56 3e 4b c6 d2 79 20 9a db c0 fe 78 cd 5a f4
1f dd a8 33 88 07 c7 31 b1 12 10 59 27 80 ec 5f
60 51 7f a9 19 b5 4a 0d 2d e5 7a 9f 93 c9 9c ef
a0 e0 3b 4d ae 2a f5 b0 c8 eb bb 3c 83 53 99 61
17 2b 04 7e ba 77 d6 26 e1 69 14 63 55 21 0c 7d
```

## 散列函数初始值

```
MD5 IV (小端):
  A=0x67452301, B=0xefcdab89, C=0x98badcfe, D=0x10325476
  字节: 01 23 45 67  89 ab cd ef  fe dc ba 98  76 54 32 10

SHA-1 IV:
  0x67452301, 0xefcdab89, 0x98badcfe, 0x10325476, 0xc3d2e1f0

SHA-256 IV:
  0x6a09e667, 0xbb67ae85, 0x3c6ef372, 0xa54ff53a
  0x510e527f, 0x9b05688c, 0x1f83d9ab, 0x5be0cd19

SHA-512 IV (前4个):
  0x6a09e667f3bcc908, 0xbb67ae8584caa73b
  0x3c6ef372fe94f82b, 0xa54ff53a5f1d36f1
```

## BoringSSL / OpenSSL 识别

```
BoringSSL: libssl.so 极简 (Android 用)
  → 观察 SSL_write 等导出/动态解析线索与调用关系

OpenSSL: libssl.so + libcrypto.so (标准)
  → 导出符号与模块依赖可提供定位线索
```

## CRC 表 (常见)

```
CRC-32 表开头:
0x00000000, 0x77073096, 0xee0e612c, 0x990951ba
0x076dc419, 0x706af48f, 0xe963a535, 0x9e6495a3

CRC-16 表 (常用于固件):
0x0000, 0x1021, 0x2042, 0x3063, 0x4084, 0x50a5, 0x60c6, 0x70e7
```

## 自研密码案例

```
TC3 (garden-bot):
  magic: 74 63 03 (tc\x03)
  S-box: 256B 自定义 (非标准 AES S-box)
  key:   原案例为20字节密钥，具体值不随通用包分发
```
