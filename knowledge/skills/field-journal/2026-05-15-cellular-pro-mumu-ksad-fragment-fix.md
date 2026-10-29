# 2026-05-15 Android 模拟器崩溃与 Fragment 生命周期

来源：skills/field-journal/2026-05-15-cellular-pro-mumu-ksad-fragment-fix.md

## 案例问题与环境

原案例为 Cellular-Pro 重建 APK 在 MuMu 12 上的启动失败。隐私同意或启动页之后的崩溃并非单一故障：第三方广告 SDK 的设备指纹/广告网络路径与模拟器翻译层不兼容，后续又出现应用 Fragment 生命周期语义受损。

## 观察与决策

第一层现象涉及 Kuaishou / KSAd 的设备信息、指纹和网络路径。源材料列出的包/类为 `com.yxcorp.kuaishou.addfp`、`com.kwad.sdk.utils.bc`、`com.kwad.sdk.core.network`，它们帮助把崩溃栈归到第三方 SDK。第一处 native 崩溃消失后，启动推进到更晚路径，仍暴露其他 SDK 兼容性问题，说明“第一处不再崩溃”并不是应用启动已完整恢复。

## 失败路径与修复机制

第二层故障为 `SuperNotCalledException`。应用 Fragment 的 `onResume()` 被替换成空操作，只剩无返回值结束语义，使 AndroidX Fragment 约定的父类生命周期状态未更新。源修复恢复对 `androidx.fragment.app.Fragment` 父类方法的直接调用。技术要点是生命周期状态机依赖父类行为，不能以“函数能返回”代替协议完成。

## 验证与可迁移经验

原记录的两层修复分别处理模拟器中的第三方 SDK 路径与应用自身的 Fragment 契约。可复核的观察是崩溃栈从 native/SDK 转到 Fragment、父类调用是否存在、启动能否跨过后续阶段。原文没有附全量日志、每类方法清单或完整回归结果，因此不补造测试通过范围。

对重建 Android 应用的可迁移知识是：模拟器专有失败、第三方采集路径、空方法替换和生命周期异常可能是相邻但不同的根因；修复一层后新露出的故障不应再次笼统归为同一 native 问题。相关知识领域为 APK 逆向与报告，原双括号包外路由不作链接。
