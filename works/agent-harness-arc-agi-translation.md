---
sourceTitle: "Agent Harness Design: Why an ARC-AGI-3 Score Tripled"
sourceUrl: "https://umesh-malik.com/blog/agent-harness-design-arc-agi-3"
title: "智能体线束设计：ARC-AGI-3 分数为什么翻了三倍"
requestedUrl: "https://dev.to/umesh_malik/agent-harness-design-why-an-arc-agi-3-score-tripled-38fo"
author: "Umesh Malik"
coverImage: "https://media2.dev.to/dynamic/image/width=1200,height=627,fit=cover,gravity=auto,format=auto/https%3A%2F%2Fumesh-malik.com%2Fblog%2Fagent-harness-design-arc-agi-3-cover.png"
siteName: "DEV Community"
publishedAt: "2026-08-04T19:53:13Z"
summary: "一个基准测试分数的变化揭示了关键问题：OpenAI 的 ARC-AGI-3 分数从 13.3% 升至 38.3%，模型权重未变。这对你的 agent 循环意味着什么。"
adapter: "generic"
capturedAt: "2026-08-06T08:30:34.569Z"
conversionMethod: "defuddle"
kind: "generic/article"
language: "en"
translatedAt: "2026-08-06"
translatedTo: "zh-CN"
sourceFigureCount: 0
---

# 智能体线束设计：ARC-AGI-3 分数为什么翻了三倍

一个基准测试分数从 13.3% 跃升至 38.3%，而模型完全没有变化。相同的权重，相同的推理预算，没有任务专用工具。唯一的变量是**智能体线束设计（agent harness design）**——模型周围的代码，它决定模型在每一轮看到什么。

这就是 OpenAI [ARC-AGI-3 harness 文章](https://openai.com/index/how-two-settings-tripled-our-arc-agi-3-scores/)的完整故事（发布于 2026 年 7 月底），也是今年最有用的基准测试争议——不是因为谁领先，而是因为它给大多数构建 agent 的团队从未测量过的东西标上了数字：**他们自己的 harness 正在丢弃多少性能。**

如果你正在生产环境运行多轮智能体（multi-turn agent），你很可能正在因为 OpenAI 提到的这两个原因，损失相当倍数的性能。

## TL;DR

- **智能体线束设计是模型与环境之间的代码**——格式化、历史记录、推理持久化、上下文淘汰。这是你写的部分，它为模型能力设定了上限。
- OpenAI 报告 GPT-5.6 Sol 在 ARC-AGI-3 公开集上从 **13.3% → 38.3%**，**模型未变**，仅启用两项配置：**保留推理（retained reasoning）**和**压缩（compaction）**。
- **保留推理**阻止 agent 每轮重新推导自己的结论。**压缩**用摘要替代盲目截断。OpenAI 同时报告 **约 6 倍更少的输出 token**。
- **ARC Prize 拒绝这种做法没有错。**排行榜必须固定 harness，否则就不再是模型对比。双方测量的是真实但不同的东西。
- 要点不是排行榜。而是**你应该今天就去对比自己 agent 的第 N 轮和第 N+1 轮**，找出你正在删除什么。

## 什么是智能体线束设计？

**智能体线束设计（agent harness design）是设计和实现模型与环境之间交互代码的工程实践：它决定观测如何渲染为文本、前序轮次中什么能留存、中间推理是否向前传递、上下文窗口满时什么被淘汰。**权重是冻结的、供应商提供的。线束是你的。

大多数团队不把它当作设计表面。他们把它当作胶水——一个 `while` 循环、一个消息列表、一个 append。这种定位就是系统如何在没有单个失败测试的情况下，悄无声息地丢失三分之二能力的原因。

ARC-AGI-3 让这种代价异常清晰，因为它是一个*交互式*基准测试。ARC Prize [描述它](https://arcprize.org/arc-agi/3/)挑战 agent「探索新环境、即时获取目标、构建可适应的世界模型、持续学习」。没有指令。agent 采取行动、观察网格、形成理论、测试、修正。每一步都依赖前一步。一个会遗忘的 harness 不是在这个任务上略逊一筹——它根本无法完成任务。

## 数据

| 配置 | ARC-AGI-3 公开集 | 来源 |
| --- | --- | --- |
| GPT-5.5，官方 harness | 0.4% | ARC Prize 排行榜 |
| GPT-5.6 Sol，官方 harness | 7.8% | ARC Prize 排行榜 |
| Claude Opus 5，官方 harness | 30.2% | ARC Prize 排行榜 |
| GPT-5.6 Sol，OpenAI 基线运行 | 13.3% | OpenAI |
| GPT-5.6 Sol，保留推理 + 压缩 | 38.3% | OpenAI |
| 人类测试者（平均） | ~48% | ARC Prize |

表格中有两点比标题更值得关注。

首先，诚实的差距：OpenAI 自己复现的*官方* harness 得分 13.3%，而非 ARC Prize 榜上的 7.8%。这是双方都认为是相同配置的基线中 5.5 个百分点的差异，并且没有解释。三倍提升的声明是相对 OpenAI 自己的基线测得的，不是公开的那个——这个细节在这个故事的每个总结中都丢失了。

其次，38.3% 是自我报告的、在公开集上的、尚未被独立复现的。[The Decoder 的报道](https://the-decoder.com/openai-claims-gpt-5-6-sol-beats-opus-5-on-arc-agi-3-with-its-latest-api-and-two-additional-settings/)正确地将其框定为声明而非结果。把它当作关于 harness 敏感性的强信号，而非已定的排名。

信号在两个警告之下都成立。无论确切基线如何，harness 变更带来的数字移动，都超过了同系列两代模型之间的差距。

## 保留推理：你的 agent 每轮重启自己的思考

用一句话描述这个失效模式：在标准多轮循环中，模型思考、发出动作、思考被扔掉。下一轮从可见记录开始——移动和结果——但不包括产生它们的推理。

所以 agent 看到它向左移动了并且网格改变了，但看不到*它为什么相信向左移动会测试它正在测试的假设*。它从证据重新推导理论，每一轮，而且不保证两次推导出相同的理论。探索任务变成了失忆症患者不断翻到自己笔记的独立猜测序列。

**保留推理通过跨轮次持久化模型的推理项（reasoning items），而非在动作边界丢弃它们，来修复这个问题。**在 OpenAI 的 Responses API（响应式 API）上，这意味着用 `previous_response_id` 链接轮次，而非手工组装消息数组——推理在服务器端保留并向前传递。

这也是约 6 倍输出 token 减少的来源。值得理解为什么*更*有状态的设置*更*便宜。重新推导结论每次都消耗推理 token。向前传递只消耗一次。效率增益不是独立的优化；它是从计费角度看到的同一个修复。

普适教训与 OpenAI 的 API 无关：**如果你的 agent 第 N+1 轮的 payload 不包含第 N 轮的推理，你就在反复为那个思考付费，并且每次得到不同答案。**大多数基于 chat-completions 风格 API 构建的手工循环默认剥离推理，因为 API 形状鼓励你自己重建消息列表。

## 压缩：截断是无声的脑叶切除术

第二项配置解决上下文耗尽时发生什么。

大多数 harness 中的默认行为是滚动截断：丢弃最老的消息直到 payload 装得下。这是一行代码，而且是探索型 agent 最糟糕的淘汰策略，因为在探索中**最老的观测是承重最大的**。第 3 轮是你学到规则的地方。第 90 轮是你应用它的地方。截断删除第 3 轮。

**压缩用浓缩摘要替换最老轮次中已建立的内容，而非删除它们。**OpenAI 的实现在约 175,000 token 阈值时触发。结论留存；记录不留。

权衡是真实的，值得指明：压缩是一种无法检视的有损操作。截断的上下文至少是可读的——你确切知道什么丢失了。压缩过的上下文被一个过程总结过，其省略内容不可见，直到 agent 对它不再知道的东西采取行动。你在用一个已知的、灾难性的损失，交换一个未知的、更小的损失。这通常是正确的交换，而且它仍然是个交换。

如果你在自己构建压缩层，有用的纪律是明确总结*结论和约束*，而非要求通用摘要。「目前建立的规则」和「排除的假设」值十倍于它们的 token 成本。对发生了什么的散文回顾几乎不值钱。

## ARC Prize 也是对的

很容易把这读成 OpenAI 暴露了一个有缺陷的基准测试。不是，而且反方立场很直接。

ARC Prize 运行标准化 harness——无供应商特定配置——因为排行榜的全部工作是比较，比较要求除变量外一切固定。一旦每个供应商发布自己的 harness，排名测量的供应商工程努力就和模型能力一样多，「模型 A 击败模型 B」这句话就停止携带信息。

所以两个声明同时为真：

- **ARC Prize 测量模型。**在相同条件下，Claude Opus 5 得分 30.2%，GPT-5.6 Sol 得分 7.8%。该比较有效，而且是排行榜应该发布的数字。
- **OpenAI 测量产品。**没人发布标准化 harness。用户接触的是 ChatGPT 和 Codex，它们运行保留推理和压缩配置。那个数字也有意义，而且它不是排行榜数字。

错误是要求一个数字做两份工作。排行榜回答「哪个模型」，而你几乎从不问那个问题。你在问「我的系统变好了吗」，只有你的 harness 能回答它。这和你在自己 harness 上跑的评估为什么在决定发布什么时击败公开基准测试的原因相同。[链接此处的评估框架文章](https://umesh-malik.com/blog/llm-eval-framework-smevals)。这也是[模型发布基准测试表](https://umesh-malik.com/blog/deepseek-v4-flash-0731-benchmarks)应该被读作能力下限、而非你将观察到什么的预测的原因。

## 实际该怎么做

四件事，按回报大小排序。

1. **对比你的轮次。**记录一次真实多轮运行的第 N 轮和第 N+1 轮的完整输入 payload 并比较它们。如果第 N 轮的推理或工具调用理由在 N+1 中缺失，那就是发现。这花二十分钟，而且是唯一真正非可选的步骤。
2. **找到你的淘汰策略。**读代码——或框架文档——看上下文限制时发生什么。如果答案是「丢弃最老的消息」，你有截断问题。用保留已建立事实和排除分支的总结替换它。
3. **使用你的提供商提供的有状态路径。**如果你的 API 有保留推理的响应链接原语，使用它而非重建消息数组。手工历史管理是推理被悄无声息丢弃的方式。
4. **把 harness 纳入测试。**给它版本号，并且在它变更时重跑你的评估套件。harness 变更是与提示变更相同爆炸半径的部署——而且根据这个证据，远超模型升级。

## 团队在哪里搞错这个

最常见的错误是把 harness 工作当作要完成的管道，而非要拥有的组件。症状：没有版本号、没有测试、没有负责人、以及一个自原型以来未被阅读过的 `while` 循环。

第二个是当 agent 跨轮次表现不一致时伸手要更大模型。跨轮次不一致是状态丢失的特征，而不是推理能力弱——而且升级模型让被丢弃的推理更贵，而非更便宜。这是和[发布 demo 并称其为生产](https://umesh-malik.com/blog/production-grade-ai-agents-vibe-to-live-gap)相同的陷阱：原型工作是因为运行足够短，以至于还没东西被淘汰。

第三个是过度纠正成永远把一切塞进上下文。压缩存在是因为无界上下文既不可负担也不有效。目标是保留*结论*，不是日志。

## 要点

这个故事中有趣的声明不是 38.3%。而是朴素 harness 和深思熟虑的 harness 之间的差距，大于同系列两代模型之间的差距——而且直到供应商去找排行榜论据时没人发现它。

你的 agent 的 harness 不是胶水代码。它是决定你的模型知道什么的组件，根据当前证据它可能是你栈中杠杆最高的代码。大多数团队从未对它分析过。去对比两个连续轮次，找出你一直在删除什么。

如果你接下来在构建周边基础设施，[设计 agent 到人类的移交](https://umesh-malik.com/blog/agent-to-human-delegation)和[搭建 MCP 服务器](https://umesh-malik.com/blog/how-to-build-mcp-server)是同一个「模型实际看到什么」问题再次出现的两个地方。

## FAQ

**什么是智能体线束（agent harness）？**
智能体线束（agent harness）是模型周围的代码，决定模型在每一轮看到什么：观测如何格式化、历史的哪些部分留存、中间推理发生什么、更老的上下文何时被丢弃或总结。模型权重是固定的；harness 是你实际写的部分。因为它控制输入，它为性能设定了上限，任何模型能力都无法突破这个上限。

**什么是保留推理（retained reasoning），为什么重要？**
保留推理意味着将模型的内部推理项（reasoning items）向前传递到下一轮，而非在动作发出后丢弃它们。没有它，多轮智能体每轮仅从可见记录开始，重新推导它已经形成的假设，并且再次为那个思考付费。在 OpenAI 的 Responses API 上，它通过 `previous_response_id` 链接轮次启用，而非手工重建消息数组。

**什么是上下文压缩（context compaction）？**
压缩在上下文窗口满时，将更老轮次的浓缩摘要放入上下文窗口，而非硬截断最老的消息。截断是最糟糕方式的有损——它悄无声息地删除最早的观测，在探索任务中它们通常是承重最大的。压缩保留结论并丢弃记录。

**OpenAI 的模型在 ARC-AGI-3 上变好了吗？**
没有。OpenAI 报告相同模型、无额外训练、无更大推理预算、无任务专用工具。唯一改变的是在模型和基准测试环境之间调停的 harness，这正是结果对工程师而非模型研究者有趣的原因。

**为什么 ARC Prize 使用标准化 harness 而非每个供应商的最佳配置？**
因为排行榜是比较，比较要求除模型外一切固定。如果每个供应商提供自己的 harness，数字测量的供应商工程努力就和模型一样多，跨模型排名就停止有意义。两种立场都站得住脚：ARC Prize 测量模型，OpenAI 测量产品。

**我如何知道我自己的 agent 有这个问题？**
记录一次真实多轮运行的第 N 轮和第 N+1 轮发送的确切输入 payload 并对比它们。如果第 N 轮的推理或工具调用理由在 N+1 中缺失，你在丢弃它。然后检查你的框架在上下文限制时做什么——如果答案是「丢弃最老的消息」，你在截断，不是压缩。

## 来源

- OpenAI，[「如何通过启用两项配置将我们在 ARC-AGI-3 基准测试上的分数提升三倍」](https://openai.com/index/how-two-settings-tripled-our-arc-agi-3-scores/)——13.3% → 38.3% 数据、两项配置、以及约 6 倍输出 token 减少。
- ARC Prize，[ARC-AGI-3](https://arcprize.org/arc-agi/3/)——基准测试的设计目标和标准化 harness 立场。
- The Decoder，[「OpenAI 声称 GPT-5.6 Sol 通过其最新 API 和两项额外配置在 ARC-AGI-3 上击败 Opus 5」](https://the-decoder.com/openai-claims-gpt-5-6-sol-beats-opus-5-on-arc-agi-3-with-its-latest-api-and-two-additional-settings/)——官方 harness 比较数字和复现状态。

---

*写于 [umesh-malik.com](https://umesh-malik.com/)——关于 AI、Web 开发和工程的无废话技术写作。*

---
