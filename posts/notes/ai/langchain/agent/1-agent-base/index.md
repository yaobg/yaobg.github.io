# LangChain 核心模块 Agent 构建复杂应用的代理系统


<!--more-->

代理的核心思想是使用LLM来选择一系列要执行的动作。

- 在链式结构（Chains）中，一系列动作执行是硬编码的（ SequentialChain 和 RouterChain 也仅实现了面向过程）。

- 在代理（Agents）中，语言模型被用作推理引擎，以确定应该采取哪些动作以及执行顺序。

# AgentType

| AgentType                                     | 描述                                                         |
|-----------------------------------------------|------------------------------------------------------------|
| `ZERO_SHOT_REACT_DESCRIPTION`                 | 零样本 ReAct 代理，在执行操作之前进行推理。                                  |
| `REACT_DOCSTORE`                              | 带文档存储的 ReAct 代理，可查找文档存储中的相关信息回答问题。                         |
| `SELF_ASK_WITH_SEARCH`                        | 带搜索功能的自问自答代理，将复杂问题拆解为更简单的问题并搜索答案。                          |
| `CONVERSATIONAL_REACT_DESCRIPTION`            | 对话式 ReAct 代理，适用于多轮对话任务。                                    |
| `CHAT_ZERO_SHOT_REACT_DESCRIPTION`            | 聊天模型的零样本 ReAct 代理，在执行操作前进行推理，专为聊天模型设计。                     |
| `CHAT_CONVERSATIONAL_REACT_DESCRIPTION`       | 对话式聊天 ReAct 代理，可能是 `CHAT_ZERO_SHOT_REACT_DESCRIPTION` 的变体。 |
| `STRUCTURED_CHAT_ZERO_SHOT_REACT_DESCRIPTION` | 结构化聊天的零样本 ReAct 代理，能够调用具有多个输入的工具。                          |
| `OPENAI_FUNCTIONS`                            | OpenAI 函数代理，优化用于调用 OpenAI 的函数功能。                           |
| `OPENAI_MULTI_FUNCTIONS`                      | OpenAI 多函数代理，可能用于同时调用多个 OpenAI 函数。                         |


---

> 作者:   
> URL: https://yaobg.github.io/posts/notes/ai/langchain/agent/1-agent-base/  

