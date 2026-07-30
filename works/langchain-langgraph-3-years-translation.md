---
# LangGraph 三年图工程实践

title: "LangGraph 三年图工程实践"
sourceTitle: "3 Years of Graph Engineering with LangGraph"
sourceUrl: "https://www.langchain.com/blog/3-years-of-graph-engineering-with-langgraph"
sourceAuthor: "LangChain Team"
sourcePublishedAt: null
translationMethod: "baoyu-translate skill (refined mode)"
translatedAt: "2026-07-30"
translatorAudience: "technical"
translatorStyle: "technical"
language: "zh-CN"
sourceFigureCount: 4
---

# LangGraph 三年图工程实践

![](https://cdn.prod.website-files.com/65c81e88c254bb0f97633a71/6a60b77e476b5a0300795e7c_100%20(1).png)

"图工程（Graph engineering）"这个概念在上周末浮出水面，由这条[推文](https://x.com/steipete/status/2078277297791189132)引发：

![](https://cdn.prod.website-files.com/65c81e88c254bb0f97633a71/6a60b8a312e166dbcab58c6e_Screenshot%202026-07-20%20at%206.39.28%E2%80%AFPM.png)

这是 X 平台（前 Twitter）AI 内容工厂生产的最新术语，与提示工程（prompt engineering）、上下文工程（context engineering）、约束工程（harness engineering）和循环工程（loop engineering）并列。虽然将这些术语称为流行语（buzzwords）既诱人又准确，但它们的存在和涌现自有其原因：它们确实描述了构建者面临的真实挑战和设计决策。

归根结底，目标是利用 LLM 的能力为我们做有用的事情。无论你使用提示、智能体（agents）、循环还是图，这些都是实现细节。之所以存在如此多的术语，是因为让 LLM 工作**很困难**。它们是一种新型的非健壮、非确定性软件，我们不断尝试新策略来让它们运作。而这些新策略催生了新的流行语。

抛开流行语不谈，将智能体系统表示为图（"图工程"）是利用 LLM 能力的一种非常合理的方式。具体而言，它允许你（作为构建者）将对系统应如何工作的预判施加到更受约束的路径中，而不是完全依赖 LLM 的判断。更具体地说，它让你在希望智能体遵循特定路径时能够更严格地控制行为。

这一直觉驱使我们在三年前构建了 [LangGraph](https://docs.langchain.com/oss/python/langgraph/overview)，作为帮助构建此类智能体系统的框架。如今，LangGraph 每月下载量超过 6500 万次，被初创公司和企业广泛使用。

与市面上众多其他智能体框架相比，LangGraph 之所以流行，是因为它在确定性路径（deterministic paths）和智能体步骤（agentic steps）之间取得的平衡。

以下是我们多年来将智能体系统构建为图所学到的经验。

## 将智能体建模为图

图为你提供了一种具体的方式来定义智能体所遵循的工作流。

在 LangGraph 中，[节点（nodes）](https://docs.langchain.com/oss/python/langgraph/graph-api#nodes)执行工作。节点可以是确定性代码、单次 LLM 调用、工具调用，或是具有自身内部循环的完整智能体。

[边（edges）](https://docs.langchain.com/oss/python/langgraph/graph-api#edges)定义接下来会发生什么。一些边是确定性的。另一些边是条件性的，基于节点的结果、当前状态或某些外部信号。

你可以将此视为状态机（state machine）。图定义了工作流、在其中流转的状态，以及步骤之间的转换。

## 何时将智能体表示为图

现实世界的智能体工作流通常具有可预测的结构：支持智能体在回答或升级之前对问题进行分类，编码智能体在提出更改之前检查代码库，合规工作流在执行外部操作之前需要审批。

图让你能够直接编码这种结构：有效的路径、模型可以选择的地方，以及系统应该强制执行确定性行为而不是每次都寄希望于模型做出正确决策的地方。

通过将系统表示为图，你正在编码关于该系统应如何工作的领域知识。正如提示（prompts）包含将你的智能体与通用 ChatGPT 区分开的领域知识一样，这些"[认知架构（cognitive architectures）](https://www.langchain.com/blog/what-is-a-cognitive-architecture)"也能做到这一点。

以一个使用三个子智能体进行搜索的知识库智能体为例：一个 **GitHub 智能体**用于代码、issues 和 pull requests，一个 **Notion 智能体**用于内部文档和 wiki，以及一个 **Slack 智能体**用于相关线程。工作流有三个固定阶段：分类、搜索、综合。

![](https://cdn.prod.website-files.com/65c81e88c254bb0f97633a71/6a60b947479f1133e19e8a2c_fanout_and_synthesize_langchain_brand_v6.svg)

结果是代码和模型推理协同工作：模型在能增加价值的地方进行推理，代码处理其余部分，智能体变得更便宜、更快速、更可预测。

## 何时不使用图

某些任务本质上更具智能体特性，将它们强制纳入确定性路径是错误的做法。在这些情况下，你不希望将系统表示为图，而只是使用一个[智能体编排框架（agent harness）](https://www.langchain.com/blog/the-anatomy-of-an-agent-harness)（如 [Deep Agents](https://docs.langchain.com/oss/python/deepagents/overview)）。

通用深度研究就是一个很好的例子：研究智能体需要以难以预先确定的方式进行规划、委派、搜索、阅读和综合。我们最初在预定义的 LangGraph 工作流上构建深度研究，然后转向了更具智能体特性的核心循环。[GPT Researcher](https://github.com/assafelovic/gpt-researcher)，一个流行的深度研究实现，也做出了同样的转变，[将其图形状的多智能体管道换成了 Deep Agents](https://github.com/assafelovic/gpt-researcher/blob/main/deep_agents/README.md)，使得规划、委派和上下文管理在编排框架中自然涌现，而不是在图中硬编码。

## 构建 LangGraph 教会我们的事

我们在过去三年中一直在构建由图驱动的智能体。以下是我们学到的经验。

**首先，智能体图通常不是有向无环图（DAGs）。**

生产环境的智能体需要循环：重试失败的工具调用、向用户询问缺失的信息、验证后修订答案、反复调用工具直到获得足够的上下文，以及在恢复之前暂停以获取人工输入。循环是智能体系统的核心部分，因此它们很可能不是 DAG。

**其次，循环是简单的图。**

[循环工程（Loop engineering）](https://www.langchain.com/blog/the-art-of-loop-engineering)并不是图的替代方案，更像是图的简化版本。[正如 David Khourshid 所说](https://x.com/DavidKPiano/status/2079209887158989231)，循环只是一个有向的、循环的图。事实上，基于简单智能体循环的 [LangChain](https://docs.langchain.com/build-overview) 框架就是构建在 LangGraph 之上的。

**第三，动态转换很重要。**

你并不总是想预先定义每条边。有时一个节点在运行时决定要创建多少工作。映射-归约（Map-reduce）是经典案例：将输入分割成片段，将每个片段发送给一个工作器，然后合并结果。工作器的数量取决于输入，而你无法提前知道这个数字。

LangGraph 通过 [`Send`](https://docs.langchain.com/oss/python/langgraph/use-graph-api#map-reduce-and-the-send-api) API 来处理这个问题，它让节点能够动态地将工作路由到一个或多个下游节点，而无需静态定义每个转换。

这很重要，因为有用的智能体系统混合了已知结构和运行时变化。你可能知道研究应该扇出（fan out）然后综合，但不知道会有多少个来源。你可能知道监督者应该将任务委派给工作器，但在任务开始之前不知道要使用哪些特定的工作器。图仍然需要在运行时保持灵活性。

## 实际上有什么是新的

将智能体系统表示为图并不新鲜，我们已经做了三年了！在这波新的"图工程"浪潮中，有什么变化吗？

一个宽容的解释是，改变的是你可以在节点内放什么。早期，节点是确定性代码或单次 LLM 调用。现在智能体本身已经足够可靠，可以信任它们完成实际工作，节点可以是一个完整的智能体运行——你在编排智能体，而不仅仅是 LLM 调用。

编码智能体就是一个很好的例子。它们是当今生产环境中最有效、最具影响力的智能体之一，将其作为节点嵌入到更大的图中是一种新近实用的模式。

考虑一个文档智能体，它将这样的 Slack 请求：

![](https://cdn.prod.website-files.com/65c81e88c254bb0f97633a71/6a60b8e64f4cd4dd93a114a6_Screenshot%202026-07-22%20at%208.11.31%E2%80%AFAM.png)

转化为准备好审查的 pull request：

![](https://cdn.prod.website-files.com/65c81e88c254bb0f97633a71/6a60b93c4f4cd4dd93a15455_classify_and_act_langchain_brand_v4.svg)

此图中的每个节点都位于确定性到智能体这个尺度上的不同点：

- **固定步骤：** Slack 和 Linear 操作由固定代码和 API 调用驱动。
- **模型步骤：** 分类器和综合步骤使用单次 LLM 调用，不带工具。
- **智能体步骤：** 参考文档智能体和概念文档智能体在各自相关的代码库中完成更开放式的工作。

这里确定性与智能体特性的混合，正是使这个文档智能体可预测、强大且高效的原因。

## 更大的理念

图工程并不是一个新想法。它是构建可靠智能体这一成熟方法的最新名称。

它与[循环工程](https://www.langchain.com/blog/the-art-of-loop-engineering)和[约束工程](https://www.langchain.com/blog/how-to-build-a-custom-agent-harness)背后的理念相同：在每个步骤中，将模型推理放在正确的位置，配以正确的上下文。

如果你想尝试图工程，不妨试试 [LangGraph](https://docs.langchain.com/oss/python/langgraph/overview)。

### 致谢

感谢 [@huntlovell](https://x.com/@huntlovell) 和 [@nfcampos](https://x.com/@nfcampos) 的深思熟虑的审阅。
