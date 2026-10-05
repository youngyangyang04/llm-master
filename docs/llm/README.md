---
title: 2026最全大模型学习路线：从零入门大模型应用开发完整教程（RAG/Agent/微调/Transformer）
description: 程序员怎么学大模型？这是一条从零入门到上手项目的大模型学习路线。Java、C++、Go后端转大模型应用开发，不用从公式推导学起。11章覆盖大模型入门、Prompt工程、RAG检索增强、Agent智能体、微调选型、部署压测、Transformer原理与手撕实现，从会用到懂原理，从跑Demo到上生产，配套大模型面试与简历指南。每日更新首发公众号「卡码大模型」。
keywords: [大模型教程, 大模型应用开发, LLM教程, RAG教程, Agent教程, Transformer原理, 大模型微调, 大模型部署, Prompt工程, Llama架构, 大模型入门, AI应用开发, 大模型面试, LLM面试, Function Calling, 向量数据库, 大模型简历]
tags: [大模型, 应用开发, LLM, AI教程]
---

# 大模型学习路线：从零入门大模型应用开发

> 这里是「卡码大模型」专栏，一条给程序员的大模型学习路线。

<a href="https://programmercarl.com/xunlian/damoxing.html">
  <img src="https://file1.kamacoder.com/i/web/2026-03-02_18-13-50.jpg?v=20261005" style="width:100%;cursor:pointer;" alt="卡码大模型应用开发训练营">
</a>

本专栏持续更新中，目前最新文章发布在公众号：卡码大模型

<p align="center"><img src="https://file1.kamacoder.com/i/web/卡码大模型二维码.jpg?v=20261005" width="200" alt="卡码大模型公众号二维码"/></p>

我在[知识星球](https://programmercarl.com/other/kstar.html)里辅导过太多录友，对每年求职变化非常了解。

现在无论你是做Java后端、C++开发、还是Go开发，大模型相关的知识已经不是"加分项"了，越来越多公司的JD里加了一行："**有大模型相关经验优先**"。

于是很多录友来问同一个问题：**程序员到底该怎么学大模型？从零开始学，第一步学什么、按什么顺序学？**

难就难在，大模型方向太新了，**没有标准教材，没有成熟的学习路线**。网上文章要么太浅——跟着教程跑个Demo就完了；要么太深——上来就推公式推导，做应用开发根本用不上。后端转大模型应用开发的人，卡在中间，不知道下一步往哪走。

中间地带是空的。

**这个专栏，就是来补上这条学习路线的**：从大模型入门，到 Prompt、RAG、Agent，再到微调、部署、Transformer 原理，一步一步带你从"会用"走到"懂原理"。

> 温馨提示：如果你还没有用过 Codex、Claude Code 这些Agent工具，那就是上古时期的程序员，赶紧用起来：[ChatGPT Plus、Pro，Claude Pro充值方法](https://github.com/youngyangyang04/gpt-daichong)

## 这个专栏讲什么

**应用开发者需要懂的，讲清楚**

**算法岗才需要深挖的，点到为止就行。**

本专栏不是零散文章堆砌，是一条从"会用"到"懂原理"的完整学习路径。

讲解风格将延续[【代码随想录】](https://programmercarl.com/)的风格，循序渐进，把大模型时代，大家作为一名开发人员，需要掌握的知识，给大家讲清楚。


## 大模型应用开发全链路

### 第一章：大模型入门——是什么、怎么学、岗位怎么选

搞清楚大模型是什么、岗位怎么选、应用开发到底在做什么。

- 👉 [大模型学习路线：程序员从零入门到上手项目，该按什么顺序学](./intro/llm_learning_roadmap.md)（**新手先看这篇**，整条路线的总览）
- [大模型关键词全解：从Prompt到Agent到MCP，一篇搞懂13个核心概念](./intro/llm_keywords.md)
- [AI编程产品的三层架构](./intro/ai-coding-three-layers.md)
- 👉 [Claude学习专栏：Claude Code从入门到工程实践完整教程](./claude/)（**想系统学习Claude Code看这里**）
- [大模型应用开发、算法岗、开发岗什么区别](./intro/application_development.md)
- [大模型应用开发到底在做什么](./intro/app_dev_overview.md)
- [大模型API到底怎么计费](./intro/llm_pricing.md)
- [大模型到底是怎么训练出来的](./intro/how_llm_trained.md)
- [大模型蒸馏到底是什么？硬蒸、软蒸、蒸馏其他厂商模型，一篇讲明白](./intro/model_distillation.md)

### 第二章：Prompt 工程与大模型调用基础

从"会聊天"到"会开发"，掌握和大模型交互的基础能力。

- [从聊天框到业务系统：一个请求是怎么被大模型处理的](./app/model_integration.md)
- [Prompt Engineering不是“写提示词”：结构化Prompt设计](./app/prompt_engineering.md)
- [GPT-5.6 Sol Prompt怎么写？别再沿用5.5的操作方式](./intro/gpt56_sol_prompt_guide.md)
- [Few-shot、CoT与自我反思：示例、推理和自检怎么选](./app/prompt_fewshot_cot_reflection.md)
- [结构化输出：JSON Schema怎么约束](./app/structured_output.md)
- [上下文窗口有多大？Context Engineering入门](./app/context_engineering.md)
- [同步、异步、流式输出怎么选](./app/streaming_output.md)
- [Token、成本与延迟：大模型应用的三个硬约束](./app/token_cost_latency.md)
- [Function Calling详解：大模型怎么调用工具](./app/function_calling.md)

### 第三章：RAG 检索增强怎么学（从原理到优化）

掌握RAG全链路，从原理到优化到评估，能独立设计和面试回答。

- [为什么有了大模型还需要RAG](./app/why_rag.md)
- [RAG完整链路拆解：离线阶段和在线阶段](./app/chain_of_rag.md)
- [Embedding是什么：语义压缩与模型选型](./app/embedding.md)
- [向量数据库解决了什么问题](./app/vector_database.md)
- [RAG切片策略：四种方式对比](./app/how_to_chunking.md)
- [RAG系统答不准的常见问题排查](./app/rag_problems.md)
- [RAG优化思路：Query改写到Context压缩](./app/rag_optimization.md)
- [长文档与代码怎么检索？固定切片为什么不够](./app/long_document_code_retrieval.md)
- [Agentic RAG是什么：从传统RAG到智能体检索](./intro/agentic_rag.md)
- [RAG怎么评估？检索质量和生成质量的度量](./app/rag_evaluation.md)
- [面试官怎么问RAG？高频问题与回答框架](./app/rag_interview_framework.md)
- 更多文章持续更新中……

### 第四章：AI Agent 智能体设计与工程落地

理解Agent的设计思路、工程挑战和评估方法。

- 👉 [AI Agent 学习路线：程序员怎么从零学 Agent 开发，该按什么顺序学](./app/agent_learning_roadmap.md)（**想专攻 Agent 先看这篇**）
- [Agent到底是什么？和普通大模型问答有什么区别？](./app/agent_intro.md)
- [ReAct、Reflection、规划执行：Agent三种常见思路怎么选？](./app/react_reflection_planning.md)
- [Agent vs Workflow：什么时候根本不需要Agent？](./app/agent_vs_workflow.md)
- [工具设计决定Agent上限：Tool Use、Function Calling、参数Schema和返回值怎么设计](./app/agent_tool_design.md)
- [MCP协议详解：Agent工具调用的新标准，和Function Calling有什么区别](./app/mcp_protocol.md)
- [Agent为什么容易翻车？死循环、误调用、上下文污染、权限越界怎么兜底](./app/agent_failure_modes.md)
- [Agent的记忆：短期记忆、长期记忆、RAG到底什么关系](./app/agent_memory.md)
- [Agent怎么评估？任务完成率与可靠性度量](./app/agent_evaluation.md)
- [Plan-and-Execute怎么落地成DAG执行器？拓扑调度、Checkpoint与局部Replan](./app/plan_execute_dag.md)
- [Planner、Worker、Reviewer怎么分工？Multi-Agent角色边界、任务包与验收机制](./app/multi_agent_roles.md)
- [多Agent上下文、消息和Token怎么治理？独立Thread、Artifact与全局预算](./app/multi_agent_context_governance.md)
- [Browser Agent怎么读取大型网页？DOM清洗、可访问性树、局部读取与Token预算](./app/browser_agent_dom_context.md)
- 更多文章持续更新中……

### 第五章：大模型微调选型（SFT、LoRA、RLHF）

不亲手训，但要懂选型边界。面试必问。

- 持续更新中……

### 第六章：大模型部署、推理与压测工程化

从Demo到生产的关键一步，工程化能力是应用开发者的核心竞争力。

- [云API、托管推理还是自部署？大模型部署方案怎么选](./app/deployment_options.md)
- [KV Cache为什么会吃光显存？从PagedAttention到Prefix Cache](./app/kv_cache_paged_attention.md)
- [量化不是只看4bit/8bit：权重、激活和KV Cache怎么选](./app/model_quantization.md)
- [大模型服务怎么压测？TTFT、TPOT、吞吐与Goodput](./app/stress_testing.md)
- 更多文章持续更新中……

### 第七章：多模态大模型入门

不只是文本，了解多模态的原理、场景和工程挑战。

- 持续更新中……

### 第八章：Transformer 原理（应用开发者视角）

拆开大模型看看里面是什么，从应用开发者视角理解Transformer。

- [为什么都绕不开Transformer](./transformer/transformer_base_1.md)
- [数据流动全解析：从输入到输出每一步](./transformer/transformer_data_flow.md)
- [三种架构详解与对比](./transformer/transformer_base_encoder_decoder.md)
- [Attention机制：Q、K、V是什么](./transformer/qkv.md)
- [Attention计算全过程一步步拆解](./transformer/qkv_cal.md)
- [Multi-Head Attention：为什么一个头不够](./transformer/mha.md)
- [位置编码：Transformer为什么必须知道顺序](./transformer/pos_encode.md)
- [残差连接、LayerNorm、FFN：缺一不可的配角](./transformer/ffn_ln.md)
- [一层Transformer Block长什么样](./transformer/transformer_structure.md)

### 第九章：手撕 Transformer（面试代码实现）

用最简代码实现每个组件，加深理解，面试手撕有底气。

- [手撕Attention：不依赖框架从零实现注意力机制](./transformer/attention_code.md)
- [手撕Multi-Head Attention：从单头扩展到多头](./transformer/mha_code.md)
- [手撕LayerNorm与残差连接](./transformer/layernorm_residual_code.md)
- [手撕FFN：前馈网络代码实现](./transformer/fnn_code.md)
- [手撕Transformer Block：把组件拼起来](./transformer/transformer_block_code.md)
- [手撕Tiny Transformer：从零拼出完整模型](./transformer/tiny_transformer_code.md)

### 第十章：大模型家族与 Llama 架构

从标准Transformer进化到主流模型架构，学会读技术报告。

- 持续更新中……

### 第十一章：大模型最新动态与模型评测

大模型领域实时动态与产品分析。

- [ChatGPT暂停Pro 20X新订阅：200美元套餐暂时停售](./news/chatgpt-pro-20x-subscription-pause.md)
- [彻底杀疯了！分享9个让人惊艳的GPT-6 Astra案例](./news/gpt-6-astra-amazing-cases.md)
- [GPT-6 Pro开始向Pro用户开放：100美元、200美元套餐先上车](./news/gpt-6-pro-rollout.md)
- [GPT-6 Astra发布：跑分几乎刷满，但我更在意它开始接管鼠标了](./news/gpt-6-astra.md)
- [ChatGPT、Claude、Grok集体宕机：故障重叠93分钟](./news/chatgpt-claude-grok-outage-sept-2026.md)
- [Claude Fable 5.1发布：缓存价降75%](./news/claude-fable-5-1.md)
- [Claude永久提额25%？Codex重置额度](./news/claude-codex-usage-limits-reset.md)
- [混元Hy4 preview发布：770B开源、1M上下文](./news/hunyuan-hy4-preview.md)
- [OpenAI终止与Cursor合作](./news/openai-cursor-partnership-end.md)
- [GLM-5.3-Flash发布](./news/glm-5-3-flash.md)
- [DeepSeek V4-Flash-Vision-Exp多模态实测](./news/deepseek-v4-flash-vision-exp.md)
- [GLM-5.3发布](./news/glm-5-3.md)
- [DeepSeek Harness安装与使用教程](./news/deepseek-harness-guide.md)
- [DeepSeek V4全系列正式版上线，API执行峰谷新定价](./news/deepseek-v4-official-api-pricing.md)
- [Claude给文字打隐形水印](./news/claude-invisible-text-watermark.md)
- [DeepSeek V4-Flash正式版上线](./news/deepseek-v4-flash-official.md)
- [长鑫科技上市与DRAM存储芯片壁垒](./news/cxmt-ipo-dram-barriers.md)
- [GPT-5.6后不用Superpowers了](./news/gpt-5-6-no-superpowers.md)
- [Claude Opus 5发布](./news/claude-opus-5.md)
- [Kimi K3发布](./news/kimi-k3.md)
- [GPT-5.6发布](./news/gpt-5-6.md)
- [Claude Code为什么针对中国IP封号](./news/claude_code_china_ip_ban.md)
- [DeepSeek招Agent Harness工程师](./news/deepseek-agent-harness-hiring.md)
- [Codex Record & Replay 发布](./news/codex-record-replay.md)
- [Kimi K2.7-Code发布](./news/kimi-k2-7-code.md)
- [GLM-5.2发布](./news/glm-5-2.md)
- [MiniMax M3评测](./news/minimax-m3.md)
- [DeepSeek V4发布](./news/deepseek-v4.md)
- [GPT-5.5发布](./news/gpt-5-5.md)
- [Claude Opus 4.7发布](./news/claude-opus-4-7.md)
- [Claude Opus 4.8发布](./news/claude-opus-4-8.md)
- [DeepSeek V4降价75%实测](./news/deepseek-v4-price.md)
- [DeepSeek V4-Pro永久降价75%](./news/deepseek-v4-pro-permanent-price-cut.md)

## 大模型面经

学到能做项目，下一步就是把它换成 offer。这套专栏配套一个持续更新的[大模型面经汇总](../interview/llm/)，全是大厂真实考过的题，按方向整理好了，照着对照自查最省事：

- [Transformer 大厂面试题汇总](../interview/llm/transformer_interview.md)
- [RAG 大厂面试题汇总](../interview/llm/rag_interview.md) ｜ [RAG 落地最难的地方在哪](../interview/llm/rag_hardest_parts_interview.md)
- [Agent 大厂面试题汇总](../interview/llm/agent_interview.md) ｜ [多 Agent 通信与编排面试详解](../interview/llm/multi_agent_communication_interview.md)
- [SFT、RLHF、DPO 微调面试详解](../interview/llm/finetuning_sft_rlhf_interview.md)
- [Claude Code 大厂面试题汇总](../interview/llm/claude_code_deep_dive.md) ｜ [Vibe Coding 大厂面试题汇总](../interview/llm/vibe_coding_interview.md)
- [字节 Agent 开发四面面经](../interview/llm/20260506bytedance.md)

更多面经在[大模型面经汇总](../interview/llm/)里，按 Transformer、RAG、Agent、微调、AI 编程分好类，持续更新。

## 每日更新

**每日更新**首发在公众号「卡码大模型」，定期同步至网站。

<p align="center"><img src="https://file1.kamacoder.com/i/web/卡码大模型二维码.jpg?v=20261005" width="200" alt="卡码大模型公众号二维码"/></p>

扫码关注，每日更新不错过。
