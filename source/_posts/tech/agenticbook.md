---
title: 《Agentic Design Patterns》记录
date: 2025-11-22
tags:
- AI agent
- AI
- tech learning
categories:
- AI
---

最近读了《Agentic Design Patterns》。之前做过agent相关的项目，所以书里的很多想法都或多或少接触过。这本书挺适合没有任何背景的人了解一下Agent和AI相关的技术，感觉非常全面。

一些感受就是Agent可能是编程语言的未来形式，或者说编程语言之上的一层抽象，以前我们用程序做一件事，我们可能需要用编程语言描述它，但现在我们可以完全使用自然语言来描述。通过自然语言的描述，LLM就能理解要做什么，那相比程序化的写法有什么好处呢？一是用户的input可以就是自然语言，可以比用户直接call某个特定的api包含更多信息，且更灵活（因为用户描述的自然语言可能需要灵活地call多个api来实现），那用户自然可以自己call这些api达到相同的效果，但是多了一层LLM后，就可以做到**语言就是实现**；二是模型可以经过微调适应更复杂多变、难以描述成程序语言的逻辑，比如说分析error log并做出下一步处理，或者根据特定标准评价一段代码。

所以基于当前LLM的推理速度，只有描述成编程语言后过于复杂的任务，才适合用LLM处理。而剩下的任务还是由传统代码实现（比如用MCP或tool的方式）。LLM可以处理几乎任何任务，但带来的问题就是：一可能存在幻觉和不确定，二比传统逻辑更大的开销（token和时间）。

下面记录一些我觉得比较有趣的pattern：

* Routing pattern: routing pattern就像编程语言里的if，但是通过LLM来做branching。如果以后LLM的推理足够快，会不会LLM也会成为像if语言一样标准化的语句？比如在程序执行的过程中，有一个指令就是 llm "一些逻辑"，然后代码会自动调用llm，根据当前程序运行的复杂状态判断branch到哪个位置。

* Learning and adaption pattern: 会不会LLM会在运行中自主修改自己的代码，去适应更加复杂多变的情况？或者就直接编写代码自主执行？这可能比较类似现在的Planning pattern，planning是自主确定某个有限制的行动序列，但未来的adaption可能会是自我修改代码。但可能有的问题是需要有办法去validate LLM对自己代码的修改。

* Memory management: 有了memory以后，agent能记住更多东西，就像一个人一样，它会拥有chat history以外的context。其实也和普通程序一样，比如我们会记录一些用户数据来让应用更符合用户偏好，LLM有了memory也可以达到相同的效果。

* Goal setting pattern: 这有点像强化学习，它让agent生成结果多了一个evaluation的步骤，就像reward function，只有当reward 达到特定的threshold，才会输出执行。感觉很适合和adaption pattern做结合，比如adaption的结果达到特定的goal，再执行下一步。

* Inter-Agent communication pattern （A2A）: 感觉和MCP比较类似，MCP把传统软件/API做了抽象，而A2A把Agent拿出来单独做了层抽象，可以像call MCP一样调用别的agent。

