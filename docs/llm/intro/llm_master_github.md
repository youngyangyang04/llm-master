---
title: 2026最新大模型应用开发学习路线Github开源了：150+教程，从RAG、Agent到Transformer与面试
description: 介绍GitHub开源仓库LLM Master，一套面向程序员的大模型应用开发完整学习路线，收录150+篇中文教程，覆盖大模型入门、模型API调用、Prompt Engineering、RAG、Agent、MCP、微调、模型部署、Transformer、AI编程、项目实战和大厂面试题，并给出不同基础开发者的使用方法。
keywords: ["LLM Master", "大模型学习路线", "大模型应用开发学习路线", "大模型入门教程", "大模型开源教程", "RAG教程", "Agent教程", "MCP教程", "大模型微调", "模型部署", "Transformer教程", "AI编程", "大模型面试题", "GitHub开源项目"]
tags: ["大模型", "学习路线", "GitHub开源项目", "应用开发"]
---

# 全网最全的大模型应用开发学习路线，开源了

关于如果学习大模型知识的文章，我一直都有在写，除了发票公众号上，我也都整理到网站【卡码笔记】：[https://notes.kamacoder.com](https://notes.kamacoder.com)

在我写的文章里，有很多 Agent、Rag 相关文章在公众号上成为爆款：

仅仅是转发就上万的， 因为公众号有推荐机制，但看阅读量 很多水文也可以冲阅读。 

真正看一个文章的含金量 还是看转发量，读者是否想把这篇文章分享给其他人，这里我随便截两篇大模型技术文章的数据：

![](https://file1.kamacoder.com/i/web/20260914163239-rr6s4q.png?v=20261005)  

![](https://file1.kamacoder.com/i/web/20260914164019-vwogrq.png?v=20261005)

转发量上千的更是比比皆是：

![](https://file1.kamacoder.com/i/web/20260914163424-pb989g.png?v=20261005) 

![](https://file1.kamacoder.com/i/web/20260914163913-g8mmie.png?v=20261005) 

![](https://file1.kamacoder.com/i/web/20260914163948-jv3sop.png?v=20261005) 

![](https://file1.kamacoder.com/i/web/20260914164131-bfa6ok.png?v=20261005) 

很多读者都非常认可我的教程，还有要找我找文章PDF版本的。

![](https://file1.kamacoder.com/i/web/20260914163643-w3aixb.png?v=20261005)

而这次我直接把我的所有大模型技术文章开源，而且把文章按学习路线整理好，大家直接可以看 markdown 源文件。

开源地址： 

👉 [https://github.com/youngyangyang04/llm-master](https://github.com/youngyangyang04/llm-master)

我可以很有底气地说：**这是目前全网最完整、最适合程序员的大模型应用开发学习路线之一。**

它不是把一堆文章扔进文件夹，而是从第一次调用模型 API 开始，一路带你做到可评估的 RAG、可靠的 Agent、能上线的 AI 系统，最后再补齐 Transformer 原理和大模型面试。

![LLM Master仓库首页](https://file1.kamacoder.com/i/web/20260914160629-9qx7x3.png?v=20261005)

从仓库首页就能看到它的定位：**程序员的大模型全栈学习路线**。目前已经整理了 150+ 篇中文教程，而且还在持续更新。

## 这是一条能走通的路线

很多所谓的“学习路线”，最后只是给你列一张几十门课程的清单。

看起来很全，真正开始学时还是不知道先看哪个。

`LLM Master` 最大的区别，是它按知识依赖把内容拆成了六个阶段：

1. **全局认知**：先看懂岗位、术语和技术版图。
2. **模型调用**：掌握 Prompt、流式输出、结构化输出和 Function Calling。
3. **RAG**：打通切片、Embedding、检索、Rerank、引用和评估。
4. **Agent**：继续学习工具、状态、规划、记忆、权限和失败恢复。
5. **生产工程**：补齐部署、缓存、量化、压测、监控和容量规划。
6. **原理与面试**：理解 Transformer，把项目经验整理成面试表达。

![大模型应用开发完整学习路线](https://file1.kamacoder.com/i/web/20260914114955.jpg?v=20261005)

这张路线表最有价值的地方，不只是告诉你“学什么”，还写清楚了**每个阶段的完成标志**。

比如 RAG 不是“看完几篇文章”就算学会，而是要做出一个包含混合检索、答案引用和离线评测集的知识库；Agent 也不是调通一次工具调用，而是要做到可追踪、可中断、可恢复、可评估。

**学习最终要落到作品上。** 这才是程序员真正能执行的路线。

## 到底有多全？

我在本地统计了一下，目前仓库中已经有 **172 个 Markdown 文件**，包括教程、路线、专题索引和面试资料。

具体覆盖范围，可以看这张表：

| 方向 | 主要内容 |
|---|---|
| 基础入门 | 大模型关键词、训练流程、岗位选择、Token 与 API 成本 |
| 应用开发 | Prompt、结构化输出、流式响应、Function Calling、上下文工程 |
| RAG | Chunk、Embedding、向量数据库、混合检索、Rerank、评估与优化 |
| Agent | ReAct、工具设计、规划、记忆、状态、失败恢复、Multi-Agent |
| 微调 | SFT、RLHF、DPO、LoRA、QLoRA、蒸馏以及和 RAG 的选型 |
| 部署与性能 | vLLM、SGLang、KV Cache、PagedAttention、量化、压测 |
| Transformer | Q/K/V、Attention、位置编码、FFN、LayerNorm、手写实现 |
| AI 编程 | Claude Code、Agent CLI、Skills、Hooks、MCP、工程工作流 |
| 求职面试 | RAG、Agent、微调、Transformer、AI 编程和真实大厂面经 |

给大家看看仓库的文章： 

![](https://file1.kamacoder.com/i/web/20260914155817-8n19od.png?v=20261005)

![](https://file1.kamacoder.com/i/web/20260914155627-wt2r9z.png?v=20261005) 

![](https://file1.kamacoder.com/i/web/20260914155707-u0s6bq.png?v=20261005)

这里既有“Agent 是什么”这样的入门文章，也有 Context Engineering、Agent Harness、多 Agent 上下文治理、RAG 评估、PagedAttention 这类更偏生产实践的内容。

而且 Transformer 不是停留在背 Q、K、V 定义，仓库里还整理了 Attention、Multi-Head Attention、LayerNorm、FFN 和 Tiny Transformer 的手写代码。

**从应用到原理，从 Demo 到生产，从学习到求职，这几条线是连在一起的。**

## 150+篇教程，都已经做好索引

资料一多，最怕找不到。

所以仓库除了完整路线，还单独整理了全量资料索引和专题索引。你可以按 RAG、Agent、微调、部署、Transformer、AI 编程进入专题，也可以直接在全量索引中查具体问题。

地址：[https://github.com/youngyangyang04/llm-master/blob/main/docs/README.md](https://github.com/youngyangyang04/llm-master/blob/main/docs/README.md)

![](https://file1.kamacoder.com/i/web/20260914160032-b3843d.png?v=20261005)

截图只展示了索引开头的一部分。继续往下还有应用开发、Transformer、AI 编程与 Claude、行业动态、面试专题等完整栏目。

这意味着你既可以把它当课程，按路线从头学；也可以把它当一本大模型应用开发手册，做项目遇到问题时随时回来查。

## 连项目和面试都给你串起来了

只看概念，很容易产生“我好像懂了”的错觉。

仓库给每个阶段都安排了可交付项目：

- 做一个支持流式输出、JSON Schema、错误处理和成本统计的 AI 应用。
- 做一个包含混合检索、Rerank、答案引用和固定评测集的企业知识库。
- 做一个有权限边界、Checkpoint、Trace、失败恢复和完成率评估的工具型 Agent。
- 把服务继续做生产化，补上限流、缓存、监控、灰度、P99、TTFT、TPOT 和容量报告。

![](https://file1.kamacoder.com/i/web/20260914153009-yqvgpp.png?v=20261005) 


学完之后，你不只是多背了几个名词，而是手里真的有项目。

项目做完，还能顺着仓库里的大模型面试路线继续准备。里面既有 RAG、Agent、Transformer、微调等高频题，也有字节 Agent 开发四面、番茄小说 Agent 应用开发一面这类真实面经。

![](https://file1.kamacoder.com/i/web/20260914155159-qh0y47.png?v=20261005)

**知识、项目、简历、面试，形成了一个完整闭环。**

## 不同基础，怎么使用这个仓库？

如果你是第一次接触大模型，不要直接打开全量索引一篇篇刷。

从 完整学习路线：https://github.com/youngyangyang04/llm-master/blob/main/docs/roadmap/README.md 开始，按六个阶段走。

![](https://file1.kamacoder.com/i/web/20260914155305-dzyexq.png?v=20261005)

如果你已经做过模型 API 调用，可以直接进入 [RAG 专题](https://github.com/youngyangyang04/llm-master/blob/main/docs/topics/rag.md)或者 [Agent 路线](https://github.com/youngyangyang04/llm-master/blob/main/docs/roadmap/agent.md)，边看边做项目。

如果你正在准备大模型求职，就从[面试路线](https://github.com/youngyangyang04/llm-master/blob/main/docs/roadmap/interview.md)开始，先补目标岗位需要的专题，再练习“业务问题—技术选型—系统设计—评估指标—故障优化—最终结果”这套项目表达。

如果你只是遇到一个具体问题，就去[全部资料索引](https://github.com/youngyangyang04/llm-master/blob/main/docs/README.md)搜索关键词。

## 最后 

把分散的文章重新分类，检查链接，设计学习顺序，再补齐专题索引、项目标准和面试路线，这件事远比看起来费时间。

大模型技术还在快速变化，这个仓库也会继续更新 RAG、Agent、MCP、模型部署、AI 编程和真实面经。

如果这套资料对你有帮助，**希望大家去 GitHub 右上角点一个 Star 支持一下**：

👉 [给 LLM Master 点个 Star](https://github.com/youngyangyang04/llm-master)

地址：[https://github.com/youngyangyang04/llm-master](https://github.com/youngyangyang04/llm-master)

整理不易，你的一个 Star，就是我们持续更新下去最大的动力。
