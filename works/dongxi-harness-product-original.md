---
title: "Harness 才是产品：决定 Coding Agent 体验的不是 Model，是它周围的整套 Runtime"
sourceTitle: "Harness 才是产品"
sourceUrl: "https://sotasync.com/reader/2026-06-09-dongxi-nlp-harness-is-the-product/"
sourceAuthor: "马东锡 NLP (@dongxi_nlp)"
sourcePublishedAt: "2026-06"
sourceSiteName: "X / SOTA Sync"
summary: "马东锡 NLP 的 Harness 系列观点收束：聊 coding agent 第一个该问的不是「用哪个 model」，而是「Harness 到底负责什么」。Coding agent = 把 model 放进一套 runtime（查真实 repo、请求 tool、编辑文件、跑检查、记住发生过什么、多轮推进）——这层 runtime 就是 harness，「对 coding agent 来说，harness 本身就是 product」。"
sourceLanguage: "zh-CN"
language: "zh-CN"
translationMethod: "中文原文收录（完整版）"
sourceFigureCount: 0
---

# Harness 才是产品：决定 Coding Agent 体验的不是 Model，是它周围的整套 Runtime

> **原文链接：** https://sotasync.com/reader/2026-06-09-dongxi-nlp-harness-is-the-product/
> 
> **作者：** 马东锡 NLP (@dongxi_nlp)
> 
> **发布时间：** 2026年6月

每次聊 coding agent，话题都会从 model 开始——"用哪个 model？context 多大？写代码能力怎么样？" 这些问题重要，但已经 **不再是我们应该最先问的问题** 。第一个问题应该换成：

> **Harness 到底负责什么？**

LLM 在基础任务里预测文本，reasoning model 能跟结构、输出 tool call、在 protocol 里工作。coding agent 多走一步——它把 model 放进一套 **runtime** 。这个 runtime 能查真实 repo、请求 tool、编辑文件、跑检查、记住发生过什么、在多轮中继续推进。 **这层 runtime 就是 harness。**

对 coding agent 来说， **harness 本身就是 product** 。

## Mini Agent 的盲区：5 个真实会爆的边界

一个 mini 版的 coding agent 通常画成「observe → model → tool → observe」的循环草图。这张草图有用，但同一个系统跑起来很快会出五类问题：

- Model 要编辑一个它从没读过的文件，会发生什么？
- Shell command 触碰 workspace 外的文件，会发生什么？
- Tool 返回 50,000 行输出，会发生什么？
- 文件已经在磁盘上变化，transcript 里仍然保留旧 file read，会发生什么？
- Tool result 和产生它的 tool call 对不上，会发生什么？

这个 system 还 **没有成为 coding agent** 。Model 可以提 proposal，harness 负责做决定。

## 关键 surface 不在这一行代码

从 architecture 角度去看，关键 surface 超出「model → tool」这一行。 **它更接近一张多组件协作图** ——每一次交互是一根带方向的箭头， **每一根箭头都是 harness 可以保护用户的位置，也可能是它悄悄失控的位置** 。

## Coding Agent 的六个核心组件

把 coding agent 压成六块：

### 1. Live repo context

Agent 不该从空 prompt 开始。它需要知道 workspace、当前文件、相关 project docs、 **哪些 repo state 可以安全暴露** 。

### 2. Prompt shape

**Context quality 经常看起来像 model quality** 。稳定的 prefix、清晰的 tool contract、当前 request、受控的 history——这些带来的行为差异 **可能比换 model 更大** 。

### 3. Structured tools

Tools 不应被当成 helper functions。它们是 **model proposal 和真实 side effects 之间的 contract** 。Harness 解析 arguments、验证 paths、检查 policy、执行操作、裁剪 output、记录发生了什么。

### 4. Context reduction

如果 harness 盲目追加所有东西，model 最后会看到太多、太少、或者看到错误的东西。 **好的 context 是一个 projection，而非不断膨胀的 blob** 。

### 5. Transcripts and memory

- **Transcript 回答：「发生过什么？」**
- **Working state 回答：「现在什么重要？」**

**这是两份不同的任务，必须区别对待** 。

### 6. Delegation

Subagents 也不等于 magic parallelism。它们应该是 **有边界的 workers** ——scoped tools、isolated state、distilled results。 **当 delegation 充当 context firewall 时，它才真正有用** 。

## Model 在 loop 里，harness 拥有 loop

Coding agent 是一个 observe-act loop。 **真正有价值的部分发生在 observe 和 act 之间** 。Model 在 loop 里，harness 拥有 loop。这个区别很关键。

假设 model 发出一个 tool call—— `edit src/config.py` 。Model 做出了一个 proposal。 **harness 接下来要回答一串问题** ：

- `src/config.py` 是否在 workspace 内？
- 这个 path 是否会通过 symlink 逃逸？
- 这是新文件，还是 overwrite？
- Model 最近是否读过现有文件？
- 已知的 file baseline 是否过期？
- 这次 write 是否需要 human approval？
- Edit result 是否应该包含 diff summary？
- 后面是否应该跑 validation command？
- 有多少 output 应该进入下一轮 prompt？
- 需要更新哪些 state，避免下一轮 stale？

**这些判断不该交给 model 的随机处理** 。Prompt 可以描述期望行为。 **Harness 负责落实边界** 。

## 一旦你开始看见 harness，debug 会变简单

只要开始看见 harness 这一层，很多 coding-agent 行为都会变得 **更容易诊断** ：

- Agent 不断重复 → 检查 loop 和 retry policy
- 编辑 stale code → 检查 file-state baselines
- 越跑越差 → 检查 context projection
- 运行了意外的东西 → 检查 permission policy
- 无法从 tool errors 恢复 → 检查 tool result objects
- 解释不清发生了什么 → 检查 traces、audit、doctor surfaces

Model 当然重要—— **只是 model quality 只占其中一层** 。真正的 agent experience 来自 model **周围的整套 harness** 。

## 一条工程规则，不只是 slogan

"the harness is the product" 不只是 slogan。它是一条工程规则：

> **Anything that must be reliable belongs in the harness.**

下一篇会讲 **The Stale Read Trap** ——值得继续追这个作者。

---

## 与本仓库其他文章的关联

| 本文概念 | 对应文章 |
|---------|---------|
| "Harness 拥有 loop" | #28 Ralph、#41-43 Loop Engineering 三部曲 |
| "Context quality 看起来像 model quality" | #3 LangChain Context Rot、#4 Anthropic Context Anxiety |
| "Tool 是 contract" | #5 HumanLayer 六大杠杆、#19 传感器谱系 |
| "Context 是 projection" | #17 Claude Code 逆向工程的 context 管理 |
| "可靠的东西属于 harness" | #2 Fowler 控制论框架、#1 OpenAI 六大概念 |

---

**收录说明：** 本文为中文原文完整版，作者为马东锡 NLP (@dongxi_nlp)，是其 Harness 系列文章的观点收束。另有姊妹篇 #18《关于 subagent》已收录为 `works/dongxi-subagent-original.md`。使用 baoyu-url-to-markdown 抓取于 2026-08-07。
