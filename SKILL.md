---
name: ui-taste-loop
description: AI UI 设计三阶段工作流（源自 Anshu Chimala 方法论）。Discover 用 seed string 注入外部随机性发散 4 个方向卡（自带配色素描/字体配对/布局隐喻），Build 前先立 design token 契约（craft baseline，[references/craft-baseline.md](references/craft-baseline.md)），Critique 用 fresh-context critic 只看桌面+移动双视口截图做锚定对比评审（新旧两版 A/B 判决为主信号、绝对分仅记趋势、问题按六维组织），迭代到复合退出条件，Deliver 做减法、AI Tells 审计与文案轴 Copy Audit（lint + 人填真值）。面向用户的界面生成或视觉改版任务（landing page、面板页、组件视觉重做）先走本流程；纯逻辑实现、无视觉产物的任务不适用。
---

# UI Taste Loop

核心判断：LLM 默认输出「取悦平均审美的安全牌」（紫渐变、bento 卡片、营销黑话）。模型不是瓶颈，工作流才是——多样性从外部注入，评审用独立 fresh context，交付靠减法。

单文件入口 + references/ 渐进披露：本文件只保留编排骨架与循环纪律，载荷细节按需加载——[craft-baseline.md](references/craft-baseline.md)（方向卡/token 契约/检查单）、[critic-prompt.md](references/critic-prompt.md)（评审模板/豁免清单/状态抽查）、[copy-audit.md](references/copy-audit.md)（文案轴）、[seedream.md](references/seedream.md)（可选生图）。

## 与已装 skills 的分工

| skill | 阶段 | 职能 |
|---|---|---|
| ui-taste-loop（本 skill） | 全流程编排 + craft 基线 | 发散 → 评审 → 减法循环骨架；方向卡与 token 契约自带（references/craft-baseline.md） |
| frontend-design | 生成端增强 | 反默认风格校准与微文案写作节（craft 基线已自带，装了是增量约束） |
| emil-design-eng | 细节增强 | 动效频率、easing、组件反模式（在 craft 基线之上加深） |
| web-design-guidelines | 合规下限 | a11y/性能审计（review my UI）——管「对不对」 |
| impeccable（npx，未安装） | 审美下限（Phase 3 出场） | 确定性 slop/AI Tells lint——管「俗不俗」，不管「对不对」 |

不冲突：本 skill 管流程与 craft 基线，其余在各自阶段被调用。

降级：本表为作者环境的分工，流程本身不依赖这些 skill。craft 基线（方向卡 + token 契约 + 检查单）为本 skill 自带，无外部依赖——未安装 frontend-design / emil-design-eng 时基线仍然成立，仅失去增量约束（反默认校准、动效深度细节），风格反模式另由 critic 模板的 AI Tells 清单与 Phase 3 审计兜底，产品 UI 微文案两层规则以 [references/copy-audit.md](references/copy-audit.md) 为准；未安装 web-design-guidelines 时 a11y 审计人工对照 WCAG 或跳过；impeccable 为 npx 零安装，无依赖。

## 票内时序：与 /tdd、/code-review 的编排

本 skill 与 /tdd（行为轴）、/code-review（代码轴）是同一张 UI 票内并列的三条验证轴，不互相替代（示例为 Matt Pocock skills 语境，无此环境时映射到等价的测试先行/代码评审流程）：

1. **行为先行**：/tdd 红绿环把功能做绿——跑不起来的 UI 没法评审美
2. **视觉迭代**：行为绿 → token 契约 + 实现 → 桌面/移动双视口截图 → critic 锚定评审循环（Phase 2）
3. **减法收尾**：AI Tells Audit + Copy Audit 文案轴（lint + 人填真值）（Phase 3）
4. **代码轴收口**：/code-review 双轴照常跑；critique 轮次中的 CSS/结构改动若触碰行为，回到 /tdd 补红绿

**完成判定：一张 UI 票完成 = 测试全绿 + critic 评审通过（Phase 2 复合退出条件）+ Phase 3 收口（AI Tells/Copy Audit 命中清零或经裁决留档、真值槽位已填）+ code-review 通过，缺一不可。**

## Phase 1: Discover——发散，禁止直接收敛

**载体两种**：小改直接在票内跑完整三阶段；大方向探索走 /prototype detour——在 throwaway 原型里**只跑本 Phase**（seed 出 4 张方向卡供人挑选），选定后经 /handoff 折回主线，正式票内再跑 Phase 2/3。critic 9/10 是对正式实现的要求，throwaway 原型不跑完整评审循环。

1. **Seed string 注入随机性**（让模型「be random」无效，多样性必须来自模型外部）：用 shell 生成随机串 `LC_ALL=C tr -dc 'A-Za-z0-9' </dev/urandom | head -c 48`，让 agent 从字符串的子模式/重复字符/数字提取设计方向（配色基调、布局隐喻、字体气质），一次产出 **4 张方向卡**——字段规范（方向名/灵感源/配色素描/字体配对/布局隐喻/敢在哪）见 [references/craft-baseline.md](references/craft-baseline.md)，空话方向无法评审。用户挑选；生成的页面不得暴露字符串本身。
2. **Ambitious brief**：选定方向后，brief 要敢想——引用灵感源（某个游戏、某次室内设计趋势、某个艺术装置），而非「做一个好看的后台页面」。「这不可能行吧」的感觉恰是方向正确的信号。
3. **可选：生图做目标图/moodboard**（火山 Seedream，需 `ARK_API_KEY`）：脚本与注意事项见 [references/seedream.md](references/seedream.md)。

## Phase 2: Build & Critique——锚定评审闭环

1. **Token 契约**：写第一行代码前，把选定方向展开成六段 token 契约（color/type/spacing/layout/motion/principles），实现只取契约内的值。规范、craft 检查单与实测依据见 [references/craft-baseline.md](references/craft-baseline.md)——实测中 critic 反复挑的间距/字阶/色彩层次问题，靠每轮 fresh critic 修是最贵的路径，build 时一次做对；契约要改时同步改全部已实现处，不留漂移。
2. **Build**：按契约实现，frontend-design 约束下进行；moodboard/参考图可喂给实现 agent 当质量锚点（参考，不是抄）。完成后过 craft 检查单（确定性项命中即修，不流入 critic）。
3. **Critique**：写代码的 agent 不能自评设计（放不下自己的决策，无法 zoom out）。另起 **fresh-context subagent**，只喂截图、不给代码/实现/历史：R1 喂本轮各页，R2 起喂上一轮+本轮两组做锚定对比——裁判每轮更换，绝对分跨裁判不可比（实测裁判间方差 ±0.5~1.2），可比信号只有同一裁判眼里的新旧两版。**截图协议（双视口）**：桌面全页 + 移动 375px 视口各一组，关键页（首页、信息密度最高页）移动视口必截——只截桌面 happy path 会把质量上限封在 critic 看不到的面上。prompt 模板、豁免清单与状态抽查方式见 [references/critic-prompt.md](references/critic-prompt.md)。截图用浏览器截取或让实现 agent 先渲染。
4. **循环纪律**：
   - 驱动信号 = 判决 + 新增缺陷数；绝对分只记趋势，不单独触发任何动作
   - 收口（其一）：critic 明说「无新增缺陷」且多数页判「更好」；或绝对分进入 8.5±0.5 带且当轮无新增缺陷。9/10 是方向，不是门槛
   - 交人裁决：连续 2 轮全页「持平/更差」→ 报告用户做 taste 裁决（实测：裁决撤 1 个元素解锁单页 +1.1，为全程最大涨幅——polish 修不掉的，往往是人才能拍的决策）
   - 状态抽查（收口前，一轮即止）：票内涉及空态/错误态的，各截一张图单独送评——只判「是否达到主界面同一 craft 水平」，不进循环
   - 像素 diff 哨兵（可选，确定性）：本轮 vs 上轮截图 diff，改动区与修复意图对账，预期外的新变化先自查再送评
   - 豁免清单：循环开始时一次性导出进 prompt 固定段，三类来源与规则见 [references/critic-prompt.md](references/critic-prompt.md)；不逐轮打补丁（实测补丁无界增长且只治症状）
   - 停止线不写进 critic 的 prompt（保持其客观）；角色分离：critic 用强档模型（只做判断，贵得起），实现用便宜档

## Phase 3: Deliver——减法与收尾

AI 只会加不会减，克制本身就是高级感。打磨阶段主要工作是删：

1. **砍元素**：多余 label/badge/空容器、无动机装饰、比原生组件丑的自绘控件（对照平台规范）。
2. **AI Tells Audit**：跑 `npx impeccable detect`（62 条确定性 slop 检测，零安装；其中 3 条为文案类规则，归第 3 步 Copy Audit 处理）；检测项不够再按本项目 grep 自写。不要在第一轮生成时就全面禁用套路（模型会瘫痪），在 polish 阶段一次清完。
3. **Copy Audit（文案轴：lint → 真值）**：两层边界——营销层归本审计，产品 UI 微文案归 frontend-design 写作节（两层规则相反，不混用）。lint 规则、中文词表（本仓库单点）、真值流程与误伤裁决见 [references/copy-audit.md](references/copy-audit.md)；headline/副标/CTA 终稿人拍板：更短、更口语、零营销腔。

## 收口

- 记录留档：seed string、4 张方向卡、每轮 critic 绝对分与 A/B 判决序列、豁免清单快照、Copy Audit 词表命中与裁决、最终截图，供下次 retro 提炼个人 taste 偏好
- 3D 视觉向的深度循环可选装 achimala/dream-loop（需 Blender MCP）；常规 UI 用本流程即可

方法论来源：Anshu Chimala《How to turn your AI into a world-class designer》（Lenny's Newsletter, 2026-09-01，[付费原文](https://www.lennysnewsletter.com/p/how-to-turn-your-ai-into-a-world)／[免费镜像](https://github.com/CodingRookie98/cr-agent-arsenal/blob/master/docs/reference/articles/how-to-turn-your-ai-into-a-world-class-designer.md)）。其中「一次出 4 个方向」为本地工程化裁量（源出原文「先跑 1-2 轮看收敛性再放开，防烧 token」）；锚定对比评审、豁免清单、复合退出条件为 2026-11-18 首次全流程实测后的修订——五轮绝对分 7.2→8.0→7.5→7.6→8.3 呈锯齿，冻结页裁判间方差 ±0.5~1.2 证明裸绝对分跨裁判不可比，实测数据与四机制分析见 [docs/evaluation-2026-11-18.md](docs/evaluation-2026-11-18.md)；Copy Audit 文案轴（lint/中文词表/人填真值）为 2026-11-18 调研驱动的本地裁量——源出原文 T8「文案手写」但做了细化，词表与阈值行为经 impeccable 源码复核；craft baseline（方向卡 + token 契约 + 检查单，[references/craft-baseline.md](references/craft-baseline.md)）为 2026-11-19 实测驱动的本地裁量——token two-pass 吸收自 frontend-design skill（claude-plugins-official，Anthropic）、craft 密度与动效纪律吸收自 emil-design-eng（Emil Kowalski），字段与数值门槛为本仓库定义；双视口截图协议与 critic 问题六维组织同为 2026-11-19 实测驱动修订（critic 问题以间距/字阶/色彩层次等 craft 微观细节为主、原截图协议仅覆盖桌面态），与 craft baseline 一并待下次 UI 票对照实测；渐进披露拆分（SKILL.md 编排骨架 + references/ 载荷文件）为 2026-11-19 公开仓工程裁量；依赖降级说明（未安装相关 skill 时的兜底路径）为公开仓工程裁量；其余流程均为原文技巧（image-gen 闭环的 skill 化成品另见 [achimala/dream-loop](https://github.com/achimala/dream-loop)）。
