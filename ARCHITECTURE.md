
# Neryven IME — Architecture

## 第一原则

输入主链拥有绝对优先级：

```text
按键 → Fcitx/libime → 候选 → 上屏
                     │
                     └─ Companion 只做旁路观察/事件，不得阻塞
```

任何网络、数据库、Memory、Harness、MCP、模型或 Companion UI 故障，都必须允许普通输入继续工作。Context/Event 采集必须异步、非阻塞、有界；瞬时事件允许丢弃，不能形成输入热路径上的背压。具体延迟预算以 V0/V1 真机基线测量后确定，不凭空写死。

## 目标分层

```text
Android IME / UI
├─ Fcitx Input Core          上游优先，尽量少改
├─ Context Engine            本地，规则优先
├─ Privacy Gate              本地，敏感数据硬拦截
├─ Event Store               本地、TTL、离线可用
├─ Companion UI              Avatar / Peek / 快速交互
├─ Companion Runtime         Standalone 模式
├─ Memory Store              Standalone 才拥有长期 Memory
└─ Network Gateway           唯一联网边界
   ├─ Model Providers
   └─ Harness Bridge
```

V1 不要求以上都一开始拆成独立 Gradle module。先保持职责边界，只有当隔离/测试/复用收益明确时再物理拆模块。

## Harness / Side Channel

```text
                    同一个 Harness / Agent
                           │
          ┌────────────────┼────────────────┐
          │                │                │
        main              ime             sims4
       channel        side channel      side channel
          │                │                │
          └────── Recall Router / Memory ───┘
```

- 每个 channel 有自己的原始历史、Working Context、Summary、token budget。
- 长期人格、长期 Memory、工具能力由 Harness 统一拥有。
- Side Channel 不把历史常驻塞入 main。
- Recall Router 先通过轻量索引判断相关性，再按需读取命中位置附近原始消息。
- V1 只要求 main ↔ ime 的最小 Side Channel；多 Side Channel、通用 Recall Router 与 Channel visibility 是 V2+ 泛化目标。
- 高置信度命中可直接取少量消息；低置信度不取，避免无意义工具调用和成本。
- Active Recall 是短期工作上下文，不是 Memory。

## Context / Clipboard Pipeline

```text
输入/剪贴板
   ↓
Privacy Gate
   ↓
本地分类 / 去重 / 长度与成本判断
   ↓
Event Store / 临时原文（按规则）
   ↓
必要时检索少量片段
   ↓
Network Gateway
```

- 密码/OTP/Token/支付/凭据等敏感内容不得进入普通 Event/Memory/Network。
- 长文本不整块发送给模型。
- 用户明确“给你看”属于高权限主动分享，但仍须经过 Secret 检查。

## 数据 Owner
- Fcitx 输入学习数据：Fcitx owner。
- Standalone Companion 长期 Memory：本 App owner。
- Harness 模式长期 Memory：Harness owner。
- Side Channel 原始聊天：对应 channel owner。
- Secure Vault：独立安全域；任何 Companion/Memory 组件都不是 Secret owner。

## 网络边界
- V1 单 APK。
- INTERNET 只服务 Network Gateway。
- 输入热路径不得 await 网络。
- Companion 关闭或未配置连接时，Network Gateway 不应产生 Companion 请求。
- 未来如果公开用户强烈要求“键盘 APK 无 INTERNET”，架构应允许再拆 Companion 网络进程/APK，但 V1 不提前承担该复杂度。

## 当前已确认的上游耦合风险
- `app/build.gradle.kts` 同时存在 namespace 与 applicationId；V0 只计划修改 applicationId。
- JNI/C++ 中大量硬编码 `org/fcitx/fcitx5/android/...` 类名，因此 V0 不应全量改 namespace/package。
- Manifest 中大部分 IPC permission / provider authority 使用 `${applicationId}`，有利于独立安装身份。
- `AndroidPluginAppConventionPlugin.kt` 已改为从统一的 `mainApplicationId` Gradle property 派生 release/debug 主 App identity；上游同步时必须继续保持它与主 App 的 debug suffix 规则一致。
- `lib/plugin-base` 的 query / metadata / MANIFEST action 已改为 `${mainApplicationId}` placeholder，并已用 `clipboard-filter` debug APK 验证解析到 `com.neryven.ime.debug`；官方插件二进制兼容仍不是 V1 目标。
- `PluginDescriptor.pluginPackagePrefix` 当前硬编码 `org.fcitx.fcitx5.android.plugin.`；V0 不全量改插件包名，先写入兼容说明，真正启用自有插件体系前再统一命名。
- `input_method.xml` 的 `settingsActivity` 是源码 namespace 下的类 FQN；因为 V0 不改 namespace，所以应保留并真机验证，不应跟着 applicationId 机械改名。
- F-Droid/README/CI 中存在官方包名引用；V0 不清理公开发行元数据，避免把身份迁移和发行工程混成一个任务。

## 技术未知项
以下必须通过代码/原型/测试回答，不靠讨论猜：
- applicationId 变更后所有 plugin/IPC/authority 的实际兼容结果；
- Windows 构建环境是否完整；
- Companion DB 的最终加密方案；
- Harness Bridge 的最终 transport；
- Android 上 MCP Server 的可行 transport（V2+）；
- Compose 在 IME 热路径中的实际性能与启动代价。