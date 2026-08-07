---
title: "智能体 Harness 的解剖学"
sourceTitle: "The Anatomy of an Agent Harness"
sourceUrl: "https://www.langchain.com/blog/the-anatomy-of-an-agent-harness"
sourceAuthor: "Vivek Trivedy (LangChain)"
sourcePublishedAt: "2026-03"
sourceSiteName: "LangChain Blog"
summary: "LangChain 给出 harness 的精确定义和完整组件清单。核心公式：Agent = Model + Harness。Harness = 模型之外的一切代码、配置和执行逻辑。从期望的智能体行为反推 harness 设计，推导出六大组件：文件系统、Bash+代码执行、沙箱、记忆与搜索、上下文管理、长时执行。附 Context Rot（上下文腐烂）、Ralph Loop、Terminal Bench 2.0 案例。"
sourceLanguage: "en"
language: "zh-CN"
translationMethod: "人工整理逐段翻译（cloud agent，对照原文全文）"
sourceFigureCount: 0
---

# 智能体 Harness 的解剖学

> 本文由 Vivek Trivedy 提出了一个通过"harness engineering"视角理解 AI 智能体架构的框架。

---

## 核心概念

### **Agent = Model + Harness**

Harness 的定义：**"模型之外的一切代码、配置和执行逻辑。"**

这包括：
- 系统提示词
- 工具
- 基础设施
- 编排逻辑
- 执行中间件

---

## 六大核心组件

本文从期望的智能体行为**反推** harness 设计，推导出以下组件：

### **1. 文件系统（Filesystems）**

- 提供持久存储和上下文管理
- 使智能体能够跨会话保持工作
- 为多智能体系统创建协作界面

### **2. Bash + 代码执行**

- 给智能体一个通用工具，而非为每个动作预先构建工具
- 允许通过代码生成实现自主问题解决

### **3. 沙箱（Sandboxes）**

- 提供安全、隔离的执行环境
- 支持可扩展的智能体工作负载
- 包含预配置的工具，如运行时、CLI 和浏览器

### **4. 记忆与搜索（Memory & Search）**

- 基于文件系统的记忆（如 AGENTS.md）实现持续学习
- Web 搜索和 MCP 工具提供超越训练截止日期的访问能力

### **5. 上下文管理（Context Management）**

**Context Rot（上下文腐烂）：** 上下文窗口填满后性能退化。

**解决方案：**
- **Compaction 策略**：防止退化
- **Tool call 卸载**：减少噪音
- **Skills 渐进式披露**：避免前期加载过多工具

### **6. 长时执行（Long-Horizon Execution）**

- **Ralph Loop**：重新注入提示词，使工作跨上下文窗口继续
- **Planning**：帮助分解复杂目标
- **Self-verification**：为正确性创建反馈循环

---

## 关键洞察

### **Harness 与模型训练的耦合**

模型训练与 harness 设计正在变得耦合——模型在特定 harness 内进行后训练。

### **Harness 优化的价值**

尽管如此，harness 优化仍然有价值。作者展示了一个案例：

> **Terminal Bench 2.0：** 纯 harness 优化可以把排名从 Top 30 拉到 Top 5

**模型会 overfit 到特定 harness**——换 harness 表现可能暴跌。

---

## 未来研究方向

开放的研究领域包括：
- 在共享代码库上编排并行智能体
- 自我分析执行轨迹以修复失败
- 动态的即时工具组装

---

## 核心论点

> **模型提供智能，Harness 通过系统设计使智能变得有用。**

---

## 组件清单（完整版）

根据原文 `references/articles.md` 中的记录，完整组件清单包括：

- **System Prompts** - 系统提示词
- **Tools** - 工具
- **Skills** - 技能
- **MCP** - Model Context Protocol
- **沙箱基础设施** - 文件系统、浏览器
- **编排逻辑** - 子智能体、handoff、模型路由
- **Hooks/中间件** - compaction、续接、lint 检查

---

## 与本仓库其他文章的关联

| 本文概念 | 对应文章 |
|---------|---------|
| Agent = Model + Harness 公式 | 本文首次提出，后被 #31 Osmani、#37 马东锡广泛引用 |
| Context Rot | #4 Anthropic Context Anxiety 的 LangChain 版命名 |
| Ralph Loop | #28 Ralph 原始出处、#41-43 Loop Engineering 三部曲 |
| Harness 与模型训练耦合 | #13 Inside the Scaffold、#35 How good is your harness 论文 |
| Terminal Bench 2.0 案例 | 纯 harness 配置优化带来排名跃升的实证 |
| 渐进式披露 | #1 OpenAI "地图而非手册"、#2 Fowler 框架 |

---

**译者注：** 本文是 "Agent = Model + Harness" 公式的首次正式提出，是 harness engineering 学科的定义性文章。后续 #31 Osmani 学科汇流、#37 马东锡产品论断都大量引用本文。Terminal Bench 2.0 的案例首次用数据证明了"纯 harness 优化可以带来模型级别的性能提升"。
