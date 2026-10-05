# Neryven IME — START HERE

这是 Neryven IME 的唯一长期接班入口。

## 当前状态
- 阶段：V0 foundation。
- 项目家族前缀：`Neryven`；输入法最终用户产品名仍是 TBD，当前工程名 `Neryven IME`。
- 主仓：`yuyu1838309000-cmd/fcitx5-android`
- 本地仓：`E:\NereviaIME`（历史本地目录名，仅本机路径，不代表品牌）
- 上游：`fcitx5-android/fcitx5-android`
- 长期 Android applicationId：`com.neryven.ime`；debug 安装身份为 `com.neryven.ime.debug`。
- 当前源码 namespace / Kotlin package 暂不全量改名，优先保持上游结构。
- DeepSeek V0 预审已完成：方向 PASS WITH CHANGES。
- 未改包名的 arm64 debug baseline build 已 PASS；`com.neryven.ime.debug` 改名后的完整重构建已 PASS；`clipboard-filter` 插件跟随新主包 ID 构建也已 PASS。
- APK 已安装到真机并启用；`Neryven IME (Debug)` 已成为当前默认输入法。
- 真机 P0 功能实测 PASS：简中全拼、候选上屏、中英切换、数字/标点、删除、回车、候选栏/翻页、设置入口均正常；测试后进程仍存活，近期无 Neryven 崩溃日志。
- “与官方 Fcitx5 同时安装”尚未做双安装实测；当前只能确认 package identity 已独立。
- 输入手感当前不够顺手，明确留到 V1 的键盘层/交互层调优，不作为 V0 blocker。
- V0 acceptance review 已完成：`PASS WITH CHANGES`，无 BLOCKER；Reviewer 提出的文档一致性、debug/release 品牌说明、debug suffix 同步风险与 manifest BOM 均已处理。

## 真相优先级
当前代码 / git / tests / runtime > 本文件当前状态 > DECISIONS.md / ARCHITECTURE.md / ROADMAP.md / UPSTREAM_COMPATIBILITY.md > REFERENCES.md > 旧聊天或旧方案。

## 新窗口 / 新 Agent 启动顺序
1. 读本文件。
2. 读 `DECISIONS.md`。
3. 读 `ARCHITECTURE.md`。
4. 只按当前阶段读取 `ROADMAP.md`、`UPSTREAM_COMPATIBILITY.md` 和必要代码。
5. 用 git、代码、构建与真机结果校验文档，不根据聊天记忆猜当前进度。

## 工作规则
- Upstream-first：Fcitx5/libime 是持续更新的底座，不是一次性复制源码。
- 输入主链优先：任何 Companion / 网络 / Memory / UI 失败都不得阻塞打字。
- 实现前先查主工程与 REFERENCES.md 中的参考项目；避免重复造轮子。
- GPL 参考仓默认只读研究；复制代码前必须单独做 license check。
- dirty / untracked work 受保护；不要为了“清爽”覆盖未知改动。
- 设计阶段：ChatGPT 主导整合，DeepSeek Reviewer 做独立 challenger；只有出现新的 BLOCKER 证据或关键结论冲突时才复审，不固定安排第二轮。
- 实施阶段：Codex/Builder 主做；Reviewer 不与 Builder 抢同一工作区。
- 验收阶段：Reviewer 独立审 diff / tests / 架构偏移，ChatGPT 做最终裁决。
- 不为流程本身增加流程；发现重复、过度设计或无收益步骤时主动删减。

## 当前下一步
1. 提交 V0 foundation 变更；不把 P1 手感问题混进这个基线提交。
2. 进入 V1，第一优先级是输入手感与键盘层/候选层体验，而不是先叠 Companion。
3. P1 剩余人工项（Emoji、用户词频/学习、第二个常用 App）在 V1 日用基线里补测。
4. 官方 Fcitx5 + Neryven 双安装实测保留为发行/兼容验证项；当前 applicationId、authority、IPC 与 plugin identity 已静态和构建产物验证独立。
