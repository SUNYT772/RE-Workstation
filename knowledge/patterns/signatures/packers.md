# 加固/壳特征库

> 文件名、DEX形态和资源组合是识别线索，版本变化可能改变特征。

## 芒果加固 (com.mx.shell)

```
识别特征:
  lib/arm64-v8a/libshell.so            (壳 SO)
  lib/armeabi-v7a/libshell.so
  lib/arm64-v8a/libtobEmbedEncrypt.so  (加密 SO，常与壳同目录)
  classes.dex 极小 (< 1KB) 或不存在

VMP特征:
  Modo SDK 的 Java 类不可读 (VMP保护)
  React Native 框架类通常不受保护

分析关注点:
  运行时已还原代码、加载时机、内存结构、保护范围与检测分支。


实战案例:
  花园世界 (cn.lbwdhysj.gf) — 芒果 + VMP
```

## 360加固

```
识别特征:
  lib/armeabi-v7a/libjiagu.so
  lib/armeabi-v7a/libjiagu_a64.so  (v2)
  assets/jiagu_data.bin

分析关注点:
  运行时已还原代码、加载时机、内存结构、保护范围与检测分支。

```

## 腾讯乐固

```
识别特征:
  lib/arm64-v8a/libshell-super.so
  lib/arm64-v8a/libshell-super.2.so
  lib/armeabi-v7a/libshell-super.so
  lib/armeabi-v7a/libshella-*.so
  assets/tos_lcg 或 assets/0OO00l111l1l

分析关注点:
  运行时已还原代码、加载时机、内存结构、保护范围与检测分支。

```

## 网易易盾

```
识别特征:
  lib/arm64-v8a/libprotect.so
  lib/arm64-v8a/libnesec.so 或 libNetHTProtect.so
  assets/tos*.dat
```

## 梆梆加固

```
识别特征:
  lib/arm64-v8a/libDexHelper.so
  lib/arm64-v8a/libDexHelper-x86.so
  classes.dex 被修改为其他名
```

## 爱加密

```
识别特征:
  lib/arm64-v8a/libexec.so
  lib/arm64-v8a/libexecmain.so
  assets/ijiami.dat
```

## 几维安全

```
识别特征:
  lib/arm64-v8a/libkwscmm.so
  lib/arm64-v8a/libkwscr.so
```

## 阿里聚安全

```
识别特征:
  lib/arm64-v8a/libmobisec.so
  lib/arm64-v8a/libmobisecy.so
```

## 娜迦加固

```
识别特征:
  lib/arm64-v8a/libchaosvmp.so
  lib/arm64-v8a/libddog.so
  lib/armeabi-v7a/libnqshield.so
```

## DEX 整体加固 (百度/DexProtector)

```
识别特征:
  classes.dex 正常但内容不是标准 DEX 头
  assets/ 下有加密的原始 DEX
```
