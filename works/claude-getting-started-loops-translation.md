---
title: "循环工程入门：四类循环的实战指南"
sourceTitle: "Loop engineering: Getting started with loops"
sourceUrl: "https://claude.com/blog/getting-started-with-loops"
sourceAuthor: "Delba de Oliveira, Michael Segner (Claude Code Team)"
sourcePublishedAt: "2026-06-30"
sourceSiteName: "Claude.com"
summary: "Anthropic 官方把 loop engineering 从社区话语固化为产品级定义：智能体重复工作周期直到满足停止条件。按触发方式、停止条件、原语、适用场景给出四类循环（Turn-based / Goal-based / Time-based / Proactive）的清晰分类学，配上质量与成本管理的双清单。"
sourceLanguage: "en"
language: "zh-CN"
translationMethod: "人工整理逐段翻译（cloud agent，对照原文全文）"
sourceFigureCount: 0
---

# 循环工程入门：四类循环的实战指南

**循环（loop）是智能体重复工作周期，直到满足停止条件。** 这是 Claude Code 团队给出的官方定义，也是本文的核心。我们按触发方式、停止条件、使用的原语和适用场景，把循环分成四类。

---

## 四类循环

### **Turn-based 循环（回合制循环）**

- **触发方式：** 用户提示词
- **停止条件：** Claude 完成任务或需要更多上下文
- **最适合：** 较短、非周期性任务
- **用量管理：** 具体的提示词 + 验证 skills

这是最基础的智能体循环——你手动引导每一轮。Claude 收集上下文、采取行动、检查工作、响应。你通过把验证步骤编码进 `SKILL.md` 文件来提升质量，使用定量检查。

**验证 skill 示例包括：**
- 启动开发服务器并打开编辑过的页面
- 直接与变更交互
- 检查浏览器控制台的错误
- 运行性能追踪

---

### **Goal-based 循环（目标驱动循环，`/goal`）**

- **触发方式：** 实时手动提示
- **停止条件：** 目标达成 **或** 达到最大轮数
- **最适合：** 有可验证退出标准的任务
- **用量管理：** 明确的完成标准 + 显式轮数上限

通过定义成功标准来扩展迭代。一个 evaluator 模型检查条件，持续工作直到满足条件或达到轮数上限。**在确定性标准（如测试数量或分数阈值）下效果最好。**

**示例：**
```
/goal get the homepage Lighthouse score to 90 or above, stop after 5 tries
```
（让首页 Lighthouse 分数达到 90 或以上，最多尝试 5 次后停止）

---

### **Time-based 循环（定时循环，`/loop` 和 `/schedule`）**

- **触发方式：** 指定的时间间隔
- **停止条件：** 用户取消或工作完成
- **最适合：** 周期性工作或对接外部系统
- **用量管理：** 更长的间隔或基于事件的反应

`/loop` 按间隔重新运行提示词（在你的计算机上运行）。`/schedule` 把循环移到云端。

**示例：**
```
/loop 5m check my PR, address review comments, and fix failing CI
```
（每 5 分钟检查我的 PR，处理审查意见，修复失败的 CI）

---

### **Proactive 循环（主动循环）**

- **触发方式：** 事件或计划，无需实时人工介入
- **停止条件：** 每个任务在目标达成时退出；routine 持续运行直到关闭
- **最适合：** 周期性、定义明确的工作流，如 bug 报告、分诊、迁移
- **用量管理：** 把任务路由到较小模型，用强模型做判断

组合多个原语，包括 `/schedule`、`/goal`、skills、动态工作流和 auto 模式。

**示例提示词结构：**
每小时检查反馈，分诊报告，使用并行 worktrees 和对抗性审查。

---

## 保持代码质量

系统设计决定输出质量：

1. **保持代码库整洁** — Claude 会遵循已有模式
2. **通过 skills 启用自我验证** — 把验证标准编码进去
3. **让文档易于访问** — 文档是智能体的知识库
4. **用第二个智能体做代码审查** — "带着全新上下文的审查者不那么有偏见，也不受主智能体推理的影响"

---

## 管理 token 用量

清晰的边界帮助控制成本：

- **选对原语和模型** — 模型和 effort level 选择是最大的成本杠杆
- **定义清晰的成功/停止标准** — 确定性判据最有效
- **大规模运行前先试点** — 小切片试跑
- **确定性工作用脚本** — 别让智能体做机械计算
- **让间隔匹配实际变化频率** — 别轮询太快
- **用 `/usage`、`/goal`、`/workflows` 命令复盘用量**

---

## 开始上手

推荐步骤：

1. **识别瓶颈** — 找到你是瓶颈的工作
2. **确定可交出的东西** — 验证检查、目标定义、还是计划任务？
3. **运行循环** — 选对原语，启动
4. **观察结果** — 看质量和成本
5. **迭代方法** — 根据观察调整

---

## 关键洞察

- **回应"控制激活策略"缺口：** 触发方式（人 / 目标 / 时间 / 事件）第一次有了官方分类学
- **`/goal` 的判停由独立 evaluator 模型执行** — "做与查分离"内置进原语
- **确定性判据最有效：** "测试通过数、分数阈值"这类可机械判定的条件让 `/goal` 不必自由裁量"够好了没有"
- **质量与成本双清单：** 质量靠"仓库本身干净 + skills 编码验证标准 + 文档易达 + 第二个智能体做 review"；成本靠"选对原语与模型、明确停止条件、大规模前先小切片试跑、确定性工作用脚本、复盘用量"
- **组合示例：** `/schedule` 每小时查 bug 报告 + `/goal` 定义完成 + workflow 三个 worktree 并行探索方案 + 对抗评审 + auto mode 免审批 — 四个原语拼成一条 proactive 流水线

---

## 四类循环对照表

| 循环类型 | 你交出的 | 适用场景 | 用什么 |
|---------|---------|---------|--------|
| **Turn-based** | 检查这一步 | 探索或决策中 | 自定义验证 skills |
| **Goal-based** | 停止条件 | 你知道"完成"长什么样 | `/goal`（独立 evaluator 模型判停，可设轮数上限） |
| **Time-based** | 触发时机 | 周期性工作 / 对接外部系统 | `/loop`（本机间隔重跑）、`/schedule`（云端 routine） |
| **Proactive** | 提示词本身 | 持续到来的明确定义工作流 | 以上全部 + dynamic workflows |

---

**译者注：** 本文是 loop engineering 从 Twitter 话语到 shipping feature 的标志文。与 #41 Osmani《Loop Engineering》（社区定调）、#42 Ronacher《The Coming Loop》（工程师视角的反思）形成三角 — Osmani 给概念、Ronacher 给警告、本文给产品级分类学和操作手册。
