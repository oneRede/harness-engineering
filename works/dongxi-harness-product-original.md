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
translationMethod: "中文原文收录"
sourceFigureCount: 0
---

# Harness 才是产品：决定 Coding Agent 体验的不是 Model，是它周围的整套 Runtime

> **核心论断：** 决定 coding agent 体验的不是 model 本身，而是围绕它的整套 runtime（即 harness）。第一个问题应该是"**Harness 到底负责什么？**"而非关注 model 参数。

---

## Mini Agent 的五个真实边界问题

在构建 coding agent 时，会遇到这些真实爆点：

1. **Model 要编辑从未读过的文件**
2. **Shell command 触碰 workspace 外的文件**
3. **Tool 返回 50,000 行输出**
4. **磁盘文件已变化，但 transcript 保留旧内容**
5. **Tool result 与 tool call 对不上**

**画出「observe → model → tool」草图的系统还不是 coding agent。**

---

## Coding Agent 的六个核心组件

### 1. Live repo context
需要知道 workspace、当前文件、相关文档及可安全暴露的 repo state。

### 2. Prompt shape
> **"Context quality 经常看起来像 model quality"**

带来的行为差异可能比换 model 更大。

### 3. Structured tools
是 **"model proposal 和真实 side effects 之间的 contract"**。

### 4. Context reduction
> **"好的 context 是一个 projection，而非不断膨胀的 blob"**

### 5. Transcripts and memory
- **Transcript** 回答"发生过什么"
- **Working state** 回答"现在什么重要"

### 6. Delegation
当充当 **context firewall** 时才真正有用。

---

## 关键架构原则

### **Model 在 loop 里，harness 拥有 loop**

当 model 发出一个 `edit src/config.py` tool call 时，这只是一个 **proposal**。

Harness 需要回答十来个裁决问题：
- Path 是否在 workspace 内？
- 会否通过 symlink 逃逸？
- Baseline 是否过期？
- 要否 human approval？
- 多少 output 进下一轮 prompt？
- 需要更新哪些 state？

**"这些判断不该交给 model 的随机处理。"**

---

## 工程规则

> **2026 年最锋利的工程规则：**
> 
> **"Anything that must be reliable belongs in the harness."**

这意味着：
- Prompt 可以描述期望行为
- **Harness 负责落实边界**

真正的 agent experience 来自 model 周围的整套 harness，而非 model quality 单一层面。

---

## 结论

**对 coding agent 来说，harness 本身就是 product。**

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

**收录说明：** 本文为中文原文，作者为马东锡 NLP (@dongxi_nlp)，是其 Harness 系列文章的观点收束。另有姊妹篇 #18《关于 subagent》已收录为 `works/dongxi-subagent-original.md`。
