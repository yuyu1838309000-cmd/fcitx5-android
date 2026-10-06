# Neryven IME — Roadmap

## V0 — 独立、可构建、可日用的 Fcitx 基线 ✅ CLOSED

目标：证明 fork 能安全成为 Neryven IME 的工程底座。V0 不做 AI，不大改 UI。

### 第 0 步：先证明上游基线能构建
1. 建立 upstream remote 与同步规则。
2. 恢复/确认全部必要 submodule。
3. 固定 Windows 工具链：CMake 3.31.6、NDK 28.0.13004108、Build-Tools 36.1.0、Android 36、MSYS2 UCRT64 gettext/ECM、Gradle wrapper 9.6.1。
4. 在**未改 applicationId**的状态先完成 arm64 debug baseline build。

### applicationId 最小迁移
5. 审计 applicationId、plugin、IPC permission、provider authority、JNI/package 耦合。
6. 将 release applicationId 改为 `com.neryven.ime`；V0 debug 继续使用上游 `.debug` suffix，因此调试安装身份为 `com.neryven.ime.debug`。
7. 暂不全量改 namespace/Kotlin package/JNI 类名。
8. repo 内插件若继续构建，其 MAIN_APPLICATION_ID / Manifest action 必须跟随新的主 App identity；插件自身 package prefix 暂不迁移。
9. 保持 debug signing；V0 不承载需要迁移的重要 Companion 数据。

### 真机验收
**P0 必过：**
- [x] APK 可安装/启用。
- [ ] 与官方 Fcitx5 Android 同时存在（尚未做双安装实测；package identity 已独立）。
- [x] 简中全拼可输入并上屏候选。
- [x] 中英切换。
- [x] 删除/回车。
- [x] 设置入口可打开。

**P1 随后验证：**
- [x] 候选选择/翻页。
- [x] 英文、标点、数字。
- [ ] Emoji。
- [ ] 用户词频/词库基本正常。
- [ ] 至少 2 个常用 App 输入无明显退化。

证据至少记录 APK SHA256、package/dumpsys 信息、最终权限列表与 P0/P1 结果。

### V0 收尾
- 记录 Upstream Compatibility Notes：改了哪些上游关键点、为什么、同步时检查哪里。
- F-Droid / Transifex / README 品牌元数据不属于 V0，不为了“看起来独立”提前重写发行体系。

### V0 Stop Gate
- 未改包名的 baseline build 不通过：先修构建，不叠功能。
- applicationId 修改需要大规模 namespace/JNI 重写：暂停，重新评估方案。
- 输入主链未稳定：不进入 V1。

## V1 — Neryven IME 第一版产品形态

目标：用户愿意长期用的中文输入法 + 最小可用 Companion。

### 输入/UI
- V1 第一阶段先完成 `V1_INPUT_EXPERIENCE.md`：同机参考采样 → 冻结输入体验 Brief → 再实现；在此之前不靠散改参数试手感。
- 第一优先级是达到“愿意长期日用”的键盘几何、候选层与常用交互；明显区别于上游的视觉放在手感稳定之后。
- 不全量 Compose 重写；优先保留稳定输入热路径。
- 顶部 Companion avatar / Peek 在普通输入体验通过后再加入。
- 主 App 主页、完整聊天、角色/Provider/隐私设置。

### Standalone
- OpenAI Chat Completions / Responses / Anthropic Messages / Gemini。
- 中转站自定义 Base URL/Key/Model/Headers/Path。
- 本地聊天历史、Summary、Local Memory。
- token budget Context Manager；Maintenance Model 独立配置。

### Context / Privacy
- Privacy Gate。
- per-app 授权。
- 简单 Context Event。
- 剪贴板 opt-in、敏感过滤、长度/成本控制。
- 离线 Event Store。
- Context/Event 采集必须异步、非阻塞、有界；瞬时事件允许丢弃，禁止拖慢输入主链。
- Companion 敏感数据加入前必须确定 Keystore、数据库加密和 Android backup 规则。

### Harness
- V1 只做 main ↔ ime 的最小 Basic Harness Bridge：
  - IME → privacy-filtered Context Event
  - IME Side Channel ↔ Harness
  - Harness → Companion Peek/response
- Bridge 必须能用本地 stub/mock 验收，不能让 V1 完全依赖外部 Harness 在线。
- 不做复杂 Memory 双向同步。
- 不做通用多 Side Channel/Recall Router。
- 不做 MCP Server。
- 不做工具生态。

## V2+
- 多 Side Channel / Recall Router 泛化、Channel 可见性策略。
- 高级 Context/Recall 检查器。
- 本地 embedding / RAG / 可选本地小模型。
- MCP Server（先验证 transport）。
- Secure Vault 与正式 Autofill/Credential integration。
- 云同步/账号（若未来真的需要）。
- 正式公开发行、完整 license audit、隐私政策、升级迁移、更多 ROM/设备兼容。

## 发行节奏
- V0：开发验证，只给自己。
- V1：以本人长期日用为第一验收，可少量朋友内测。
- Public Beta：在 V1 真正稳定后再考虑，不为了上架提前增加行政/兼容负担。
