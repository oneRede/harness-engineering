---
title: "Harness Engineering：在智能体优先的世界中驾驭 Codex"
sourceTitle: "Harness engineering: leveraging Codex in an agent-first world"
sourceUrl: "https://openai.com/zh-Hans-CN/index/harness-engineering/"
sourceAuthor: "Ryan Lopopolo (OpenAI)"
sourcePublishedAt: "2026-02-11"
sourceSiteName: "OpenAI"
summary: "OpenAI 官方文章，提出 Harness Engineering 六大核心概念：仓库即记录系统、地图而非手册、机械化执行、智能体可读性、吞吐量改变合并理念、熵管理。3 人团队用 Codex 从空仓库到 100 万行代码，零手写代码。本文是本仓库的学习起点，所有概念笔记的来源。"
sourceLanguage: "zh-CN"
language: "zh-CN"
translationMethod: "官方中文版"
sourceFigureCount: null
---

# Harness Engineering：在智能体优先的世界中驾驭 Codex

> **说明：** 本文为 OpenAI 官方发布的中文版文章。原文链接：https://openai.com/zh-Hans-CN/index/harness-engineering/
> 
> 由于 OpenAI 网站访问限制，本文件仅作为索引记录。请直接访问官方链接阅读完整内容。

---

## 文章概要

**作者：** Ryan Lopopolo (OpenAI)  
**日期：** 2026-02-11

### 核心内容

OpenAI 团队分享了如何用 3 人团队，借助 Codex 智能体，在 5 个月内从空仓库到 100 万行代码，**零手写代码**。

### 六大核心概念

1. **仓库即记录系统（The repository is the system of record）**
   - 不在仓库里的东西，对智能体不存在
   - Slack 讨论、Google Docs、脑子里的知识 = 对智能体不可见

2. **地图而非手册（Maps, not manuals）**
   - AGENTS.md 是目录页，不是百科全书
   - ~100 行的入口文件，指向更深层的文档
   - 渐进式披露

3. **机械化执行（Mechanical enforcement）**
   - 自定义 linter + 结构测试 = 不变量的守护者
   - Lint 错误信息里内嵌修复指令

4. **智能体可读性（Agent readability）**
   - 优先为智能体的推理能力优化
   - 选"无聊"技术（API 稳定、训练集覆盖好）

5. **吞吐量改变合并理念（Throughput changes merge philosophy）**
   - PR 生命周期很短
   - 纠错成本低，等待成本高

6. **熵管理 = 垃圾回收（Entropy and garbage collection）**
   - 智能体会复现仓库中已有的模式——包括坏模式
   - 定期后台任务扫描偏差、更新质量评分、发起重构 PR

---

## 关键数据

| 指标 | 数据 |
|------|------|
| 团队规模 | 3 人 → 7 人 |
| 时间跨度 | 5 个月 |
| 代码量 | ~100 万行 |
| PR 数量 | ~1,500 个 |
| 人均日 PR | 3.5 个 |
| 单次运行时长 | 6+ 小时 |

---

## 在本仓库中的地位

本文是本仓库的**学习起点**，所有概念笔记（`concepts/` 目录）的来源。

**相关概念笔记：**
- `concepts/00-overview.md` - 六大核心概念总览
- `concepts/01-repo-as-source-of-truth.md` - 仓库即记录系统
- `concepts/02-mechanical-enforcement.md` - 机械化执行
- `concepts/03-entropy-and-garbage-collection.md` - 熵管理与垃圾回收
- `concepts/04-agent-readability.md` - 智能体可读性
- `concepts/05-throughput-changes-merge.md` - 吞吐量改变合并理念

---

## 访问说明

**官方中文链接：** https://openai.com/zh-Hans-CN/index/harness-engineering/

由于 OpenAI 网站可能有访问限制，如无法直接访问，可参考本仓库 `concepts/` 目录中对六大概念的详细拆解笔记。

---

**收录说明：** 本文为 OpenAI 官方提供的中文版，无需翻译。本文件作为索引记录，方便本仓库统一管理。
