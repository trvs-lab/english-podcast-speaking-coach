# English Podcast Speaking Coach 中拆设计

## 背景

`english-podcast-speaking-coach` 当前只有一个主体文件：

- `skills/english-podcast-speaking-coach/SKILL.md`

该文件约 819 行，同时承担以下职责：

- skill 入口、触发范围和核心原则；
- workspace 发现、初始化和启动读取；
- 课中口语教学流程；
- 用户可见语气规则；
- active recall、target hiding、correction、near-transfer 等互动细则；
- durable state schema；
- lesson-end writeback 事务；
- replay、conflict、privacy、migration 等状态治理规则；
- common mistakes。

这种结构的优点是规则集中，触发后不容易漏读。主要问题是职责过多：普通口语课也会把大量低频 schema、writeback 和 recovery 规则带入上下文，容易让用户可见回复变得流程化、行政化，也让后续维护更容易误伤实时教学规则。

项目里已有 `2026-06-12-english-podcast-speaking-coach-light-split-design.md`，建议采用 `SKILL.md` 加 4 个 reference files 的轻拆方案。该方向仍然成立，但当前主文件后来又加入了用户可见语气优化，热路径本身已经值得单独成文件。因此，本设计在轻拆方案基础上升级为 6 文件中拆。

## 目标

- 降低 `SKILL.md` 的默认体积，让它成为清晰的入口和路由文件。
- 把课中互动规则与状态治理规则分离，减少低频治理语言对普通练习语气的影响。
- 保持现有教学语义不变，包括 active recall、target hiding、mastery lifecycle、writeback、replay、privacy 和 migration。
- 让每个文件只有一个主要职责，并有明确读取条件。
- 为后续修改互动语气、状态 schema 或 writeback 格式提供稳定边界。

## 非目标

- 不重新设计口语课流程。
- 不改变 mastery lifecycle。
- 不放宽 target hiding。
- 不改变 lesson-end writeback 的事务顺序。
- 不引入脚本、自动化解析器或新运行时。
- 不压缩掉实时教学质量所需的细则。
- 不一次性拆成过多小文件。

## 方案比较

### 方案 A：沿用 4 文件轻拆

结构：

```text
skills/english-podcast-speaking-coach/
  SKILL.md
  WORKSPACE-FORMAT.md
  STATE-SCHEMAS.md
  WRITEBACK-FORMAT.md
  RECOVERY-RULES.md
```

优点：

- 改动小。
- 与既有轻拆设计一致。
- 低频状态规则可以从主文件移出。

缺点：

- `SKILL.md` 仍需承载大量课中互动细则。
- 主文件预计仍会保留 300 到 400 行。
- 后续用户可见语气规则继续与入口规则混在一起。

### 方案 B：6 文件中拆

结构：

```text
skills/english-podcast-speaking-coach/
  SKILL.md
  WORKSPACE-FORMAT.md
  INTERACTION-RULES.md
  STATE-SCHEMAS.md
  WRITEBACK-FORMAT.md
  RECOVERY-RULES.md
```

优点：

- `SKILL.md` 可以降为入口、原则摘要和读取路由。
- 课中互动规则集中在 `INTERACTION-RULES.md`，不受 writeback 和 recovery 细节干扰。
- durable state、writeback 和 recovery 各自独立，维护边界清楚。
- 文件数仍然可控，读取路径不碎。

缺点：

- 比轻拆方案多一个热路径文件。
- live coaching session 必须明确读取 `INTERACTION-RULES.md`，否则口语课质量会下降。

### 方案 C：极细拆分

结构示例：

```text
skills/english-podcast-speaking-coach/
  SKILL.md
  STARTUP.md
  LESSON-FLOW.md
  VOICE.md
  TARGET-HIDING.md
  STATE-SCHEMAS.md
  WRITEBACK-FORMAT.md
  RECOVERY-RULES.md
  MISTAKES.md
```

优点：

- 每个主题非常纯。
- 单个文件很短。

缺点：

- 读取路径太碎，agent 更容易漏读关键规则。
- target hiding、correction、help request 和 near-transfer 之间强相关，拆开后上下文成本反而上升。
- 当前项目规模还不需要这种粒度。

## 决策

采用方案 B：6 文件中拆。

`SKILL.md` 作为 runtime entrypoint，保留不可破坏的原则、最短会话流程和 reference file 路由。`INTERACTION-RULES.md` 作为 live coaching 的热路径文件，开始任何 lesson、review、free retelling 或 free-expression session 前必须读取。其他 reference files 按需读取。

## 推荐结构

```text
skills/english-podcast-speaking-coach/
  SKILL.md
  WORKSPACE-FORMAT.md
  INTERACTION-RULES.md
  STATE-SCHEMAS.md
  WRITEBACK-FORMAT.md
  RECOVERY-RULES.md
```

预期体积：

- `SKILL.md`：约 180 到 260 行。
- `INTERACTION-RULES.md`：约 300 到 380 行。
- `WORKSPACE-FORMAT.md`：约 100 到 140 行。
- `STATE-SCHEMAS.md`：约 140 到 190 行。
- `WRITEBACK-FORMAT.md`：约 90 到 130 行。
- `RECOVERY-RULES.md`：约 80 到 120 行。

这些数字是迁移目标，不是硬性指标。优先保证边界清楚和语义不变。

## 文件职责

### `SKILL.md`

保留入口和摘要，不承载完整格式示例。

内容：

- frontmatter；
- `Overview`；
- `Core Principles`；
- `Reference Files`；
- `Session Startup` 简版；
- `Lesson Flow` 简版；
- `Durable State Summary`；
- `Lesson-End Summary` 简版；
- `Common Mistakes` 精简版。

必须保留的硬规则：

- active recall comes before answer reveal；
- target chunks 在 learner attempt 前不得泄露；
- 只有学习者主动产出或成功 near-transfer repair 可以支持 durable mastery；
- copied correction、answer reveal、keyword hint、sentence frame 和 English menu 都不能直接升级为 `active`；
- completed lesson files 是 source of truth，`state/*.md` 和 `phrase-bank/*.md` 是 materialized views；
- pronunciation coaching out of scope；
- 写入 durable state 前需要匿名化敏感细节。

`SKILL.md` 需要明确：只要进入实际口语练习，就必须先读 `INTERACTION-RULES.md`。

### `WORKSPACE-FORMAT.md`

存放 workspace 与启动读取规则。

迁入内容：

- `Workspace Discovery`；
- `Workspace Files`；
- `Default Mission`；
- `Startup Reading Set`；
- workspace 文件职责说明。

读取条件：

- 发现或创建 `english-coach/` workspace；
- 修复缺失、空白或损坏的 `MISSION.md`；
- 判断 startup read set；
- 用户显式提供项目目录或课程目录；
- workspace schema version 相关问题。

### `INTERACTION-RULES.md`

存放 live coaching 热路径。任何实际口语练习前必须读取。

迁入内容：

- `Lesson Setup`；
- `Guided Preview`；
- `Learner-Facing Voice`；
- `Active-Recall Blocks`；
- `Learner-Facing Templates`；
- `Help Requests`；
- `Comprehensible Input Guard`；
- `Prompt Leakage Guard`；
- `Phrase Substitution`；
- `Near-Transfer Vocabulary`；
- `Free Retelling`；
- `Free Expression`；
- `Coverage Tracking`；
- `Speed Round`；
- `Correction`；
- `Lesson-End Expression Areas`；
- 与用户可见互动直接相关的 common mistakes。

边界：

- 保留课中互动和用户可见表达细则。
- 不放完整 durable state schema 示例。
- 不放 writeback summary 完整格式。
- 不放 replay、conflict、migration 细节。

### `STATE-SCHEMAS.md`

存放 durable state item 的格式、生命周期和去重规则。

迁入内容：

- `Durable State Schemas`；
- `Mastery Lifecycle` 完整定义；
- `ID Allocation`；
- `Evidence Identity`；
- `Review Queue Item`；
- `Phrase Bank Item`；
- `Repair Bank Item`；
- active、dormant、retired 分区；
- active review 和 active repair 的容量规则。

读取条件：

- 创建、更新、去重或合并 review queue item；
- 创建、更新或重建 phrase bank item；
- 创建、更新或合并 repair bank item；
- 分配 `PB-*`、`RQ-*` 或 `RB-*` ID；
- 判断 mastery upgrade 或 downgrade；
- 重建 `state/phrase-bank-index.md`。

### `WRITEBACK-FORMAT.md`

存放课末写回事务和可重放 summary 格式。

迁入内容：

- `Lesson-End Writeback`；
- `Writeback Summary`；
- `Current Summary Shape`；
- `Learning Records`。

读取条件：

- 写 completed lesson file；
- 准备或校验 `Writeback Summary`；
- 应用 lesson-end state updates；
- 更新 `state/CURRENT.md`；
- 创建 `learning-records/*.md`；
- 追加 `state/writeback-ledger.md`。

### `RECOVERY-RULES.md`

存放恢复、冲突处理和治理规则。

迁入内容：

- `Replay And Recovery`；
- `Conflict Handling`；
- downgrade rules；
- `Privacy And Anonymization`；
- `Migration Rules`。

读取条件：

- replay completed lesson writebacks；
- 处理 ledger missing、stale 或 suspect；
- 状态文件缺失、损坏或互相矛盾；
- active review 或 repair 内容超出启动预算；
- 用户纠正已存状态；
- 处理 privacy 或 anonymization；
- migration 旧版 workspace。

## 读取策略

`SKILL.md` 中新增或改写 `Reference Files` 小节：

```md
## Reference Files

Read these files by need:

- `INTERACTION-RULES.md`: required before any live lesson, review, free retelling, or free-expression session.
- `WORKSPACE-FORMAT.md`: read during workspace discovery, initialization, default mission restoration, or startup read-set decisions.
- `STATE-SCHEMAS.md`: read when creating, updating, deduplicating, reconciling, or grading durable review queue, phrase bank, or repair bank items.
- `WRITEBACK-FORMAT.md`: read before writing a completed lesson file, preparing a `Writeback Summary`, applying lesson-end state updates, or creating learning records.
- `RECOVERY-RULES.md`: read when replaying writebacks, resolving conflicts, handling malformed state, anonymizing durable examples, or migrating old workspace versions.
```

读取原则：

- 讨论、审阅或编辑 skill 本身时，不创建或修改 `english-coach/` workspace。
- 普通 live coaching session 至少读取 `SKILL.md` 和 `INTERACTION-RULES.md`。
- workspace 初始化或启动状态读取时，再读取 `WORKSPACE-FORMAT.md`。
- durable state 变更前读取 `STATE-SCHEMAS.md`。
- lesson-end 写回前读取 `WRITEBACK-FORMAT.md`。
- replay、conflict、privacy 或 migration 发生时读取 `RECOVERY-RULES.md`。

## 迁移策略

本次拆分只重组文档结构，不改变行为语义。

迁移顺序：

1. 新建 5 个 reference files。
2. 将现有 `SKILL.md` 按 heading 搬迁到对应文件。
3. 压缩 `SKILL.md`，保留入口、摘要和读取路由。
4. 对搬迁内容做轻量去重，只删除明显重复的说明，不重写教学机制。
5. 在 `INTERACTION-RULES.md` 保留所有课中互动关键规则。
6. 在 `SKILL.md` 和对应 reference files 的开头重复关键读取条件。
7. 做文本验证和人工场景验证。

优先搬迁原文。只有在主文件摘要需要压缩时，才做概括性改写。

## 验证标准

文本验证：

- `SKILL.md` 包含 `Reference Files`。
- `SKILL.md` 明确要求 live coaching session 读取 `INTERACTION-RULES.md`。
- 5 个 reference files 都存在。
- `SKILL.md` 不包含完整 schema 示例，如 `### RQ-0007`、`### PB-0012`、`### RB-0004`。
- `SKILL.md` 不包含完整 `Writeback Summary` 示例。
- `INTERACTION-RULES.md` 包含 `Prompt Leakage Guard`、`Active-Recall Blocks`、`Help Requests`、`Correction` 和 `Speed Round`。
- `STATE-SCHEMAS.md` 包含 review queue、phrase bank、repair bank schema。
- `WRITEBACK-FORMAT.md` 包含 `writeback_status: ready` 和 `writeback_complete: true`。
- `RECOVERY-RULES.md` 包含 replay、conflict、privacy 和 migration 规则。
- pronunciation workflow 不被重新引入。

人工场景验证：

- 有新 lesson material 时，仍先 active recall，再 reveal target English。
- 学习者请求提示时，仍按 `keyword hint -> sentence frame -> complete natural expression` 逐步展开。
- near-transfer 不要求复制刚揭示的 correction。
- 基本自然的回答获得短反馈，明显问题获得 problem-triggered correction。
- free retelling 和 free expression 的切换仍清楚。
- Speed round 只测试未真正产出的 core targets、repair targets 和 practiced extension expressions。
- 课末四区仍区分 active phrase bank、review queue、extension expressions 和 lesson vocabulary。
- lesson-end writeback 仍先写 completed lesson file，再 mutation state files。

## 风险与处理

### 风险：live coaching 时漏读 `INTERACTION-RULES.md`

处理：在 `SKILL.md` 的 `Reference Files` 和 `Session Startup` 中同时写明读取条件。`INTERACTION-RULES.md` 开头也声明它是 live coaching 必读文件。

### 风险：主文件过短导致教学质量下降

处理：`SKILL.md` 保留硬规则摘要。所有课中互动细则集中保留在 `INTERACTION-RULES.md`，并作为实际练习必读文件。

### 风险：拆分时改变状态语义

处理：schema、writeback、recovery 规则优先搬迁原文，不在拆分提交里重写 lifecycle 或事务顺序。

### 风险：source-of-truth 关系变模糊

处理：`SKILL.md` 保留 source-of-truth 总原则。`WRITEBACK-FORMAT.md` 说明 completed lesson file 和 `Writeback Summary` 的职责。`STATE-SCHEMAS.md` 说明 materialized state item 的职责。

### 风险：reference file 数量继续膨胀

处理：本轮固定为 5 个 reference files。除非出现新的稳定职责，不再新增文件。

## 后续实施计划

拆分实施应分 4 步完成：

1. 新建 reference files 并搬迁内容。
2. 压缩 `SKILL.md` 为入口和路由。
3. 做文本验证。
4. 做人工场景验证。

拆分完成后，再根据实际使用情况决定是否继续压缩 `INTERACTION-RULES.md`。该优化不属于本设计的第一阶段。
