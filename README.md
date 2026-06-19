# English Podcast Speaking Coach

English Podcast Speaking Coach 是一个 Codex skill，用于把 ESLPod、EnglishPod、播客转写稿和对话课材料转成主动口语练习。它会围绕真实语境组织角色扮演、主动回忆、近迁移修复、复述和自由表达，并把学习状态写入可查看的 Markdown 工作区。

这个 skill 适合已经有英文听力材料或课程文本、希望把「听懂」推进到「能说出来」的学习流程。它不提供发音训练，重点是表达召回、句块复用、自然改写和跨课复习。

## 使用方式

建议为英语口语练习创建一个独立目录，并把 skill 安装到这个目录中。这样课程文本、学习状态和本地 skill 配置会放在同一个学习空间内，后续继续练习时更容易恢复上下文。

## 首次设置（只做一次）

首次设置只需要做一次。完成后，后续学习只需要重新打开同一个目录并拖入课文。

### 1. 创建学习目录

在 Finder 或文件管理器中新建一个目录，例如 `EnglishPod Coach`。后续课文、练习记录和自动生成的学习状态都会放在这个目录里。

### 2. 用 Codex 打开学习目录

在 Codex 中打开刚才创建的学习目录。首次使用时，目录可以是空的。

### 3. 让 Codex 安装 skill

在 Codex 对话框发送：

```text
请帮我安装 English Podcast Speaking Coach skill（https://github.com/trvs-lab/english-podcast-speaking-coach）。请安装到当前目录，不要全局安装。
可参考的安装命令：npx skills add https://github.com/trvs-lab/english-podcast-speaking-coach --skill english-podcast-speaking-coach
```

如果 Codex 请求确认运行安装命令，确认即可。**安装完成后，记得重启 Codex，让本地 skill 被加载。**

## 每次学习（每节课都这样做）

完成首次设置后，每次学习都从同一个学习目录开始。

### 1. 用 Codex 打开学习目录

打开第一次设置时创建的 `EnglishPod Coach` 目录。

### 2. 拖入课文

把 EnglishPod 文本、ESLPod 文本或其他播客转写稿拖到 Codex 对话框。

### 3. 开始这一课

发送：

```text
带我学习这一课
```

如果课文文件已经放在学习目录里，也可以直接写文件名：

```text
请带我学习 lesson-01.txt 这一课。
```

一次完整练习通常包含：

1. 进入真实语境，说明本节课要练的表达方向。
2. 通过角色扮演触发主动回忆，先让学习者尝试说出英文。
3. 卡住时逐步给提示，从关键词到句型框架，再到完整表达。
4. 对 Chinglish、搭配、时态、冠词、信息顺序等问题做针对性修复。
5. 用近迁移场景重新练习，而不是只照抄答案。
6. 通过复述、自由表达和 Speed round 检查可主动说出的表达。
7. 在课末写回学习状态，区分已说出的表达、下次还要练的表达、听过但未练熟的表达和词汇。

## 学习工作区

第一次开始口语练习时，skill 会在学习目录下创建可见的 `english-coach/` 工作区。后续继续练习时，重新用 Codex 打开同一个学习目录即可复用已有状态。

典型结构如下：

```text
english-coach/
  WORKSPACE.md
  MISSION.md
  NOTES.md
  RESOURCES.md
  GLOSSARY.md
  state/
    CURRENT.md
    phrase-bank-index.md
    review-queue.md
    repair-bank.md
    writeback-ledger.md
  phrase-bank/
  lessons/
  learning-records/
  archives/
```

这些文件用于保存长期学习记忆：

- `state/review-queue.md`：仍需复习或刚修复过的表达。
- `state/repair-bank.md`：反复出现的个人错误模式。
- `phrase-bank/*.md`：已经能主动说出的表达和可复用句型。
- `lessons/*.md`：已完成课程的证据和写回记录，是恢复学习状态的主要依据。
- `RESOURCES.md`：常用课程、播客系列、转写稿目录或学习资料链接。

## 练习原则

- 先尝试表达，再显示目标英文。
- 提示分层给出，避免一开始直接展示答案。
- 只有无提示的主动产出才会被视为稳定掌握证据。
- 课末状态以完成的 lesson 文件为准，状态文件可以从 lesson 记录恢复。
- 长期记录会保留语言模式，避免写入不必要的隐私细节。

## 适用材料

适合使用的材料包括：

- ESLPod、EnglishPod 或类似对话课文本。
- 播客、访谈、课程音频的转写稿。
- 包含真实生活、工作、旅行、观点表达或讲故事场景的英文材料。
- 学习者想复述、改写或转成口语表达的英文文本。

不适合作为主要任务：

- 单词发音纠正。
- 纯语法题讲解。
- 没有可迁移口语表达的长篇资料整理。
- 要求逐字背诵完整 transcript 的练习。

## 仓库结构

```text
skills/
  english-podcast-speaking-coach/
    SKILL.md
    INTERACTION-RULES.md
    WORKSPACE-FORMAT.md
    STATE-SCHEMAS.md
    WRITEBACK-FORMAT.md
    RECOVERY-RULES.md
    REGRESSION-SCENARIOS.md
```

- `SKILL.md` 是运行入口，定义触发条件、核心原则和引用文件读取规则。
- `INTERACTION-RULES.md` 定义实时教学、角色扮演、目标隐藏和反馈方式。
- `WORKSPACE-FORMAT.md` 定义 `english-coach/` 工作区结构。
- `STATE-SCHEMAS.md` 定义复习队列、短语库和修复库的状态格式。
- `WRITEBACK-FORMAT.md` 定义课末写回和学习记录格式。
- `RECOVERY-RULES.md` 定义状态恢复、冲突处理和隐私保护规则。
- `REGRESSION-SCENARIOS.md` 记录行为回归检查场景。
