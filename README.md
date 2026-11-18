# ui-taste-loop

AI UI 设计三阶段 skill：seed 发散 → 锚定 A/B critic 闭环 → 减法交付。
A taste-driven UI loop for coding agents: seeded divergence, anchored A/B critique, subtractive delivery.

核心判断：LLM 默认输出「取悦平均审美的安全牌」（紫渐变、bento 卡片、营销黑话）。**模型不是瓶颈，工作流才是**——多样性从外部注入，评审用独立 fresh context 信号，交付靠减法。方法论源自 Anshu Chimala（前 Apple 设计工程负责人 12 年）的三阶段工作流，skill 化后经一次全流程实测修订了评分协议。

## 三个阶段

| 阶段 | 做什么 | 关键机制 |
|---|---|---|
| Phase 1 Discover | seed string（外部随机源）一次发散 4 个设计方向，人挑 1 个 | 多样性不靠模型自觉，靠 `/dev/urandom` 注入 |
| Phase 2 Build & Critique | 实现 → fresh-context critic 只看截图评审 → 修改循环 | 新旧两版**锚定 A/B 判决**为主信号，绝对分只记趋势；豁免清单防不可修项反复扣分 |
| Phase 3 Deliver | 减法：砍元素、AI Tells 审计（`npx impeccable detect`）、Copy Audit 文案轴（同趟 lint + 中文 grep 词表 + 人填真值） | AI 只会加不会减，克制本身就是高级感 |

## 为什么评分协议长这样（本 skill 的差异化）

首次全流程实测（4 页面 × 5 轮 critic）发现：fresh critic 每轮换人，**绝对分跨裁判不可比**——三个近乎像素冻结的页面，换裁判后分数漂移 ±0.2~1.2；五轮问题条数恒定 21-23（prompt 要求每页必出问题 → 分数被格式性封顶）；被契约禁止或待人工裁决的项被反复扣分形成天花板。于是：

- **A/B 判决替代绝对分驱动**：同一裁判眼里的新旧对比是唯一可比信号
- **复合退出条件替代裸 9/10**：「无新增缺陷 + 多数页更好」或「8.5±方差带内无新增缺陷」
- **连续 2 轮全页持平/更差 → 交人裁决**：实测中 polish 修不掉的最大扣分项，是人拍板撤掉一个元素后解锁的（单页 +1.1，全程最大涨幅）
- **豁免清单**：契约禁项/已裁决保留项/静态截图盲区，循环开始一次性声明，不逐轮给 critic prompt 打补丁

完整数据与四机制分析：[docs/evaluation-2026-11-18.md](docs/evaluation-2026-11-18.md)

## 安装

适用于支持 skill 目录的 coding agent（ZCode / Claude Code / Codex 等，Markdown skill 格式通用）：

```sh
git clone https://github.com/shanyuhai007-netizen/ui-taste-loop.git
# ZCode
mkdir -p ~/.zcode/skills && cp -r ui-taste-loop/ ~/.zcode/skills/ui-taste-loop
# Claude Code
mkdir -p ~/.claude/skills && cp -r ui-taste-loop/ ~/.claude/skills/ui-taste-loop
```

（SKILL.md 内嵌的可选生图步骤依赖火山 Seedream API，`ARK_API_KEY` 环境变量即可，不需要可整段跳过。）

## 使用

面向用户的界面生成或视觉改版任务（landing page、面板页、组件视觉重做）先走本流程；纯逻辑实现、无视觉产物的任务不适用。与测试先行（行为轴）、代码评审（代码轴）在同一张票内并列——**完成判定 = 测试全绿 + critic 评审通过 + code-review 通过，三者缺一不可**。

细节（seed string 操作、critic prompt 模板、循环纪律、AI Tells 审计）见 [SKILL.md](SKILL.md)。

## 状态

- 流程骨架（三阶段、seed 发散、减法交付）与「两轮不涨交人裁决」：**已实测有效**
- 锚定 A/B 评分协议、豁免清单、问题三分类：**实测驱动修订，尚未二次实测**（下次 UI 票跑完做对照验证）
- Copy Audit 文案轴与 critic 文案评审项（impeccable 文案规则 + 中文词表初版 + 人填真值）：**调研驱动设计，尚未实测**（词表随实测迭代）
- critic 需要 agent 具备子 agent 派发与截图能力（浏览器截图或实现侧先渲染）

## 来源与致谢

- 方法论原文：Anshu Chimala《[How to turn your AI into a world-class designer](https://www.lennysnewsletter.com/p/how-to-turn-your-ai-into-a-world)》（Lenny's Newsletter, 2026-09-01，[免费镜像](https://github.com/CodingRookie98/cr-agent-arsenal/blob/master/docs/reference/articles/how-to-turn-your-ai-into-a-world-class-designer.md)）
- 同源 skill 化成品（image-gen 闭环向）：[achimala/dream-loop](https://github.com/achimala/dream-loop)
- 本仓库在三阶段骨架之外的部分（锚定 A/B 协议、豁免清单、复合退出条件、问题三分类、Copy Audit 文案轴、评测数据）为独立工程化裁量

## License

[MIT](LICENSE)
