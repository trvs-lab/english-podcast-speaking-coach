# English Podcast Speaking Coach 轻拆设计

## 背景

`english-podcast-speaking-coach` 已经从单课口语教练扩展为有状态的学习工作区。当前 `SKILL.md` 约 741 行，包含实时口语教学流程、workspace 初始化、状态 schema、lesson-end writeback、replay、conflict、privacy、migration 等内容。

这种单文件结构的优点是规则集中、触发后不容易漏读；缺点是普通 coaching session 也会加载大量低频格式规则，增加上下文负担，并且后续维护 schema 或 writeback 时容易误伤口语教学流程。

Matt Pocock 的 `teach` skill 采用短 `SKILL.md` 加多个格式文件的结构，例如 `MISSION-FORMAT.md`、`RESOURCES-FORMAT.md`、`LEARNING-RECORD-FORMAT.md`。该结构适合作为参考，但不应一比一复制。`english-podcast-speaking-coach` 的实时互动规则比 `teach` 更密集，拆分时必须保留课中核心规则在主文件中。

参考：

- [`teach` 目录](https://github.com/mattpocock/skills/tree/main/skills/productivity/teach)
- [`teach/SKILL.md`](https://raw.githubusercontent.com/mattpocock/skills/main/skills/productivity/teach/SKILL.md)

## 目标

- 降低普通 coaching session 的默认上下文负担。
- 提高状态格式、writeback、recovery 等低频规则的可维护性。
- 保持现有行为语义不变。
- 保留实时教学质量所需的核心规则在 `SKILL.md`。
- 为后续 schema 或 writeback 调整提供清晰文件边界。

## 非目标

- 不重新设计口语课流程。
- 不修改 mastery lifecycle、target hiding、writeback、replay、privacy 或 migration 的语义。
- 不引入脚本或自动化解析。
- 不把 `SKILL.md` 压缩到极短，导致实时教学规则依赖附录文件。
- 不完全复制 `teach` 的结构。

## 推荐结构

采用轻拆结构：

```text
skills/english-podcast-speaking-coach/
  SKILL.md
  WORKSPACE-FORMAT.md
  STATE-SCHEMAS.md
  WRITEBACK-FORMAT.md
  RECOVERY-RULES.md
```

预期效果：

- `SKILL.md` 从约 741 行降到约 250-350 行。
- 普通 coaching session 主要读取 `SKILL.md`。
- 初始化、写回、恢复、迁移等高风险动作按需读取 reference files。

## 文件职责

### `SKILL.md`

保留每次口语教练会话都可能直接影响回复质量的规则：

- `Overview`
- `Reference Files`
- `Session Startup` 简版
- `Lesson Setup`
- `Guided Preview`
- `Active-Recall Blocks`
- `Learner-Facing Templates`
- `Help Requests`
- `Comprehensible Input Guard`
- `Prompt Leakage Guard`
- `Phrase Substitution`
- `Near-Transfer Vocabulary`
- `Free Retelling`
- `Free Expression`
- `Coverage Tracking`
- `Speed Round`
- `Correction`
- `Lesson-End Expression Areas` 简版
- `Common Mistakes`

`SKILL.md` 仍需保留以下摘要规则：

- 启动时发现或创建 `english-coach/` workspace。
- 状态生命周期摘要：`attempted -> needs_review -> repaired -> active -> stable -> retired`。
- 写入 durable state 前必须有学习者产出或 near-transfer repair 的证据。
- 写入 durable state 前必须匿名化敏感细节。
- target chunks 在 learner attempt 前不得泄露。

### `WORKSPACE-FORMAT.md`

存放 workspace 与启动读写相关规则：

- `Workspace Discovery` 详细规则。
- `Workspace Files` 目录结构。
- `Default Mission`。
- `Startup Reading Set`。
- 文件职责说明。

`SKILL.md` 只保留按需读取入口：workspace discovery、初始化、默认文件修复或 startup read-set 问题发生时读取此文件。

### `STATE-SCHEMAS.md`

存放 durable state schema：

- `Durable State Schemas`。
- `Mastery Lifecycle` 完整定义。
- `ID Allocation`。
- `Evidence Identity`。
- `Review Queue Item`。
- `Phrase Bank Item`。
- `Repair Bank Item`。
- active、dormant、retired 分区与容量规则。

`SKILL.md` 只保留生命周期摘要。创建、更新、去重、合并或 reconcile review queue、phrase bank、repair bank 时读取此文件。

### `WRITEBACK-FORMAT.md`

存放 lesson-end 写回格式：

- `Lesson-End Writeback`。
- `Writeback Summary`。
- `Current Summary Shape`。
- `Learning Records`。

`SKILL.md` 保留 lesson-end 顺序提醒：先输出 learner-facing areas，再写 lesson file，再按 writeback 规则更新状态。写入 completed lesson file、应用 lesson-end state updates 或创建 learning records 前读取此文件。

### `RECOVERY-RULES.md`

存放状态恢复和治理规则：

- `Replay And Recovery`。
- `Conflict Handling`。
- downgrade rules。
- `Privacy And Anonymization`。
- `Migration Rules`。

Privacy 放在此文件中，而不是单独建立 `PRIVACY.md`。原因是 privacy 主要约束 durable writeback、recovery 和 state reconciliation；普通课中互动只需要保留一句匿名化提醒。

## 读取策略

`SKILL.md` 中新增 `Reference Files` 小节：

```md
## Reference Files

Read these files only when the current session needs them:

- `WORKSPACE-FORMAT.md`: read during workspace discovery, initialization, default file restoration, or startup read-set questions.
- `STATE-SCHEMAS.md`: read when creating, updating, deduplicating, or reconciling review queue, phrase bank, or repair bank items.
- `WRITEBACK-FORMAT.md`: read before writing a completed lesson file, applying lesson-end state updates, or creating learning records.
- `RECOVERY-RULES.md`: read when replaying writebacks, resolving conflicts, handling malformed state, anonymizing durable examples, or migrating old workspace versions.
```

读取边界：

- 课中互动规则常驻 `SKILL.md`。
- 持久化格式规则放入 reference files。
- 普通 coaching session 不主动读取完整 schema、writeback 或 recovery 规则。
- 进入写回、恢复、迁移、状态冲突处理时，必须读取对应 reference file。

## 迁移策略

本次拆分只做结构重组，不改行为语义。

迁移顺序：

1. 新建四个 reference files。
2. 将现有 `SKILL.md` 的长段内容按职责迁移到对应文件。
3. 精简 `SKILL.md`，保留主流程、摘要规则和按需读取入口。
4. 保留原有 heading 文案的核心含义，避免在拆分时顺手重写规则。
5. 验证主文件与 reference files 共同覆盖原有能力。

## 验证标准

文本验证：

- `SKILL.md` 包含 `Reference Files`。
- 四个 reference files 都存在。
- `SKILL.md` 不再包含完整 schema 示例，例如 `### RQ-0007`、`### PB-0012`、`### RB-0004`。
- `SKILL.md` 保留 `Prompt Leakage Guard`、`Active-Recall Blocks`、`Help Requests`、`Correction`、`Speed Round`。
- `SKILL.md` 保留状态生命周期摘要。
- `SKILL.md` 不包含旧 pronunciation workflow 词，如 `发音提示`、`重读`、`连读`、`跟读`。
- `STATE-SCHEMAS.md` 包含 review queue、phrase bank、repair bank schema。
- `WRITEBACK-FORMAT.md` 包含 `writeback_status: ready` 和 `writeback_complete: true`。
- `RECOVERY-RULES.md` 包含 replay、conflict、privacy、migration 规则。

人工检查：

- 普通 coaching session 只读 `SKILL.md` 时，仍能正确执行 active recall、target hiding、help request、correction、retelling、free expression 和 speed round。
- 初始化 workspace 时，`SKILL.md` 能明确指向 `WORKSPACE-FORMAT.md`。
- 生成 lesson-end writeback 时，`SKILL.md` 能明确指向 `WRITEBACK-FORMAT.md`。
- 处理状态冲突或 replay 时，`SKILL.md` 能明确指向 `RECOVERY-RULES.md`。
- 创建或更新 durable state item 时，`SKILL.md` 能明确指向 `STATE-SCHEMAS.md`。

## 风险与缓解

### 风险：拆分后 agent 漏读 reference file

缓解：在 `SKILL.md` 的 `Reference Files` 小节中写清楚触发条件，并在相关主流程处重复短提示。

### 风险：主文件过短导致口语课质量下降

缓解：课中互动规则不拆出主文件。只拆 workspace、schema、writeback、recovery 等低频格式规则。

### 风险：拆分时改变行为语义

缓解：迁移时优先搬运原文或轻量压缩；不同时重写 lesson flow、mastery lifecycle 或 writeback 逻辑。

### 风险：source-of-truth 关系变模糊

缓解：在 `SKILL.md` 保留一句总原则；在 `STATE-SCHEMAS.md` 和 `WRITEBACK-FORMAT.md` 中分别说明 active phrase-bank item 与 completed lesson writeback 的职责差异。

## 决策

采用轻拆方案。

`SKILL.md` 作为 runtime entrypoint，保留实时教学规则和按需读取入口。四个 reference files 承担低频格式规范和状态治理规则。拆分完成后，行为语义应与当前版本一致，主要收益是降低默认上下文负担和提升维护性。
