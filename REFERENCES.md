# Neryven IME — Reference Repositories

原则：实现前先查已有实现；优先借思路、边界与接口，不为了“原创感”重复造轮子。

## 主底座
### fcitx5-android/fcitx5-android
用途：
- Android IME 生命周期
- Fcitx/libime/chinese-addons
- 候选、词库、主题、剪贴板、插件框架
- 输入热路径与上游同步

本项目主仓来源于该项目。上游代码应尽量少魔改。

## 重点参考
### yuyu1838309000-cmd/Xime（上游 ximeiorg/Xime）
重点看：
- Kotlin / Compose 输入法 UI
- Rime 集成
- 本地 AI
- 插件/同步设计

许可证：GPL-3.0。默认只读研究；不得未经 license review 直接搬代码进 LGPL 主工程。

### yuyu1838309000-cmd/Deskdrop（上游 SvReenen/Deskdrop）
重点看：
- AI keyboard 交互
- Provider 抽象
- MCP / Ollama / 云模型
- 本地/远端模型切换

许可证：GPL-3.0。默认只读研究；复制代码前必须 license review。

### hc-tec/fcitx5-android
重点看：
- 在 Fcitx5 Android 上接 AI/Function Kit 的具体施工方式
- 哪些上游扩展点可复用

用途以 patch/架构参考为主，不作为第二主线 fork。

### Shan-HIT/HuoziIME
重点看：
- 端侧 LLM
- 个性化输入
- 本地推理与研究思路

许可证：GPL-3.0。偏研究参考，不作为产品底座。

## 使用流程
每个新功能开工前：
1. 查主工程是否已有能力/扩展点。
2. 查上述 reference repos 是否已有成熟实现。
3. 记录可复用的思路、依赖、许可证和耦合。
4. 能通过已有扩展点完成就不要改 Fcitx 核心。
5. 真要复制代码，先做 license compatibility check。
6. 实在没有再自己实现。

## 不做
- 不把 reference repos 混进主工作树。
- 不大块 cherry-pick GPL 代码到 LGPL 主工程。
- 不因为参考项目用了某技术就自动引入；必须先证明对当前需求有收益。
