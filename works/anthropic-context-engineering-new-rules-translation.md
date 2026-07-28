---
title: "Claude 5 代模型的上下文工程新规则"
sourceTitle: "The new rules of context engineering for Claude 5 generation models"
sourceUrl: "https://claude.com/blog/the-new-rules-of-context-engineering-for-claude-5-generation-models"
sourceAuthor: "Thariq Shihipar (Anthropic MTS)"
sourcePublishedAt: "2026-07-24"
author: "@claudeai"
coverImage: "https://cdn.prod.website-files.com/68a44d4040f98a4adf2207b6/6a639c6b2e881afef80764c6_og_the-new-rules-of-context-engineering-for-claude-5-generation-models.jpg"
siteName: "Claude"
summary: "我们为更高级的模型删除了超过 80% 的 Claude Code 系统提示。如何将我们学到的经验应用到你自己在 Claude Code 和智能体中的上下文工程。"
category: "Anthropic 工程博客 · Context Engineering"
translationMethod: "baoyu-translate skill"
language: "zh-CN"
sourceFigureCount: 2
---

# Claude 5 代模型的上下文工程新规则

我之前写过关于如何最好地[在 Claude Code 中使用上下文工程](https://claude.com/blog/effective-context-engineering)的文章。一年过去了，我们学到了很多——包括什么有效、什么无效，以及如何随着 Claude 能力的提升而改变我们的方法。

**关键要点：我们为更高级的模型删除了超过 80% 的 Claude Code 系统提示。**

本文将解释为什么我们做出这一改变，以及如何将这些经验应用到你自己的上下文工程中——无论是在 Claude Code 中还是在你自己的智能体中。

## 解除 Claude 的束缚（Unhobbling Claude）

在早期版本的 Claude Code 中，我们的系统提示包含大量约束和指令：
- "不要写注释，除非绝对必要"
- "使用简洁的变量名"
- "避免过度工程化"
- 数十条类似的规则

这些规则在当时是必要的——它们帮助引导模型产生更好的输出。但随着 Claude 变得更强大，我们发现了一个问题：**这些约束本身成为了噪音**。

![](https://cdn.prod.website-files.com/68a44d4040f98a4adf2207b6/6a63620bedb2b7813b1071e2_afa90c36.png)

通常，Claude 可以理解用户的意图并得出正确答案，但 Claude 必须更仔细地思考这些重叠和冲突的消息，然后才能决定做什么。

虽然这些约束曾经是避免最坏情况所必需的，但我们后来发现可以删除其中许多约束，让模型使用周围上下文和判断力。

此外，Claude Code 现在有更多工具。Claude 过去依赖 CLAUDE.md 作为记忆、信息和指导的来源。现在我们有 memory、artifacts 和 skills，Claude 可以使用这些工具创建跨会话加载和共享上下文的新方式。

或阅读[文档](https://code.claude.com/docs/en/overview)

## 过去与现在

许多以前的上下文工程最佳实践已经变成了迷思，包括：

![](https://cdn.prod.website-files.com/68a44d4040f98a4adf2207b6/6a63620bedb2b7813b107213_3979f6a1.png)

### 迷思 1："始终提供详细示例"

**过去：** 我们曾认为需要为每个任务提供详细示例，以"教导"模型正确的输出格式。

**现在：** Claude 5 代模型通常可以从任务描述本身推断出正确的格式。过多的示例实际上会浪费上下文窗口，并可能限制模型的创造力。

**新规则：** 仅在格式复杂或非标准时提供示例。让模型根据任务需求自行判断。

### 迷思 2："越多约束越好"

**过去：** 详尽的 DO/DON'T 列表被认为是必需的，以防止模型"偏离轨道"。

**现在：** 过度约束会造成冲突指令，迫使模型花费推理预算来解决矛盾，而不是专注于任务本身。

**新规则：** 提供高层意图和关键约束，信任模型填补细节。删除冗余或显而易见的指令。

### 迷思 3："所有上下文必须在前面"

**过去：** 系统提示应该包含所有可能需要的信息，以便模型"知道一切"。

**现在：** 渐进式披露（progressive disclosure）更有效——在需要时提供上下文，而不是预先加载所有内容。

**新规则：** 使用 Skills、References 和 @ 提及按需加载上下文。让模型在需要时请求信息。

## 实践中的应用

基于这些经验，以下是如何改进你自己的上下文工程：

### 1. 审计你的系统提示

检查你的系统提示中的每一条指令，问：
- **这条指令是否解决了模型实际存在的问题？** 如果不是，删除它。
- **这条指令是否与其他指令冲突？** 如果是，简化或合并。
- **模型是否可以从任务本身推断出这一点？** 如果可以，信任模型的判断。

### 2. 使用 Skills 而非系统提示

在 Claude Code 中，Skills 是比系统提示更好的上下文传递方式，因为：
- **按需加载**：仅在相关时才进入上下文
- **结构化**：清晰的名称和描述帮助模型理解何时使用
- **可组合**：可以组合多个 skills 而不会产生冲突

**示例：** 与其在系统提示中写"使用 TypeScript 严格模式"，不如创建一个名为 `typescript-strict` 的 skill，包含具体的配置和示例。

### 3. 使用 References 而非内联上下文

References 允许 Claude 引用深入信息，而不会让系统提示膨胀：

你可以 @ 提及文件将其作为 references 包含。References 允许 Claude 引用有关当前计划的深入信息。

这可能是 spec 文件、mockup，甚至整个代码库。通常你应该更喜欢代码形式的文件，因为它为 Claude 提供了清晰、高保真的指令——Claude 非常熟悉这种语言。例如，设计的 HTML mockup 通常比设计描述或截图产生更好的结果。

## 尝试简化

在你的系统提示、skills 和 CLAUDE.md 文件中，你可能需要像我们一样进行简化。我们推出了一个名为 `claude doctor` 的新命令，它也会自动帮助你完成此操作。有关专门提示更高级模型的更多详细信息，请查看我们的 [Fable 实战指南](https://claude.com/blog/a-field-guide-to-claude-fable-finding-your-unknowns)。

*本文由 Anthropic 技术团队成员 Thariq Shihipar 撰写。*
