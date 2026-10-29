# Shell APK 解包材料的知识转写

来源：skills/apk-reverse/scripts/decode.sh

## 主题与范围

保留 Shell 原材料的双视图与摘要语义，以及与 Windows 版本的差别。

## 知识正文

### 产物身份

输入 APK、可选任务名和输出根，以及是否包含 Java/资源视图决定材料范围。原文默认从 APK 的文件名去除 .apk，将字母、数字、点、下划线和连字符以外字符替为下划线；输出根默认 APK 父目录。任务下 jadx、apktool 两分区分别承载 Java 与资源/smali。

### 摘要语义

task_root、jadx_out、apktool_out 是材料位置；package 来自 Manifest 的 package 属性文本；java_files 是递归 .java 数；smali_dirs 是顶层 smali* 目录数；so_files 是递归 .so 数。本版本未输出 xml_files，与 Windows 页的统计面不同。

### 状态差别

本版本初始化两个视图的退出状态为零，跳过某视图时仍可能保留零值；这不同于 Windows 材料的空状态。零不能单独证明对应视图实际形成。Manifest 文本匹配与 XML 解析也不同，包名提取失败不应改写成“无包名”。

## 观察与判断

局部反编译失败与部分源码可用可共存；统计需与所含视图、文件存在性和任务身份一起解释。自动安装与输出清理属于执行资产，未转写为操作要求。

## 记录与报告

保留 APK 身份、视图范围、原状态表示、计数口径及包名来源；跨版本对比时注明状态和 XML 统计差异。
