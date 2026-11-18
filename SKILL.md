---
name: ui-taste-loop
description: AI UI 设计三阶段工作流（源自 Anshu Chimala 方法论）。Discover 用 seed string 注入外部随机性发散 4 个方向，Build+Critique 用 fresh-context critic 只看截图做锚定对比评审（新旧两版 A/B 判决为主信号、绝对分仅记趋势），迭代到复合退出条件，Deliver 做减法、AI Tells 审计与文案轴 Copy Audit（lint + 人填真值）。面向用户的界面生成或视觉改版任务（landing page、面板页、组件视觉重做）先走本流程；纯逻辑实现、无视觉产物的任务不适用。
---

# UI Taste Loop

核心判断：LLM 默认输出「取悦平均审美的安全牌」（紫渐变、bento 卡片、营销黑话）。模型不是瓶颈，工作流才是——多样性从外部注入，评审用独立 fresh context，交付靠减法。

## 与已装 skills 的分工

| skill | 阶段 | 职能 |
|---|---|---|
| ui-taste-loop（本 skill） | 全流程编排 | 发散 → 评审 → 减法循环骨架 |
| frontend-design | 生成端 | 实现时的风格约束与反模式（含产品 UI 微文案写作节：active voice、CTA 说清动作、术语一致、错误不道歉） |
| emil-design-eng | 细节 | 动效频率、easing、组件反模式 |
| web-design-guidelines | 合规下限 | a11y/性能审计（review my UI）——管「对不对」 |
| impeccable（npx，未安装） | 审美下限（Phase 3 出场） | 确定性 slop/AI Tells lint——管「俗不俗」，不管「对不对」 |

不冲突：本 skill 管流程，其余在各自阶段被调用。

## 票内时序：与 /tdd、/code-review 的编排

本 skill 与 /tdd（行为轴）、/code-review（代码轴）是同一张 UI 票内并列的三条验证轴，不互相替代（示例为 Matt Pocock skills 语境，无此环境时映射到等价的测试先行/代码评审流程）：

1. **行为先行**：/tdd 红绿环把功能做绿——跑不起来的 UI 没法评审美
2. **视觉迭代**：行为绿 → 截图 → critic 锚定评审循环（Phase 2）
3. **减法收尾**：AI Tells Audit + Copy Audit 文案轴（lint + 人填真值）（Phase 3）
4. **代码轴收口**：/code-review 双轴照常跑；critique 轮次中的 CSS/结构改动若触碰行为，回到 /tdd 补红绿

**完成判定：一张 UI 票完成 = 测试全绿 + critic 评审通过（Phase 2 复合退出条件）+ Phase 3 收口（AI Tells/Copy Audit 命中清零或经裁决留档、真值槽位已填）+ code-review 通过，缺一不可。**

## Phase 1: Discover——发散，禁止直接收敛

**载体两种**：小改直接在票内跑完整三阶段；大方向探索走 /prototype detour——在 throwaway 原型里**只跑本 Phase**（seed 出 4 方向供人挑选），选定后经 /handoff 折回主线，正式票内再跑 Phase 2/3。critic 9/10 是对正式实现的要求，throwaway 原型不跑完整评审循环。

1. **Seed string 注入随机性**（让模型「be random」无效，多样性必须来自模型外部）：
   - 用 shell 生成随机串：`LC_ALL=C tr -dc 'A-Za-z0-9' </dev/urandom | head -c 48`
   - 让 agent 从字符串的子模式/重复字符/数字提取设计方向（配色基调、布局隐喻、字体气质），一次产出 **4 个方向**
   - 用户挑选；生成的页面不得暴露字符串本身
2. **Ambitious brief**：选定方向后，brief 要敢想——引用灵感源（某个游戏、某次室内设计趋势、某个艺术装置），而非「做一个好看的后台页面」。「这不可能行吧」的感觉恰是方向正确的信号。
3. **可选：生图做目标图/moodboard**（火山 Seedream，已实测可用）。URL 24 小时失效，**必须当场下载**：

```bash
python3 - <<'EOF'
import json, os, urllib.request
# 先替换两个占位符
PROMPT = '<目标图/moodboard 的描述>'
OUT_PATH = '<保存路径>.png'
key = os.environ.get('ARK_API_KEY')
if not key:  # ZCode 会话内 env 未生效时从配置读
    cfg = json.load(open(os.path.expanduser('~/.zcode/v2/config.json')))
    key = next(p['options']['apiKey'] for p in cfg['provider'].values()
               if p.get('name') == '火山引擎' and p.get('options', {}).get('apiKey'))
req = urllib.request.Request(
    'https://ark.cn-beijing.volces.com/api/v3/images/generations',
    data=json.dumps({'model': 'doubao-seedream-5-5-261028',
                     'prompt': PROMPT, 'size': '1024x1024'}).encode(),
    headers={'Authorization': f'Bearer {key}', 'Content-Type': 'application/json'})
resp = json.load(urllib.request.urlopen(req, timeout=180))
open(OUT_PATH, 'wb').write(urllib.request.urlopen(resp['data'][0]['url'], timeout=120).read())
print('saved', OUT_PATH)
EOF
```

## Phase 2: Build & Critique——锚定评审闭环

1. **Build**：选定方向后实现，frontend-design 约束下进行；moodboard/参考图可喂给实现 agent 当质量锚点（参考，不是抄）。
2. **Critique**：写代码的 agent 不能自评设计（放不下自己的决策，无法 zoom out）。另起 **fresh-context subagent**，只喂截图、不给代码/实现/历史。R1 喂本轮 4 张；R2 起喂**上一轮+本轮两组做锚定对比**——裁判每轮更换，绝对分跨裁判不可比（实测裁判间方差 ±0.5~1.2，淹没每轮 +0.3 的真实改进），可比信号只有同一裁判眼里的新旧两版。截图用浏览器截取或让实现 agent 先渲染。
3. **Critic prompt 模板**：

```
你是顶级设计工作室的设计总监，评审一个产品的新旧两版界面截图（各按产品流程排列）。
- 截图经 Read 已在你的上下文中，直接基于像素评审；不要再调用图像转述/分析工具
- 先独立评新版，再做两版对比：
  1) 逐页 0-6 条问题（可为 0，无重大缺陷要明说），每条标类别、指到具体位置、给可执行修改：
     [缺陷] 视觉/实现层面，可直接修 ／ [裁决] 结构/产品决策，人定 ／ [偏好] 品味取向，可不采
  2) 逐页判决：新版更好/持平/更差 + 一句话理由；单独列出新版引入的问题
- 兼顾整体构图与微观细节（间距、字阶、对齐、色彩层次）
- 识别并惩罚「用力过猛/明显 AI 味」：紫青弥散光晕、渐变描边、万物容器化、
  全面药丸圆角、无动机的漂浮小标签、模板化左文右图 hero、营销腔文案
  （buzzword/空形容词——具体性应来自数字与实物）、对仗警句反复出现
- 大胆有立场，不选安全牌；想象顶级工作室会怎么执行这个美学，指出与它的最大差距
- 豁免清单（不要再提、不据此扣分）：<循环开始时一次性导出，见下>
- 末行输出四页等权平均分 `X.X/10`（固定等权，仅作跨轮趋势参考）
```

4. **豁免清单**：循环开始时一次性写进 prompt 固定段，三类来源——契约禁项（设计契约明令禁止的）、已裁决保留项（人拍板保留的）、静态盲区（hover/sticky 等已实现但截图看不到的）。critic 重提即忽略；被拒的新建议经人裁决后补进清单。不逐轮往 prompt 里打补丁（实测补丁无界增长且只治症状）。
5. **循环纪律**：
   - 驱动信号 = 判决 + 新增缺陷数；绝对分只记趋势，不单独触发任何动作
   - 收口（其一）：critic 明说「无新增缺陷」且多数页判「更好」；或绝对分进入 8.5±0.5 带且当轮无新增缺陷。9/10 是方向，不是门槛
   - 交人裁决：连续 2 轮全页「持平/更差」→ 报告用户做 taste 裁决（实测：裁决撤 1 个元素解锁单页 +1.1，为全程最大涨幅——polish 修不掉的，往往是人才能拍的决策）
   - 像素 diff 哨兵（可选，确定性）：本轮 vs 上轮截图 diff，改动区与修复意图对账，预期外的新变化先自查再送评
   - 停止线不写进 critic 的 prompt（保持其客观）；角色分离：critic 用强档模型（只做判断，贵得起），实现用便宜档

## Phase 3: Deliver——减法与收尾

AI 只会加不会减，克制本身就是高级感。打磨阶段主要工作是删：

1. **砍元素**：多余 label/badge/空容器、无动机装饰、比原生组件丑的自绘控件（对照平台规范）。
2. **AI Tells Audit**：跑 `npx impeccable detect`（62 条确定性 slop 检测，零安装；其中 3 条为文案类规则，归第 3 步 Copy Audit 处理）；检测项不够再按本项目 grep 自写。不要在第一轮生成时就全面禁用套路（模型会瘫痪），在 polish 阶段一次清完。
3. **Copy Audit（文案轴：lint → 真值）**：只管 landing/营销层与检测流程；产品 UI 微文案（按钮/错误/空态/术语一致）归 frontend-design 写作节——两层规则相反（营销层允许第一人称与叙事，UI 层禁 "we"），不混用：
   - **lint（确定性，与 AI Tells Audit 同趟跑）**：impeccable 已含 3 条文案规则——marketing-buzzword（29 短语，**单次命中即报**）、aphoristic-cadence（"X. No Y." 警句式 ≥3 次才报）、em-dash-overuse（≥8 个+密度门槛），英文项目零新增工具。词表误伤（品牌名恰含词表词——顶级站也会犯，Basecamp 自家文案就用过 industry-leading）经人裁决保留并记入豁免清单快照（收口留档，下次不重复裁决）。中文项目自建 grep 词表（现有工具与词表全英文，中文是空白）：

     ```bash
     grep -rnoE '赋能|一站式|全方位|行业领先|重新定义|颠覆|未来已来|极致体验|卓越体验|匠心打造|精心打造|无缝体验|无缝衔接|降本增效|助您|让您轻松' \
       --include='*.tsx' --include='*.jsx' --include='*.html' --include='*.vue' .
     ```

     初版词表随实测迭代；单次命中即报，误伤走人裁决留档。句式 tell（排比三连、"不是…而是…"、"不止于…更是…"）不进 grep——由 Phase 2 critic 循环按密度判，Phase 3 不重跑 critic
   - **真值（人）**：AI 文案一律当 Lorem ipsum，但人不是重写全部，是**填真值**——真实数字/客户名/机制名只有人有（顶级 landing 的具体性全靠 50,000/$1.9T/99.9999% 式实物，空形容词是 AI tell）；headline/副标/CTA 终稿人拍板：更短、更口语、零营销腔

## 收口

- 记录留档：seed string、4 个方向、每轮 critic 绝对分与 A/B 判决序列、豁免清单快照、Copy Audit 词表命中与裁决、最终截图，供下次 retro 提炼个人 taste 偏好
- 3D 视觉向的深度循环可选装 achimala/dream-loop（需 Blender MCP）；常规 UI 用本流程即可

方法论来源：Anshu Chimala《How to turn your AI into a world-class designer》（Lenny's Newsletter, 2026-09-01，[付费原文](https://www.lennysnewsletter.com/p/how-to-turn-your-ai-into-a-world)／[免费镜像](https://github.com/CodingRookie98/cr-agent-arsenal/blob/master/docs/reference/articles/how-to-turn-your-ai-into-a-world-class-designer.md)）。其中「一次出 4 个方向」为本地工程化裁量（源出原文「先跑 1-2 轮看收敛性再放开，防烧 token」）；锚定对比评审、豁免清单、复合退出条件为 2026-11-18 首次全流程实测后的修订——五轮绝对分 7.2→8.0→7.5→7.6→8.3 呈锯齿，冻结页裁判间方差 ±0.5~1.2 证明裸绝对分跨裁判不可比，实测数据与四机制分析见 [docs/evaluation-2026-11-18.md](docs/evaluation-2026-11-18.md)；Copy Audit 文案轴（lint/中文词表/人填真值）为 2026-11-18 调研驱动的本地裁量——源出原文 T8「文案手写」但做了细化，词表与阈值行为经 impeccable 源码复核；其余流程均为原文技巧（image-gen 闭环的 skill 化成品另见 [achimala/dream-loop](https://github.com/achimala/dream-loop)）。
