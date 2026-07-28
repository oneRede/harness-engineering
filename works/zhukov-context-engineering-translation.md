---
title: "AI 智能体的上下文工程"
sourceTitle: "Context Engineering for AI Agents"
sourceUrl: "https://vzhukov.dev/posts/2026/context-engineering-for-ai-agents"
sourceAuthor: "Viacheslav Zhukov"
sourcePublishedAt: "2026-07-18"
author: "Viacheslav Zhukov"
coverImage: "https://vzhukov.dev/images/011/context-engineering-hero.png"
siteName: "Viacheslav Zhukov"
summary: "AI 智能体上下文工程实用指南：选择示例、检索、工具与 MCP、记忆、压缩、前缀缓存和评估。"
category: "技术博客 · Context Engineering"
translationMethod: "baoyu-translate skill"
language: "zh-CN"
sourceFigureCount: 3
---

# AI 智能体的上下文工程

![AI 智能体的上下文工程](https://vzhukov.dev/images/011/derivatives/crisp/1x/context-engineering-hero.png)

模型现在应该看到什么？

在我之前关于[文本分类的 few-shot 技术](https://vzhukov.dev/posts/2026/few-shot-techniques-for-text-classification-using-llms)的文章中，我比较了为 LLM 分类器选择示例的不同方法。问题看起来很狭窄：我们有输入，有一组可能的标签，需要决定向模型展示哪些示例。

但这个小问题实际上是进入一个更大问题的有用入口。

当我们构建智能体时，我们仍然在问同样的问题，只是有更多活动部件：

> 模型现在应该看到什么？

对于分类器，答案可能是：任务描述、标签列表和 20 个最近的示例。对于智能体，答案可能包括系统指令、用户请求、检索的文档、工具 schema、MCP 资源、之前的工具输出、记忆、压缩的历史、中间状态，可能还有几个示例。

这就是人们现在所说的**上下文工程（context engineering）**。我不认为这是一个神奇的新学科。它更像是提示工程长大后遇见了软件工程。提示措辞当然仍然重要（然而，它在现代 LLM 中越来越不重要），但在许多智能体系统中，更难的部分不是如何措辞指令。更难的部分是决定什么信息应该在何时以何种形式对模型可用。

在本文中，我将介绍我如何思考 AI 智能体的上下文工程。我们将从 few-shot 提示开始，然后转向检索、MCP/工具、记忆、压缩、前缀缓存和评估。

## 为什么上下文工程很重要

一个简单的 LLM 应用通常可以用一个好提示解决问题。你写清晰的指令，可能添加几个示例，调用模型，解析结果，然后继续。

智能体不同。它们通常运行多个步骤。它们调用工具。它们观察结果。它们可能搜索文件、检查代码、请求澄清、更新计划，或将工作移交给另一个智能体。下一次模型调用的输入不再只是"提示"。它是从许多来源组装的状态。

[Anthropic 将上下文描述为](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)从 LLM 采样时包含的 tokens，并将上下文工程定义为在模型约束下优化这些 tokens 的有用性。这个定义很简单，但已经足够。上下文窗口是有限的。即使它很大，它也不是免费的，不是均等关注的，也不总是干净的。

我多次看到同样的模式：

1. 从一个小提示开始。
2. 为边缘情况添加一条规则。
3. 添加示例。
4. 添加输出 JSON。
5. 添加工具指令。
6. 添加一些检索的文档。
7. 添加对话历史。
8. 添加另一条规则，因为之前的规则破坏了某些东西。

在某个时刻，提示变成了垃圾场。模型拥有所有信息，但仍然做出错误决策，因为重要部分被埋在陈旧的指令和无关的工具输出之间。

所以目标不是最大化上下文。目标是最大化**有用的上下文**。

![智能体上下文窗口作为工作台](https://vzhukov.dev/images/011/derivatives/article-image/context-workbench.png)

## 从 few-shot 提示到上下文工程

Few-shot 提示已经是小规模的上下文工程了。

在分类文章中，我比较了随机选择、最近邻选择、基于多样性的选择和其他几种策略。有趣的部分是示例不是训练数据。它们是**运行时数据**。我们改变了模型的输入，并在不更新模型权重的情况下改变了模型行为。

这正是上下文工程的思维模式。

对于 few-shot 分类，主要问题是：

> 我应该包含哪些示例？

对于智能体，问题变成：

> 对于这一步，我应该包含哪些指令、示例、文档、工具、记忆和之前的观察？

这是粗略的映射：

| Few-shot 分类 | 智能体上下文工程 |
| --- | --- |
| 任务提示 | 系统/开发者指令 |
| 标签列表 | 领域规则和输出 schema |
| 选定的示例 | 检索的示例、文档、记忆 |
| 输入文本 | 当前用户任务和智能体状态 |
| 一次模型调用 | 多次调用，上下文变化 |
| 准确率 / F1 | 任务成功率、工具正确性、成本、延迟、人工审查率 |

更大的上下文表面使一切变得更强大，也更脆弱。有了正确的检索示例，智能体可以正确分类奇怪的请求。有了错误的陈旧记忆，它可以自信地做错事。

这就是为什么我比预期更喜欢"上下文工程"这个短语。它促使我们将上下文视为设计的对象，而不是我们不断附加的巨大字符串。

![从 few-shot 提示到智能体上下文管道](https://vzhukov.dev/images/011/derivatives/article-image/few-shot-to-agent-context.png)

## 什么算作上下文

当人们说"上下文"时，他们通常指 RAG 检索的文档。那只是其中的一部分。

在智能体系统中，上下文可以包括：

| 上下文来源 | 示例 | 何时有用 | 失败模式 |
| --- | --- | --- | --- |
| 系统指令 | 角色、约束、输出契约 | 几乎总是 | 提示变得过长且矛盾 |
| Few-shot 示例 | 标记示例、工具使用示例 | 当行为需要演示时 | 示例无关或误导 |
| 检索的文档 | 产品文档、论文、工单、代码文件 | 当任务依赖外部知识时 | 检索带来嘈杂的块 |
| 工具 schema | 搜索工具、数据库工具、MCP 工具 | 当模型必须在自身之外行动时 | 工具名称/描述模糊 |
| 工具输出 | 搜索结果、API 响应、文件内容 | 在工具调用之后 | 巨大的输出污染下一步 |
| 对话历史 | 之前的用户和助手消息 | 多轮任务 | 旧指令与当前任务冲突 |
| 记忆 | 用户偏好、之前的决策、项目事实 | 重复任务或长时间运行的工作流 | 陈旧记忆成为虚假权威 |
| 压缩 | 早期工作的摘要 | 接近上下文限制的长任务 | 摘要丢失了重要细节 |
| 轨迹/反馈 | 人工审查笔记、评估标签 | 调试和改进 | 反馈未连接到未来检索 |

一个有用的测试是：如果改变它可以改变下一个模型输出，它就是上下文。

这包括工具 schema。像 `search(query: str)` 这样的工具描述就是上下文。更具体的工具如 `search_project_docs(query: str, section: Literal["api", "billing", "auth"])` 也是上下文，但有更好的契约。模型看到它并学习哪些操作是可能的。

[模型上下文协议（Model Context Protocol）](https://modelcontextprotocol.io/docs/getting-started/intro)使这一点更加明确。MCP 是将 AI 应用连接到外部系统的开放标准。服务器公开资源、工具和提示。客户端（你的智能体）看到可用的内容并可以请求它。MCP 提供商的完整列表定义了智能体在该步骤可以看到和做什么。

## 上下文工程的实践维度

现在让我们深入几个实践维度。

### 1. 选择示例

这直接来自 few-shot 分类。

- **随机选择**：简单但不一定有用。
- **最近邻选择**：根据相似性检索示例（嵌入、BM25）。
- **基于多样性的选择**：最大化示例之间的覆盖范围。
- **硬示例挖掘**：包含模型过去失败的示例。

对于智能体，你可能根据任务类型动态选择示例。如果任务是"生成 SQL 查询"，检索 SQL 示例。如果任务是"总结代码"，检索总结示例。

关键是不要使用固定集。每一步都检索相关示例。

### 2. 检索策略

智能体通常在运行时从文档存储中检索信息。

**检索的内容**：
- 产品文档
- API 参考
- 之前的工单
- 代码片段
- 标签定义

**如何检索**：
- 语义搜索（嵌入 + 向量数据库）
- 关键词搜索（BM25、Elasticsearch）
- 混合（重新排序器）
- 元数据过滤（按日期、类型、用户）

**何时检索**：
- 在开始之前（预检索）
- 在每一步（动态检索）
- 仅在需要时（响应工具调用）

动态检索更准确，但会破坏前缀缓存。预检索保留了稳定的前缀，但可能包含无关文档。

### 3. 工具和 MCP

工具定义 schema 和描述是上下文的一部分。

**糟糕的工具描述**：
```python
def search(query: str) -> list[str]:
    """搜索某些东西。"""
```

**好的工具描述**：
```python
def search_project_docs(
    query: str,
    section: Literal["api", "billing", "auth"],
    max_results: int = 5,
) -> list[Document]:
    """
    在项目文档中搜索查询。
    返回最多 max_results 个匹配文档。
    """
```

更好的 schema = 模型更少混淆。

MCP 更进一步。你不是硬编码工具列表，而是公开服务器，智能体发现可用的内容。这意味着工具可以根据用户、项目或步骤动态变化。

### 4. 记忆

记忆是智能体在步骤或会话之间记住的事实。

**存储什么**：
- 用户偏好（"我更喜欢 TypeScript"）
- 项目事实（"主数据库是 Postgres"）
- 之前的决策（"我们决定不使用 Redis"）

**何时写入记忆**：
- 在任务结束后
- 当用户明确说明偏好时
- 当智能体学到新事实时

**何时检索记忆**：
- 在任务开始时（根据任务查询检索前 K 个）
- 在每一步（如果记忆可以改变决策）

**何时过期**：
- 使用置信度分数
- 使用时间戳
- 使用显式失效（"忽略之前关于 X 的内容"）

陈旧记忆比没有记忆更危险。

### 5. 压缩和总结

当上下文变得太长时，你有几个选择：

- **删除旧步骤**：保留最后 N 条消息。
- **总结历史**：将早期步骤总结为简短摘要。
- **删除工具输出**：保留工具调用，但删除大输出。
- **仅保留关键事实**：提取决策和发现。

Anthropic 描述了[扩展上下文窗口](https://www.anthropic.com/engineering/extended-context-windows)，其中压缩不是在 token 限制之前预防性地完成，而是当智能体接近限制时动态完成。

研究表明，长上下文中的相关信息经常被遗漏（"中间迷失"效应）。因此即使你有 200K 上下文窗口，将重要信息放在开头或结尾附近仍然有帮助。

### 6. 前缀缓存

许多 LLM 提供商（Anthropic、OpenAI）支持[前缀缓存](https://docs.anthropic.com/en/docs/build-with-claude/prompt-caching)。如果你的提示有一个稳定的前缀（系统指令、工具定义），提供商会缓存它并在下一次调用时重用它。

这可以显著降低延迟和成本。

**使缓存有效**：
- 将稳定的块放在开头（系统指令、工具 schema）
- 将动态的块放在结尾（用户输入、检索的文档）
- 避免在每一步改变工具定义
- 避免在每条消息中改变系统提示

**打破缓存**：
- 动态检索（每次不同的文档）
- 个性化系统提示（每个用户不同）
- 对话历史（每次增长）

权衡：更好的缓存命中 vs 更相关的上下文。

### 7. 构建上下文管道

我更喜欢将上下文视为一个管道，而不是一个字符串。

```
任务 → 检索示例 → 检索文档 → 检索记忆 → 组装上下文 → 渲染消息 → 调用模型
```

每个步骤都是可测试的。你可以记录选择了什么。你可以比较策略。

## 一个实用示例

让我们用代码使想法具体化。我将使用一个小的论文/请求分类助手，因为它很好地连接到 few-shot 文章。

智能体接收一段短文本并返回结构化分类。它可以检索示例、检索标签定义、使用记忆，并公开一小组工具。

首先，定义数据结构：

```python
from enum import StrEnum
from pydantic import BaseModel, Field

class ScientificArea(StrEnum):
    SOFTWARE_ENGINEERING = "software engineering"
    MACHINE_LEARNING = "machine learning"
    BIOCHEMISTRY = "biochemistry"
    PSYCHOLOGY = "psychology"
    HYDRAULICS = "hydraulics"

class FewShotExample(BaseModel):
    input_text: str
    label: ScientificArea
    reason: str | None = None

class RetrievedDocument(BaseModel):
    doc_id: str
    title: str
    text: str
    score: float

class AgentContext(BaseModel):
    task: str
    output_schema: str
    examples: list[FewShotExample]
    documents: list[RetrievedDocument]
    memories: list[MemoryItem]
    available_tools: list[str]
    compacted_state: CompactedState | None = None
```

然后将检索构建为单独的步骤：

```python
class RetrievedContext(BaseModel):
    examples: list[FewShotExample]
    documents: list[RetrievedDocument]
    memories: list[MemoryItem]

example_store = ... # 对标记示例的语义搜索
doc_store = ... # 对参考文档的语义搜索
memory_store = ... # 带语义搜索的每用户记忆存储

def retrieve_context(task_text: str, user_id: str) -> RetrievedContext:
    examples = example_store.search(task_text, k=5)
    documents = doc_store.search(
        task_text,
        filters={"type": "label_definition"},
        k=4,
    )
    memories = memory_store.search(user_id=user_id, query=task_text, k=3)

    return RetrievedContext(
        examples=examples,
        documents=documents,
        memories=memories,
    )
```

并有意识地组装模型输入：

```python
def build_context(task_text: str, user_id: str) -> AgentContext:
    retrieved = retrieve_context(task_text=task_text, user_id=user_id)

    return AgentContext(
        task=task_text,
        output_schema='{"label": string, "confidence": number, "notes": string}',
        examples=retrieved.examples,
        documents=retrieved.documents,
        memories=[
            m for m in retrieved.memories
            if m.confidence >= 0.7 and not m.is_expired
        ],
        available_tools=["search_label_docs", "ask_human_reviewer"],
        compacted_state=None,
    )
```

最后，将其渲染为消息。我更喜欢保持这个渲染步骤无聊且可测试：

```python
def render_messages(context: AgentContext) -> list[dict[str, str]]:
    system = """
你将技术文本分类到一个科学领域。
仅返回与提供的 schema 匹配的 JSON。
如果证据薄弱，降低置信度并解释缺少什么。
""".strip()

    context_block = context.model_dump_json(indent=2, exclude={"task"})

    return [
        {"role": "system", "content": system},
        {"role": "user", "content": f"上下文:\n{context_block}\n\n分类:\n{context.task}"},
    ]
```

这不是一个通用框架。重点更简单：**上下文是构建的，不是附加的**。一旦它被结构化，你就可以测试它、记录它、缓存稳定的部分，并比较策略。

![实用的上下文构建管道](https://vzhukov.dev/images/011/derivatives/article-image/context-building-pipeline.png)

## 如何评估上下文工程

听起来不错的上下文策略可能仍然使智能体变得更糟。

我至少会为每次运行记录这些：

```python
from pydantic import BaseModel

class ContextTrace(BaseModel):
    run_id: str
    selected_example_ids: list[str]
    selected_doc_ids: list[str]
    selected_memory_ids: list[str]
    available_tools: list[str]
    input_tokens: int
    cached_input_tokens: int = 0
    output_tokens: int
    latency_ms: int
    final_label: str | None = None
    human_review_required: bool = False
    task_success: bool | None = None
```

这个轨迹让你可以问有用的问题：

- 检索的示例实际上与成功的运行重叠了吗？
- 添加记忆是减少了人工审查，还是引入了陈旧的假设？
- 压缩是否在不损害任务成功的情况下降低了成本？
- 动态检索是否足够提高准确性以证明更差的前缀缓存重用是合理的？
- 当模型选择错误工具时，哪些工具描述是可见的？

[LangChain 的智能体工程状态](https://www.langchain.com/state-of-agent-engineering)报告说，可观察性和跟踪在构建智能体的团队中已经很常见。这是有道理的。没有轨迹，上下文工程就变成了感觉。你改变提示，添加检索，删除记忆，然后希望。

对于类似分类的任务，你可以使用准确率和宏 F1，就像我在 few-shot 文章中所做的那样。对于智能体，我还会跟踪：

| 指标 | 为什么重要 |
| --- | --- |
| 任务成功率 | 智能体完成了工作吗？ |
| 工具调用正确性 | 它是否用有效参数调用了正确的工具？ |
| 检索精度 | 选择的文档/示例有用吗？ |
| 人工审查率 | 智能体多久需要一次升级？ |
| 每个完成任务的成本 | 上下文可能很快变得昂贵。 |
| 延迟 | 多步智能体会放大慢调用。 |
| 缓存命中率 | 对稳定前缀和重复工作流有用。 |
| 失败类别 | 缺少上下文、嘈杂上下文、陈旧记忆、糟糕的工具 schema、糟糕的模型输出。 |

重要的部分是比较版本。不要仅仅因为听起来像智能体就添加记忆层。构建一个小的评估集，有记忆和无记忆运行，看看会发生什么。

## 实用检查清单

在为智能体设计上下文时，我会检查这个清单：

- 什么信息必须始终存在？
- 什么可以仅在需要时检索？
- 应该动态选择哪些示例？
- 对于这一步应该可见哪些工具？
- 工具 schema 足够具体吗？
- 应该总结或清除哪些工具输出？
- 应该将什么存储为记忆，何时过期？
- 应该压缩什么，什么必须保持精确？
- 哪些块应该保持稳定以进行前缀缓存？
- 轨迹中将记录什么？
- 哪个指标证明上下文更改有帮助？

这听起来有点机械，但我认为这就是重点。当上下文工程不再是一个模糊的短语，而是变成一组你可以检查的设计决策时，它就变得有用了。

## 结论

Few-shot 提示教会我们示例很重要。不是以某种抽象的方式，而是以非常实用的方式：改变示例，模型就会改变其行为。

上下文工程将相同的想法扩展到智能体。

示例仍然存在，但现在它们与检索的文档、工具定义、MCP 资源、记忆、压缩状态、工具输出和轨迹并列。其中一些上下文应该是稳定的。一些应该及时获取。一些应该写入记忆并稍后检索。一些应该在污染下一步之前被积极删除。

我不断回到的问题很简单：

> 模型现在应该看到什么？

如果你可以有意识地回答这个问题，记录答案，并衡量它是否有帮助，你就在做上下文工程。其余的是实现细节。重要的实现细节，当然。但仍然是细节。
