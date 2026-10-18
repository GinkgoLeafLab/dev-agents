# 新增 UI 设计师与 UI 评审两个角色

- **日期**：2026-10
- **状态**：提议（本仓这一半）
- **触发**：Dean 2026-10-18 的要求——改 UI 之前先由设计师出设计稿，开发严格按稿实现，
  UI reviewer 负责评审所有页面，要求美观、符合现代产品设计美学、文本清晰简洁

## 要解决什么问题

今天的 8 个角色里，**没有一个人对「界面长什么样」负责**：

| 角色 | 它看什么 | 它不看什么 |
|---|---|---|
| `frontend-dev` | 写界面代码 | 没有一份「应该长什么样」可照着写，样子由它当场拍 |
| `code-reviewer` | diff 写对没写对 | 不渲染页面 |
| `qa-tester` | 功能按需求用起来会不会坏 | 明写了「不评审风格」 |

于是界面的样子是写代码那一刻顺手定的，而且**没有一把尺子可以拿来判它对不对**——
「好不好看」在评审里要么不提，要么变成两个模型的口味之争。

## 调研

| 出处 | 它怎么做 | 我们拿什么 |
|---|---|---|
| GitLab 贡献文档「Design and user interface changes」<https://docs.gitlab.com/development/contributing/design/> | 原话：*"When the design is ready, **before** starting its implementation: … Share design specifications in the related issue"*；规格要写清 *user flow and states, design tokens, animations, responsive behavior, … accessibility behavior*；*"Attach the ~UX label to any merge request that has any user facing changes"*；*"Include Before and After screenshots"*；*"Check these aspects both when designing and reviewing UI changes"*（Writing / Patterns / Visual design / States / Responsive / Accessibility，WCAG 2.1 AA） | **先稿后码**、稿要写到状态与令牌、**设计与评审用同一张清单**、按标签标出「改了界面」的 MR、评审看截图 |
| 同一页的偏离处理 | *"Create issues with the `~Deferred UX` label for intentional deviations from the agreed-upon UX requirements"*；范围外的改进另开 issue *"to avoid scope creep"* | 偏离不能静默发生：我们更严一档——**偏离就回去改稿**；存量问题走 issue、不进当前 PR |
| GitLab 手册「Merge Request Reviews」<https://handbook.gitlab.com/handbook/product/ux/product-designer/mr-reviews/> | 带用户可见改动的 MR 由产品设计师评审；**截图看不全悬停态、窄屏与可访问性**。（这一页抓取只拿到前段原文，后段是摘要，**引文以前一条为准**） | 评审必须**自己把页面跑起来**截各状态、各宽度，不只看作者贴的截图 |
| Nielsen 十条可用性启发式 <https://www.nngroup.com/articles/ten-usability-heuristics/> | 原话：*"broad rules of thumb and not specific usability guidelines"*；第 8 条 Aesthetic and Minimalist Design；第 9 条 *"Error messages should be expressed in plain language (no error codes)"* | 「美观、简洁、文案清楚」的判据骨架；但它**自己说了不是可逐条打勾的清单**，所以角色里要落到能指着截图说清的条目 |
| W3C 设计令牌社区组 <https://www.designtokens.org/> | 原话：*"standardizing how design tokens are defined and exchanged"*，让 *"design tools, codebases, and platforms"* 共享同一套视觉语言 | 稿里一律写令牌名、不写原始值；新增令牌要显式说明 |
| OneRedOak/claude-code-workflows <https://github.com/OneRedOak/claude-code-workflows> | 一个现成的 Claude Code「design review」子代理：用 Playwright 打开页面，检查 UI/UX 一致性、可访问性与视觉问题 | 证明「AI 子代理真开浏览器做设计评审」这条路有人走过。**它那份 agent 定义本身拿不到**（`design-review/` 目录抓取被 robots 拒），所以它的分级与视口细节没有参考 |

**三个必答问题**

1. **有没有现成的标准做法？** 有，而且很一致：先出规格再实现、设计与评审共用一张清单、
   改了界面的变更由设计角色评审、评审看渲染结果而非代码。我们没有发明流程，只是把它拆成两个角色
2. **别人踩过什么坑？** 截图看不全状态与窄屏（GitLab 手册自己点的）；没有设计师的组就没人评审（同上）——
   对我们来说是「派不出设计师时流程卡住」，所以开发那边写了两种豁免；
   以及**审美意见没有终点**——这条是我们自己在代码评审上吃过的（「建议」一定会被采纳、改完带出下一条），
   所以 UI 评审也只报能指着截图说清违反哪一条的
3. **有没有更省的路？** 有两条，见下表的前两行。都比这条省，但都做不到「开发严格按稿」——因为根本没有稿

## 备选方案

| 方案 | 怎么做 | 否掉的理由 |
|---|---|---|
| **什么都不做** | 维持现状 | 问题就是「没人对样子负责」，不做它就一直在 |
| 只给 `frontend-dev` 加一段审美纪律 | 一个角色自己画、自己写 | 没有稿，「严格按稿」无从谈起；也没有第二双眼睛 |
| 只加 UI 评审，不加设计师 | 写完再看好不好看 | 把最便宜的拦截点（还没写代码时）丢了；评审没有尺子，只能按口味审 |
| 设计师与评审合成一个角色 | 自己出稿、自己验收 | 自己开自己关——和代码评审不许 resolve 自己的线是同一个理由 |
| 用 Figma 之类的设计工具出稿 | 稿在外部工具里 | 子代理没有稳定的接入，而且引入外部依赖与账号；**可渲染的静态稿 + 截图**用项目自己的样式与令牌，开发能抄、评审能并排对，零新依赖 |
| **两个角色 + 改两份既有角色**（选定） | 见下 | — |

## 选定方案

**本仓这一半（这个 PR）：**

1. 新增 **`ui-designer`**：任何用户可见的界面改动之前出设计稿——规格说明（范围、每个状态、
   布局、令牌、文案定稿、可访问性、验收点）+ 可渲染的静态稿与各目标宽度的截图。
   只产稿，不写产品代码。稿要改只能它改
2. 新增 **`ui-reviewer`**：审三种东西——设计稿本身、实现对不对稿（PR）、产品全量走查。
   自己把页面跑起来截图，只报能指着截图说清违反哪一条判据的；存量问题开 issue。
   发一条 PR 评论 + 判通过时打 UI 放行标签（标签名看项目规则，没配就只发报告）
3. **`frontend-dev`** 加一节：改界面之前先有稿、严格按稿、做不到就停下交回改稿
4. **`code-reviewer`** 加一条必修：改了界面而 PR 正文没指向设计稿、也没写豁免理由。
   它只查稿在不在，不判对不对稿——那是 `ui-reviewer` 的活

两份新角色和既有 8 份同形：frontmatter 带 `skills: - <名>-rules`，开工前「两态」读项目事实，
项目没有时自己说出来。

**模型档位**：两个都是 `opus`。判据仍是「链路两端贵」——设计师是界面这条链的第一道
（它的稿是开发和评审共用的尺子，划错了下游都发现不了），UI 评审是最后一道（它放过的样子就是用户看到的）；
而且两样活都吃对截图的视觉判断。

**消费仓那一半（不在这个 PR 里）**：各仓重接 subtree 时补上 `ui-designer-rules` / `ui-reviewer-rules`
两份 skill（设计稿放哪、令牌表在哪、目标宽度、怎么起页面截图、放行标签叫什么），
并在自己的 `CLAUDE.md` 里写明：哪些改动要稿、谁派这两个角色、UI 评审是不是一把必需检查。
**有界面的仓才需要**；没有界面的仓重接时那两份是「有定义没对象可管」，和今天不派的那几份一样。

### 明确不做什么

- **不在本仓定义放行标签名与必需检查**。那是各消费仓的流程，标签清单在 dev-infra
- **不改 `qa-tester`**。它看功能，UI 评审看样子，两者不合并
- **不给 UI 评审「建议」这一档**，理由见调研第 2 问
- 不在本仓建任何 skill（项目事实不进公开仓）

## 代价

- **一次性**：本仓两份新文件、两份既有文件各加一段；每个有界面的消费仓写两份 skill + 改流程
- **持续**：每个界面改动多两步（出稿、UI 评审），各是一次子代理调用。界面改动的总轮次会变多——
  这是要买的东西本身，但它意味着「派不出设计师就卡住」，所以开发那边留了两种豁免，必须写明
- **它挂了会怎样**：稿写得含糊 → 开发各凭感觉、评审无从判起，**表面上流程全走过了**。
  缓解是稿的那张清单「缺哪项就说缺哪项」，以及 UI 评审审稿那一步
- **钱**：只有子代理的调用量，不引入任何付费服务或新依赖

## 怎么算做完了

本仓这一半：

1. 两份新角色定义存在，`name` 与文件名一致，`skills:` 是 `<name>-rules`
2. `frontend-dev` 与 `code-reviewer` 各有那一段
3. 打出一个新 tag

整件事（跨仓，不在这个 PR 的验收范围里）：某个有界面的消费仓接上之后，
一个界面改动真的走完「出稿 → 审稿 → 按稿实现 → UI 评审」一遍。

## 落地之后

本仓没有常规文档，现状写在角色定义自己里。**本文件从此不再更新。**
