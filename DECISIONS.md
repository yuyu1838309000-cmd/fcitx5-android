# Neryven IME — Decisions

只记录已经定下的产品/工程结论。未定问题不要伪装成决定。

## 产品定位
- Android 中文主力输入法优先，AI Companion 是可选扩展。
- V1 目标是“愿意长期日用的中文输入法 + 最小可用 Companion”，不是 AI Demo。
- Companion 总开关默认 OFF；主动消息默认 OFF；Context 感知与主动消息分离。
- 敏感场景（密码、OTP、支付、凭据等）硬屏蔽。

## 底座与身份
- Android 主底座：Fcitx5 Android + libime / fcitx5-chinese-addons。
- 主仓开发期继续使用 fork；产品成熟后是否迁移 canonical repo 再决定。
- 长期 `applicationId`：`com.neryven.ime`。
- V0 只改 applicationId 等安装身份相关项；不为了品牌感全量重命名 Kotlin/Java namespace。
- 官方 Fcitx5 插件 APK 的二进制兼容不是 V1 目标；独立产品身份优先。
- V0 使用 debug signing；进入 V1 长期日用前再建立正式 release signing。
- V0 debug 构建统一显示 `Neryven IME (Debug)`，临时接受各语言环境都显示英文开发标签；release 在正式产品命名完成前仍保留上游 Fcitx5/小企鹅输入法品牌资源，且 V0 不发行 release 包。

## 输入与 UI
- V1 聚焦简体中文全拼 + 英文 + 数字符号/Emoji；其他上游能力尽量保留但不是验收重点。
- 不重写 Fcitx/libime 输入算法，除非有明确证据必须改。
- V1 不做全量 Compose 重写；输入热路径优先稳定，新页面可使用 Compose。
- 键盘顶部提供 Companion avatar / Peek；头像进入产品主页，具体 Peek 可直达聊天。
- 键盘只做 Peek、快速回复、“给你看”和少量快捷动作；完整聊天在主 App。

## 网络与 Provider
- V1 单 APK，允许 INTERNET；网络能力集中在 Network Gateway。
- 无 API/Harness 配置或 Companion 关闭时，不产生 Companion 网络请求。
- Provider 按协议支持：OpenAI Chat Completions、OpenAI Responses、Anthropic Messages、Gemini。
- 支持中转站：自定义 Base URL / API Key / Model / Headers / Path，并提供连接测试。
- Provider 与 Character 分离；Maintenance Model 可单独配置。

## Companion / Harness
- Standalone API 模式：输入法自己提供轻量 Companion Runtime、聊天、Summary、Local Memory。
- Harness 模式：V1 做 Basic Harness Bridge。
- Harness 模式拥有独立 IME Side Channel，不是主聊天镜像。
- 任一时刻长期 Memory 只能有一个可写 owner：Standalone 时由本 App 拥有，Harness 模式由 Harness 拥有；模式切换只能走“迁移 / 归档保留 / 重新开始”，禁止双写。
- Side Channel 原始历史独立保存；主窗口通过 Pull-based Recall 按需读取。
- 召回默认不常驻主 Context；命中后进入短期 Active Recall，同话题沿用，切题/TTL 后移出。
- V1 只要求 main ↔ ime 的最小 Side Channel 闭环；多 Side Channel 泛化、通用 Recall Router 与 Channel 可见性策略放 V2+。
- 正常聊天不显示召回痕迹；高级 Context/Recall 检查器保留为后续能力，不阻塞 V1。

## Context / 隐私
- 不做 keylogger 式逐键持久化。
- Raw Draft Buffer 只在内存中；草稿感知默认 OFF。
- Context 感知默认只对明确授权 App 生效。
- 剪贴板感知默认 OFF，用户单独开启。
- 剪贴板先本地隐私过滤；Secret / OTP / Token / 密码等直接 DROP 或脱敏，宁可少记。
- 长剪贴板不整段发模型：本地保存/分块/索引，默认只发 metadata/摘要，需要时取少量相关片段。
- 普通剪贴板原文默认 TTL 7 天；普通 Context Event 默认 TTL 30 天；均允许用户调整。
- 离线时本地 Event Store 继续工作；普通键入不长期保存整段原文；联网后只处理仍有价值的事件和明确待发送内容。
- Fcitx 个人词频/词库与 Companion 数据默认隔离，个人词库不属于 AI Context。

## Memory / Context Budget
- Storage != Context。
- 完整聊天历史保存在本地数据库；UI 使用分页/虚拟化，不按天清空历史。
- Working Context 按 token budget 管理，不按固定轮数。
- Summary 用于连续性和检索，不替代原始历史；失败不能阻塞聊天或删除数据。
- 维护/摘要模型与主对话模型分离配置。
- 召回内容不会因为“被召回”自动升级为长期 Memory。

## 本地安全
- API Key / Token / Harness 凭据使用 Android Keystore 保护。
- Companion 私密数据默认要求落盘加密；具体数据库加密技术施工前评估。
- V1 零遥测、零自动崩溃上传；诊断默认本地生成，用户主动导出。
- Companion 数据可独立彻底清空，不影响普通 Fcitx 输入学习数据。

## Secure Vault（V2 方向）
- Vault 与 Companion Memory 完全隔离。
- Secret 明文必须是 NEVER_CONTEXT / NEVER_NETWORK / NEVER_MEMORY。
- LLM 最多知道允许暴露的非敏感 metadata；单条可设为“连存在都不可见”。
- PIN 可作便捷解锁入口，但不能作为真实数据加密密钥。
- 以后支持认证后本地填入当前登录字段，Secret 明文不经过 LLM。
