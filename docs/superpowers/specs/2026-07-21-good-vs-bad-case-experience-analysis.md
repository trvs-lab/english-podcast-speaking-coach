# Good-case vs Bad-case 体验差异分析

日期：2026-07-21  
依据材料：

- `history/bad-case/1.md` — 仅用 `english-podcast-speaking-coach`（englishpod_0013）
- `history/bad-case/2.md` — 仅用 `english-podcast-speaking-coach`（englishpod_0025）
- `history/good-case/1.md` — `english-podcast-speaking-coach` + `learning-coach`（englishpod_0025）
- `skills/english-podcast-speaking-coach/`
- `skills/learning-coach/`

最强对照：同一课 `englishpod_0025` 下的 `bad-case/2` vs `good-case/1`。

---

## 1. 结论

体验变好，**不是因为多读了一个 skill 文件本身**，而是因为 `learning-coach` 改写了教练的**决策优先级**：

| 维度 | 仅 speaking-coach（体感差） | 叠加 learning-coach（体感好） |
| --- | --- | --- |
| 成功标准 | 过完课文对话、覆盖 10–14 个目标 | 把表达变成日常能用的工具 |
| 单题负荷 | 一次要求产出多信息整段 | 一次只练一个可复用块 |
| 近迁移 | 职场换皮（仍难、动机弱） | 默认落到生活场景 |
| 纠错后推进 | 修一次 → 下一句课文 | 同一缺口追到底，再说稳再前进 |
| 开场感 | 执行流程 / 合规上课 | 先锚定目标，再小步检索 |

`english-podcast-speaking-coach` 已经内置了 retrieval、target hiding、分层提示、近迁移——与 `learning-coach` **理念大半重合**。单独使用时体验差，主要不是缺学习科学，而是：**规则过重、默认服务「覆盖课文」、缺少硬性的难度旋钮**。

---

## 2. 体验变好的五个关键因素

### 2.1 目标锚点：从「过完这课」到「日常能用」

**bad-case/2 开场：**

> 这课是在公司会议里讨论禽流感……你会练到「切换会议议题、提出应急方案……」这些职场口语块。今天大概会抓 10-14 个核心表达。

**good-case 开场：**

> 这节课不是为了背「禽流感会议」这段对话，而是把里面的表达变成你日常能用的口语工具。

对应 `learning-coach` 原则：**Anchor / top-down**——先钉住「为什么练、练完能干什么」，材料只是原料。

用户在 good-case 里还显式说出目标：「我想要学会，并在日常生活中灵活表达」。speaking-coach 单独跑时，默认忠实于课文场景与 coverage，容易变成「把对话练完」。

### 2.2 认知负荷：一次一个块，而不是一次一整段

**bad-case/2 第一题**要求一次产出：

- 切换议题
- 媒体报道背景
- 公司缺少应急方案

→ 一整段会议开场。学习者直接崩盘（拼写、结构、多块同时丢）。

**good-case 第一题**只有一句：

> 「好了，我们进入下一个议题。」

对应 `learning-coach`：**one concept at a time**、**desirable difficulty**（难度卡在勉强够得着，而不是同时扛三个新块）。

speaking-coach 虽有 cognitive load 提示，但默认仍偏向「跟课文走、覆盖 10–14 个目标」，agent 容易开大题。

### 2.3 迁移方向：生活场景优先，而不是职场换皮

| | bad-case 近迁移 | good-case 近迁移 |
| --- | --- | --- |
| 方向 | 数据安全会议、服务器宕机（仍是职场） | 周末备用方案、聚会食物、搬家预算、交通延误 |
| 体感 | 换场景仍难，像加练 | 同一结构换到生活，有「我会用了」的 click |

用户目标是 **flexible daily expression**。bad-case 的近迁移仍服务「掌握这课会议语」——难度高、动机弱。

### 2.4 查漏是递归的，不是流水线往下推

good-case 里，复杂结构（如 `figure out what kind of impact ... might have`）第一次生活迁移失败后，会**再给一次更近的迁移**，直到说稳再往前。

这是 `learning-coach` 的 **recursive gap-filling**：一个缺口追到底。

bad-case / speaking-coach 默认更像流水线：纠错 → 近迁移一次 → 下一句课文。

### 2.5 启动与节奏：少仪式、多校准

speaking-coach 单独用时，agent 要读大量英文规则、workspace、writeback，开场偏「执行流程」。

叠上 learning-coach 后，更像：先校准目标与水平 → 小步检索 → 纠错 → 再检索。

**感觉不那么生硬**，主要来自：更短的 prompt、更清楚的「为什么练这个」、更频繁的小胜利。

---

## 3. 对现有 speaking-coach 的诊断

### 3.1 它其实已经有的（不必推倒重来）

- 先尝试再揭示（active recall / target hiding）
- 分层提示：keyword → frame → complete
- 问题触发纠错 + 近迁移（不照抄）
- 课末分区总结（已说出 / 下次再练 / 听过未练 / 词汇）

这些与 learning-coach 的检索练习、理想难度、纠错闭环高度一致。

### 3.2 真正的问题

1. **规则过重、过英文、过流程化**  
   `INTERACTION-RULES.md`  alone 500+ 行；加上 state / writeback / recovery 等，规则总量约 1600+ 行。agent 优先「合规执行」，难做难度旋钮。

2. **把「覆盖课文」写成默认成功标准**  
   日常迁移是附加步骤，不是主线。

3. **状态机与写回系统过重**  
   PB / RQ / RB、ledger、recovery 维护成本高；对学习者体感帮助有限，却占掉大量 token 与注意力。

4. **缺少一等公民的难度校准**  
   何时拆题、何时二次迁移、何时降级为 exposure-only，写得散、不够硬。

---

## 4. 重写方向建议（分析延伸，非已定稿设计）

### 4.1 三条路线

| 路线 | 做法 | 优点 | 代价 |
| --- | --- | --- | --- |
| **A. 合并重写（推荐）** | 把 learning-coach 行为原则写入新的中文 speaking-coach，不依赖第二个 skill | 单一入口、体验稳定、可维护 | 要认真砍掉旧规则大半流程细节 |
| **B. 薄适配层** | speaking-coach 只做材料→场景，教学节奏委托 learning-coach | 改动小 | 双 skill 触发不稳，行为难复现（good-case 即偶发组合） |
| **C. 两层中文** | 主 `SKILL.md` ~80–100 行硬原则 + 精简课堂脚本；workspace 压到最小 | 渐进披露，易维护 | 需单独定状态边界 |

推荐：**A + C**——合并原则进 speaking-coach，用中文短入口 + 按需 reference。

### 4.2 新 skill 建议写死的行为

1. **先问/推断目标**：日常能用 / 职场会议 / 考试——决定迁移场景池。
2. **默认：一次只练一个可复用块**；多信息意图必须拆题。
3. **近迁移默认去生活场景**（除非用户明确只要职场）。
4. **缺口不放过**：同一结构迁移失败 → 再近一层，再说稳再前进。
5. **理想难度**：卡住超过一轮就给 keyword hint；不干瞪眼，也不一次摊整句。
6. **状态极简优先**：课末三栏即可；砍掉 ledger / 多 schema，除非明确需要可机械恢复。

### 4.3 待确认的产品边界

学习状态工作区保留到什么程度（决定重写复杂度）：

- **A.** 极简：课末总结 + 一个 `review.md`（最好维护）
- **B.** 中等：保留 phrase-bank + review-queue，去掉 writeback ledger / recovery
- **C.** 完整：继续可从 lesson 文件机械恢复状态（接近现状）

---

## 5. 材料对照速查

### 5.1 同一课（0025）开场对比

| | bad-case/2 | good-case/1 |
| --- | --- | --- |
| 目标叙事 | 职场口语块、10–14 个表达 | 日常口语工具，非背会议稿 |
| 第一题 | 多从句整段开场 | 单句「进入下一个议题」 |
| 近迁移默认 | 同域职场换皮 | 生活场景 |
| 失败后 | 倾向进下一句 | 同结构再近一层迁移 |

### 5.2 Skill 体量（分析时快照）

| 文件 / 目录 | 约行数 |
| --- | ---: |
| speaking-coach `INTERACTION-RULES.md` | 528 |
| speaking-coach 全部规则 md | ~1600 |
| learning-coach `SKILL.md` + references | ~590 |
| bad-case/1 | 430 |
| bad-case/2 | 147 |
| good-case/1 | 679 |

---

## 6. 与后续工作的关系

- 本文档只固定**体验差异原因**与**重写建议方向**，不替代正式 redesign spec。
- 仓库中已有相关实现计划草案：`docs/superpowers/plans/2026-07-21-learning-centered-speaking-coach-rewrite.md`。定稿状态边界（§4.3）后，应以正式 design spec 为准对齐该 plan。
- 历史样本 `history/bad-case/`、`history/good-case/` 应保留为回归对照，重写时不要改写其内容。
