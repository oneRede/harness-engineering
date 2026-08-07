---
title: "长时应用开发的 Harness 设计"
sourceTitle: "Harness design for long-running application development"
sourceUrl: "https://www.anthropic.com/engineering/harness-design-long-running-apps"
sourceAuthor: "Prithvi Rajasekaran (Anthropic Labs)"
sourcePublishedAt: "2026-03-24"
sourceSiteName: "Anthropic Engineering"
summary: "Anthropic Labs 官方工程博客，GAN 启发的三智能体架构实战——Planner 扩展规格、Generator 按 sprint 实现、Evaluator 用 Playwright 实测打分。解决两大核心问题：Context Anxiety（接近上下文极限时提前收尾）和 Self-Evaluation 失败（智能体过度称赞自己的平庸产出）。附实战对比：solo 20 分钟 vs 完整 harness 6 小时的质量天壤之别。"
sourceLanguage: "en"
language: "zh-CN"
translationMethod: "人工整理逐段翻译（cloud agent，对照原文全文）"
sourceFigureCount: 0
---

# 长时应用开发的 Harness 设计

> 本文来自 Anthropic Labs 团队的 Prithvi Rajasekaran，描述了通过多智能体架构提升 Claude 在前端设计和自主软件工程上的性能的先进技术。

---

## 要解决的核心问题

### **上下文窗口问题**

模型在冗长任务上会因上下文填满而失去连贯性。有些模型表现出"**Context Anxiety（上下文焦虑）**"——在感知到接近极限时提前收尾。通过结构化交接的上下文重置可以解决这一问题。

### **自我评估失败**

智能体会自信地称赞自己平庸的工作，尤其是在主观任务上。**将生成与评估分离**比自我批评更有效。

---

## 前端设计架构

系统使用受 GAN 启发的生成器-评估器循环，配合四个评分标准：

1. **Design quality（设计质量）** — 连贯的氛围和身份
2. **Originality（原创性）** — 定制决策 vs 模板默认值
3. **Craft（工艺）** — 技术执行的基本功
4. **Functionality（功能性）** — 独立于美学的可用性

评估器使用 **Playwright MCP** 与运行中的页面交互后再打分。运行耗时 5-15 次迭代，最多四小时。一个荷兰艺术博物馆的例子中，系统"完全抛弃原方案，将网站重新想象为一种空间体验：一个带棋盘地板的 3D 房间。"

---

## 全栈编码系统

### **三智能体架构**

| 智能体 | 职责 |
|--------|------|
| **Planner** | 将 1-4 句提示词扩展为完整规格，强调产品上下文而非实现细节 |
| **Generator** | 使用 React/Vite/FastAPI/SQLite 技术栈实现特性，按协商好的 sprint 合同工作 |
| **Evaluator** | 使用 Playwright 测试运行中的应用，根据产品深度、功能性、设计、代码质量的阈值打分 |

### **Sprint 合同机制**

Generator 和 Evaluator 在编码开始前**协商"完成"标准**，弥合高层级规格与可测试实现之间的鸿沟。

---

## 实战结果

一个复古游戏制作器的对比显示了巨大差异：

**Solo 单智能体运行（20 分钟，$9）：**
- 核心功能损坏
- 工作流僵化
- 屏幕空间浪费

**完整 Harness（6 小时，$200）：**
- 游戏玩法正常工作
- AI 辅助生成
- 精致界面，10 个 sprint 中实现了 16 个特性

---

## Harness 演化

随着 Opus 4.6 的改进，**sprint 构造变得不再必要**。简化后的 harness 在 3 小时 50 分钟（$124.70）内完成了一个 DAW（数字音频工作站）的构建，Generator 连贯工作 2+ 小时无需分解。

Evaluator 仍然有价值，用于捕捉模型能力边缘的"**最后一英里问题**"。

### **核心原则**

> "Harness 中的每个组件都编码了一个关于模型做不到什么的假设"，需要随着模型改进持续重新评估。

---

## 关键要点

**Evaluator 的有用性取决于任务难度相对于模型能力的位置：**

- 对于 Generator 可靠范围内的任务 → Evaluator 增加开销
- 对于能力边缘的任务 → Evaluator 提供有意义的提升

文章总结道：**"有趣的 harness 组合空间不会因模型改进而缩小。相反，它会移动。"**

---

## 与仓库其他文章的关联

| 本文概念 | 对应文章 |
|---------|---------|
| Context Anxiety | #3 LangChain Context Rot 的官方命名 + 解决方案 |
| 生成与评估分离 | #2 Fowler "做与查分离"、#43 `/goal` 的独立 evaluator |
| Sprint 合同机制 | 对 #16 Symphony 的 ticket 级编排在单任务内的微缩应用 |
| GAN 启发的架构 | 首次明确提出对抗验证的工程化实现 |
| Harness 组件即假设 | #2 Fowler "Harness 应随模型变薄"的 Anthropic 版表述 |
| Playwright MCP | #6 同团队 Lance Martin 文章的工具实例 |

---

**译者注：** 本文是 Anthropic 官方少有的长时自主编码完整实战案例，配合 #6（三大模式）、#7（Managed Agents）、#50（遏制）构成 Anthropic 的 harness 工程体系。"每个 harness 组件都是一个关于模型做不到什么的假设"这句话后来被 Fowler、Osmani 等广泛引用。
