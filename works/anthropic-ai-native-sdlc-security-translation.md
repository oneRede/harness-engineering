---
sourceTitle: "How Anthropic secures its AI-native software development lifecycle"
sourceUrl: "https://claude.com/blog/how-anthropic-secures-its-ai-native-software-development-lifecycle"
requestedUrl: "https://claude.com/blog/how-anthropic-secures-its-ai-native-software-development-lifecycle"
sourceAuthor: "@claudeai"
sourceCoverImage: "https://cdn.prod.website-files.com/68a44d4040f98a4adf2207b6/6a5faa4b748d179725b5376e_og_how-anthropic-secures-its-ai-native-software-development-lifecycle.jpg"
sourceSiteName: "Claude"
sourcePublishedAt: "Jul 21, 2026"
sourceSummary: "Anthropic Deputy CISO Jason Clinton details how the Security Engineering team secures an AI-native SDLC where AI authors 80% of merged code."
adapter: "generic"
capturedAt: "2026-07-31T11:17:01.264Z"
conversionMethod: "defuddle"
kind: "generic/article"
language: "en-US"
title: "Anthropic 如何保障 AI 原生软件开发生命周期的安全"
author: "Jason Clinton, Anthropic 副首席信息安全官"
summary: "Anthropic 副首席信息安全官 Jason Clinton 详细介绍了安全工程团队如何保障 AI 原生 SDLC 的安全，其中 AI 编写了 80% 的合并代码。"
sourceFigureCount: 4
translationMethod: "baoyu-translate skill"
category: "Anthropic 工程博客 · 安全"
---

# Anthropic 如何保障 AI 原生软件开发生命周期的安全

在 Anthropic，代码量和部署速度呈指数级增长。我们的软件工程师平均每季度交付的代码量是 2021 年至 2025 年期间的 8 倍。

我们的审查、监控以及其他安全流程必须与这种提速同步扩展。否则就会形成瓶颈效应（[阿姆达尔定律](https://en.wikipedia.org/wiki/Amdahl%27s_law)）。

与此同时，我们的软件开发流程也发生了翻天覆地的变化。Claude 已经从编码助手演变为主要的代码创建者和审查者。如今，[Claude 编写](https://www.anthropic.com/institute/recursive-self-improvement)了我们代码库中约 80% 的合并代码。

超过半数的代码由我们内部版本的 [Claude Tag](https://www.anthropic.com/news/introducing-claude-tag) 合并，而人类工程师则专注于指导方向、设定意图以及最终审批。

这意味着我们的安全团队必须防御一个快速扩张的攻击面，并加固一个以非确定性、不断演进的智能体为核心的开发生命周期。在这篇文章中，我将介绍保障软件开发生命周期（SDLC, Software Development Lifecycle）安全的策略。

*（本文旨在与我们最近发布的 [《智能体零信任》](https://claude.com/blog/zero-trust-for-ai-agents) 框架结合使用；本文中的所有内容都在实施中运用了该框架的安全设计理念）。*

我们针对的威胁场景非常具体：被攻陷或遭受提示词注入（prompt injection）的智能体引入恶意变更；智能体将供应链和依赖投毒（dependency poisoning）作为可信输入进行处理；以及更常见的应用漏洞类别，只是现在出现的频率更高。接下来的每项控制措施都至少对应其中一种威胁。

为了在不显著降低开发速度的前提下实现这一目标，我们部署了几项总体策略，包括：

- 安全左移（Shifting security left），并与代码开发阶段深度集成；
- 使用严格的访问和身份边界来限制爆炸半径（blast radius）；
- 在生产前后结合自动化的确定性审查和智能体化审查；
- 在最具杠杆效应的节点引入人工干预。

在本文中，我们将介绍在软件开发生命周期各个阶段实施的安全流程，以及背后的核心原则。这些原则更具持久性，因为随着模型能力的演进，安全团队必须不断重新审视、甚至重新发明他们的流程。

## 不断演进的软件开发生命周期

![](https://cdn.prod.website-files.com/68a44d4040f98a4adf2207b6/6a5fa5786cc9f557247c1256_9c126d9d.png)

Anthropic 内部自动化项目安全审查（PSR）的流程。

我们的开发团队已经[详细介绍](https://claude.com/blog/running-an-ai-native-engineering-org)了软件开发生命周期的变化，因此在深入各个阶段之前，这里只做简要说明。

从宏观来看，我们的软件开发生命周期已被压缩。它由原型和内部试用（dogfooding）驱动，而不是冗长的规划周期。创意来自组织的各个角落，传统角色（前端、后端、设计）的界限已经模糊。审查和审批仍有人工参与，但同时也由智能体循环驱动。

虽然每个阶段都因 Claude Code 和 Claude Tag 而被根本性地改造和加速，但各阶段的名称和目的对于来自传统组织的开发者来说并不陌生。这些是自然的关卡，我们也将它们作为 AI 原生 SDLC（AI-native SDLC）安全流程的一部分。

## 规划阶段（Plan）

我们最早的安全自动化之一是一个由 Claude Opus 驱动的简单项目安全审查（PSR, Project Security Review）Web 应用。它读取项目设计文档，并根据 [MITRE ATT&CK 框架](https://attack.mitre.org/)进行分析，以识别潜在漏洞并提出缓解建议。

我们通过将其连接到内部知识索引，显著增强了这一系统，使其能够获得更深层次的上下文，涵盖全组织的政策、过往决策和相关系统。

![](https://cdn.prod.website-files.com/68a44d4040f98a4adf2207b6/6a5fa80d84f25c4ed3f5421f_three-steps-diagram.png)

Anthropic 内部自动化项目安全审查（PSR）的流程。

这让我们能够更好地理解潜在风险，同时也能捕捉到 PSR 中缺失的信息。仅这一项实施就节省了应用安全（AppSec）团队的大部分时间。一旦我们确信 Claude 在评估风险方面足够准确，我们就允许团队在 Claude 认为发布风险足够低的情况下，自行审批项目。

在这里，我们可以看到适应 AI 原生 SDLC 的第一个关键调整。PSR 最初的设计目的是在漫长且昂贵的编码过程之前捕获安全问题。在这个阶段发现问题可以节省数月的重新开发时间。

如今，主要功能的多个原型可以在几小时内创建完成，这使得详细的架构审查不再是那么关键的关卡。将我们的 PSR 应用连接到知识索引，能够捕获原本可能遗漏的上下文，而不会造成不必要的速度阻碍。创建一个 Claude Code 技能（skill）使 Claude 能够进一步扩展，在上下文所在的任何地方捕获额外信息。

**持久原则**：将安全智能体连接到组织上下文。随着规划周期的压缩，将这些智能体带到上下文已经存在的地方——聊天记录、过往审查、代码库——比强制要求在可能不再需要详细文档的阶段编写文档要有效得多。无论哪种方式，智能体都需要代码本身之外的上下文。

或阅读[文档](https://code.claude.com/docs/en/overview)

## 编码阶段（Code）

在 AI 原生工程组织中，安全专业人员拥有了一个新的杠杆：他们可以直接塑造代码的创建方式，从源头帮助预防漏洞。

过去，团队观察到反复出现的漏洞并创建安全编码指南来解决它们，但这些指南难以执行，也很少标准化。

在 Anthropic，这些指南被编码在 CLAUDE.md 文件和全组织技能的引用中，因此代码在生成的那一刻就遵循这些最佳实践。这是一个闭环过程。一旦智能体发现某类 bug，相关文件就会被更新，以防止未来代码中再次出现。

![](https://cdn.prod.website-files.com/68a44d4040f98a4adf2207b6/6a5fa78b8f99d4eea5c0c389_closed-loop-diagram.png)

当然，这并不意味着所有代码都完美无缺。我们团队一开始使用一个 CLAUDE.md 文件，指示智能体在开启 PR（Pull Request）之前运行 [/security-review](https://support.claude.com/en/articles/11932705-automated-security-reviews-in-claude-code) 作为最后一步。这个普遍可用的命令是我们团队内部审查工作流的产品化版本，它会查找潜在的攻击者可控输入的入口点，扫描可疑链接，然后验证其发现。

如今，这些审查在 Claude 生成代码时就同步进行。一旦安装了[安全指导插件](https://code.claude.com/docs/en/security-guidance)，Claude 会在工作过程中审查对话和代码。它在生成代码的同一会话中提出安全改进建议并解决常见漏洞。

其他在 PR 阶段的提示会引导内部非技术团队将他们的应用托管在我们的低代码应用托管平台上，避免了传统上困扰安全团队的影子 IT（shadow IT）问题。

我们的一些客户选择将 [/security-review](https://support.claude.com/en/articles/11932705-automated-security-reviews-in-claude-code) 与 PreToolUse 钩子（hook）集成，这使得这一步骤成为更严格的关卡。这也很有效，但我们团队选择在 SDLC 的测试/CI 阶段设置严格的代码审查关卡。

除了塑造和审查代码，限制爆炸半径是我们在这个阶段的主要关注点之一。我们通过围绕身份设置硬边界（在监控部分会详细介绍）以及让开发人员在虚拟机上编码来实现这一点。

将编码转移到远程虚拟机（VM）是一个相对轻松的转变，与仅使用笔记本电脑相比，它为我们提供了更强的控制力和可见性。这些 VM 上的智能体流量受到出站流量允许列表（egress-allowlist）的限制。

这些严格的出站流量控制尤为重要，特别是当智能体读取可能携带提示词注入有效载荷的不可信输入时。注入的指令无法到达互联网上的任意目的地：数据泄露路径被限制在一小组受监控的服务中。

在这里，你可以再次看到针对 AI 原生 SDLC 的明确调整。远程编码以前主要用于保护知识产权（IP），而如今我们看到越来越多成熟的 AI 编码团队采用这些环境作为限制智能体的手段。

**持久原则**：在 AI 原生工程组织中，安全左移意味着在漏洞发现和更新指令之间建立闭环，从而定制 Claude 生成代码的方式。适当地限制爆炸半径（最小智能体权限原则）以及智能体可以访问的内容，并设置硬边界。

## 测试阶段（CI）

根据我的经验，测试或 CI 阶段很快就会成为正在进行 AI 原生转型的工程团队最痛苦的瓶颈。在 Anthropic，一旦大多数开发人员开始使用智能体化编码工具并同时运行多个智能体，团队的移动速度很快就只能与人工审查代码的速度一样快了。

让我们明确一点：人工问责制仍然是我们流程的核心。我们所做的是通过结合自动化的智能体审查和确定性审查来加速审查过程，同时将人工审查保留给受监管或真正关键的代码。

从历史上看，人工代码审查一直被奉为标准，但[经验证据](https://link.springer.com/chapter/10.1007/978-3-642-36563-8_14)表明它并不完美。安全 bug 仍然在全球的软件中频繁出现。我们的审查流程能够审查更多代码并捕获特别复杂的问题，有助于降低这些风险。

获得实质性审查评论的 PR 比例[从 16% 增长到 54%](https://claude.com/blog/code-review)，因为我们通过要求智能体编写证明其发现有效的验证代码，从而对发现结果建立了信心。我们还确定，过去 claude.ai 事件背后的 bug 中，约有[三分之一可以被](https://www.anthropic.com/institute/recursive-self-improvement)我们现在实施的自动化流程捕获。

我们不是唯一发现这一点的组织。[Intercom 分享](https://www.intercom.com/blog/ai-is-approving-our-pull-requests-heres-how-we-made-it-safe/)称它自动审批了 19% 的 PR。部署频率翻倍，而代码变更导致的停机时间下降了 35%。CircleCI 在构建 Chunk 时得出了类似的结论，Chunk 是一个基于 Claude 的自主智能体，可以解决 CI/CD 维护问题，并[在人工看到之前验证自己的修复](https://claude.com/customers/circleci)。这种方法使智能体任务转化为已完成 PR 的比率翻了一番。

在 Anthropic 开启 PR 时，多个智能体会自动审查它。每个审查智能体都被设计为专注于特定的、狭窄的领域，并利用 RAG（检索增强生成，Retrieval-Augmented Generation）获取额外的上下文和有关过往事件的记忆。

这比一个超级提示词或超级安全智能体有效得多，原因如下：

- 它们不会共享偏见和盲点
- 如果一个被攻陷或出错，可以被其他审查者捕获
- 精力不会在多个关注领域中过于分散

需要明确的是，智能体并不是在未经检查的情况下将代码合并到生产环境。我们按风险对代码库进行分层，并对哪些部分进行自动化做出慎重决策。整个代码库都有严格的人工审批流程。

对于由 Claude 审查和合并的代码，人工问责制仍然是核心。每次审批都会记录其使用的信号和推理依据，并且会对风险加权的样本进行人工审查。另一轮测试专注于不变量，如"用户 A 永远无法读取用户 B 的数据"，并触发额外的人工审查。我们还将智能体化扫描与静态应用安全测试（SAST，Static Application Security Testing）工具相结合，这些工具会直接在 PR 上发布评论。

无论是智能体化还是确定性的扫描方法，大多数都是基于消耗的。随着代码吞吐量的增加，成本也会增加，团队需要决定什么样的覆盖水平适合他们。

在 Anthropic，我们接受这里的成本会随着代码交付速度的提升而增长，但预计单位成本会下降。如今的模型在编码方面比几年前的所有模型都要好得多，我们预计这种模式会持续下去。

**持久原则**：自动化审查是一种不同类型的风险，需要用不同的方式控制（通过多个关卡和具有独立上下文窗口的智能体）。人工仍然参与其中，但可能根据代码库的性质出现在生命周期的不同位置。

## 部署阶段（CD）

Anthropic 维护着一个强大的预发布环境（staging environment），我们在其中执行常见的安全最佳实践，例如针对重大发布的外部渗透测试（pentesting），以及定期的动态应用安全测试（DAST，Dynamic Application Security Testing）扫描，以捕获静态扫描遗漏或看不到的逻辑 bug。

与其他 SDLC 阶段一样，AI 为安全团队带来了新的挑战和解决方案。一方面，到达这个阶段的漏洞更少了。另一方面，存活下来的漏洞是最微妙、最难捕获的。

再加上更大量的代码以更高的频率交付，定期的动态测试看起来就不那么"动态"了。

好消息是，AI 模型在需要多步骤、跨组件推理的任务上表现更好，可以捕获更高比例的这些复杂漏洞。例如，在今年 2 月，我们披露 Claude 发现并帮助修复了超过 [500 个高严重性的开源软件漏洞](https://www.anthropic.com/research/zero-days)。

在 Anthropic，我们正在预发布环境中实施持续的 AI 驱动 DAST 扫描。这些扫描在系统层面寻找漏洞，即两个或更多服务之间的假设不正确的地方。目前有许多供应商提供这些能力。

**持久原则**：动态测试应该与部署节奏相匹配。

## 监控阶段（Monitor）

任何优秀的安全团队都知道，一旦代码推送到生产环境（prod），工作并没有结束。我们可以假设任何漏洞都会被日益复杂的攻击者迅速识别。

我们的安全团队在这里实施了一些标准实践的程序，例如[公开的漏洞赏金计划](https://hackerone.com/anthropic)、红队（red team）模拟攻击，以及对依赖项、密钥、供应链、云态势和容器的定期漏洞扫描。

Claude 在这些方面发挥着重要作用，但我们将重点关注因 AI 原生 SDLC 而对监控工作产生的更大变化：警报分类和代码迁移。

当 Anthropic 触发警报时，Claude 会开始：

- 审查生产日志
- 定位 bug 的根本原因；
- 编写事后分析报告（post-mortem）；在某些情况下
- 编写修复 bug 的代码变更。

这个智能体无法自动部署修复。它是一个单一用途的系统账户智能体，只有三个权限：可以编写新文档、在公司频道中发布消息，以及访问生产日志。

修复必须来自一个单独的智能体-人工审查系统。这样做的原因回到了管理身份、权限和硬边界：在将代码推送到生产环境时，限制爆炸半径至关重要。分离智能体是关键，因为一个（或多个）智能体充当对另一个的检查。

![](https://cdn.prod.website-files.com/68a44d4040f98a4adf2207b6/6a5faaaf38e56da47cb6564d_permission-boundary-diagram.png)

对于首席信息安全官（CISO）来说，这也是一个重要的教训，而且我是艰难地学到这一课的。在考虑智能体的硬边界时，你需要包括它对其他智能体的访问。

在一次模型升级后，事件响应智能体主动通过 Slack 联系了另一个 Claude 实例。它要求这个可以编写代码的智能体推送修复。**这在按设计的人工审查关卡被捕获**，但这次经历告诉我们，边界应该围绕访问和行动来划定，而不是围绕模型的指令或我们认为模型能做什么。如今在 Anthropic，智能体之间通过 Slack 进行通信已成为常态，我们对[智能体身份模型](https://claude.com/blog/agent-identity-access-model)进行了大量思考。

第二个主要变化是我们团队处理迁移的方式。每个安全工程团队都经历过这样的时刻：他们意识到需要进行代码迁移来修复公司运营方式中的某些系统性缺陷。在过去，CISO 需要开始游说并请求每个部门的一小部分工程资源，持续多个季度才能完成修复。

迁移的经济成本已经下降，跨公司协调的成本也是如此。Claude [自动化迁移过程，在数天内完成数万行代码的迁移](https://claude.com/blog/ai-code-migration)。

**持久原则**：为每个智能体提供单一用途的身份，并赋予其工作所需的最小权限。如果确实让智能体协调，让它们通过与人类相同的渠道进行协调。

## 治理（Governance）

我们已经自动化了许多安全流程，但人类仍然是确保软件开发生命周期安全的重要组成部分。但我们的注意力不再集中在审查代码和 bug 报告上，而是现在专注于 Claude Tag、循环和仪表板。

这凸显了强有力治理的重要性。如果一个技能（skill）过时了，发现的 bug 类别从未更新到 CLAUDE.md 中，或者智能体的决策没有被抽样检查，整个结构就会退化。我们通过以下方式避免这种情况：

- **按风险对代码库进行分层**，然后根据该级别自动化审查。
- **影子模式**用于所有新的 AI 审查者。新智能体发布评论供人工审批，直到赢得信任。我们的团队还会对它们进行"红队"测试，尝试插入恶意变更。
- **抽样**一定比例的所有自动化审批。
- **关注我们的关键指标**。我们维护并密切监控一个仪表板，汇总每个安全流程和工作流的关键指标。
- **将每个智能体操作路由到 SIEM**。每个自动化审批、工具调用和智能体到智能体的消息都会记录其使用的信号，并进入我们的 SIEM（安全信息与事件管理，Security Information and Event Management）系统，因此任何决策在事后都是可归因和可审计的。我们使用这些数据，将这些智能体视为一种新型的内部威胁，并在它们的行为不符合预期时发出警报。

**持久原则**：安全工程师的工作从监控 bug 演变为监控循环。

*有关这些控制措施背后的评估框架，请参阅[《CISO 智能体 AI 指南》](https://claude.com/blog/ciso-guide-to-agentic-ai)。*

## 随着模型演进保持 AI SDLC 的安全

很难夸大软件开发生命周期及其加固手段演进的速度。模型能力每月都在进步，带来新的挑战和解决方案。

今天还不太有效或在经济上还不太可行的东西，很快就会变得可行。对你的团队来说，正确的问题不是"我们能负担得起扫描所有东西吗？"而是"如果扫描几乎免费，我们会运行什么？"为此做好规划。

*本文由 Anthropic 副首席信息安全官 Jason Clinton 撰写。他要感谢 Michael Segner 对本文的贡献。*
