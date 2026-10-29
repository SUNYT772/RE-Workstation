# Manifest 摘要字段与主入口判据

来源：skills/apk-reverse/scripts/manifest-summary.ps1

## 主题与范围

Android Manifest 的命名空间、权限、组件与启动 activity 识别规则。

## 知识正文

### XML 命名空间

package 从 manifest 根属性取得；Android 的 name/exported/enabled 属性属于 http://schemas.android.com/apk/res/android 命名空间。该 URI 是 XML 标识符，不是需要连接的服务。忽略命名空间可能让存在的属性读成空值。

### 摘要字段

package 表示包名。uses-permission 节点数为 permission_count，各非空 android:name 为 permission。application 下 activity、service、receiver、provider 分别记录数量，条目由 name、exported、enabled 三项组成。

### 主启动 activity

同一个 activity 的同一个 intent-filter 同时包含 action 的 android.intent.action.MAIN 和 category 的 android.intent.category.LAUNCHER 时，该 activity 进入主入口集合；源材料对最终名称去重，输出 main_activity。判据不是分别在无关 filter 找到两个词。

## 观察与判断

源摘要只查询 application/activity 等显式节点，不覆盖 activity-alias；空 exported/enabled 表示未显式读取到值，不等于 false。原脚本对缺失或无效 XML 不生成完整摘要，故“无摘要”不能当成“无权限/无组件”。

## 记录与报告

保留 package、权限数量/名称、各组件数量与三项属性、主入口以及覆盖限制；需要区分声明值和系统实际解析行为。
