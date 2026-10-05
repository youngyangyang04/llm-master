---
title: Claude学习专栏：Claude Code从入门到工程实践完整教程|适合小白
description: Claude与Claude Code中文学习专栏，从高效使用、CLAUDE.md、Skills、Subagents、Hooks、MCP和Prompt Cache，到Agent Loop、动态工作流、大代码库与生产级Agent工程实践，帮助开发者系统掌握Claude Code。
keywords: [Claude教程, Claude Code教程, Claude Code入门, CLAUDE.md, Claude Skills, Subagents, Hooks, MCP, Prompt Cache, Agent Loop, AI编程]
tags: [Claude, Claude Code, AI编程, Agent]
pageMeta: false
---

# Claude学习专栏

<a href="https://notes.kamacoder.com/qita/0024.claude-opus-5-china-guide.html">
  <img src="https://file1.kamacoder.com/i/web/2026-07-30_21-45-21.jpg?v=20261005" style="width:100%;cursor:pointer;" alt="">
</a>


> 从会用 Claude Code，到能把它接进真实工程。

很多人第一次用 Claude Code，只把它当成终端里的聊天框：问一句，改一段，出错了再补一句。

这样当然能用，但只用到了最浅的一层。

Claude Code 真正的价值，是把项目规则、上下文管理、工具调用、测试验证和多 Agent 协作串成一个可以持续运行的工程闭环。

这个专栏不按功能列表堆文章，而是沿着一条学习路线讲清楚：**先知道 Claude Code 能做什么，再理解它为什么这样工作，最后把能力沉淀成团队可复用的工程资产。**

## 第一阶段：先把 Claude Code 用明白

先建立完整能力地图，避免一上来就在 Skills、Hooks、MCP 之间来回试错。

- [Claude Code高效使用指南：5件事把它调教成一个会自己干活的团队](./claude_code_efficient_guide.md)
- [Claude Code完整使用指南：CLAUDE.md、Skills、Subagents、MCP、Hooks、Plugins怎么用](./claude_code_toolkit_guide.md)
- [CLAUDE.md到底怎么写？项目记忆、团队规范和上下文管理一篇讲明白](./claude_md.md)

## 第二阶段：理解上下文和代码库

Claude Code 不是把整个仓库塞进上下文。它需要搜索、读取、压缩和缓存，也需要你给它清楚的项目入口。

- [Claude Code怎么读懂大代码库？Agent搜索、CLAUDE.md、Hooks、Skills、MCP和LSP一篇讲明白](./claude_code_large_codebase.md)
- [Claude Code为什么快？Prompt Cache、Plan Mode、MCP工具加载和上下文压缩一篇讲明白](./claude_prompt_cache.md)
- [为什么Agent时代大家都在做CLI？Claude Code、Codex与命令行的前世今生](./agent_cli.md)

## 第三阶段：从Prompt走向Agent Loop

当任务从“一次回答”变成“读取代码、执行修改、检查结果、继续修正”，重点就不再是某一句 Prompt，而是整个闭环能不能稳定运行。

- [Claude Code作者说“不写Prompt，写Loop”：AI编程从提示词到Agent闭环到底变了什么](./claude_code_loop.md)
- [深入理解Claude Code：从CLAUDE.md到Hooks、Skills、Subagents](./claude_code_extensions_evolution.md)
- [Claude Skills实战：Anthropic几百个Skill总结出的9大分类和实战经验](./claude_skills.md)
- [Claude Code动态工作流详解：让Claude自己现写一套harness](./dynamic_workflows.md)
- [Loop Engineering实战：14步路线图，从判断要不要做到上线后守住](./loop_engineering_guide.md)

## 第四阶段：团队与生产工程

最后再看规模化问题：怎么拆任务、怎么隔离执行环境、怎么验证迁移结果，以及怎样让 Agent 真正进入生产系统。

- [百万行代码两周迁完：Anthropic如何用Claude Code做大规模代码迁移](./ai_code_migration.md)
- [Claude Managed Agents详解：把Agent的“大脑”和“双手”拆开](./managed_agents.md)

## 第五阶段：看清AI编程真正改变了什么

工具会继续变，但人和Agent怎么分工、什么能力会变得更值钱，是更长期的问题。

- [40万次Claude Code会话揭示：真正拉开差距的不是Prompt](./claude_code_400k_sessions.md)

如果你刚开始用 Claude Code，从第一阶段按顺序看就够了。

如果你已经在项目里使用，可以直接从自己卡住的位置进入：规则总失效看 CLAUDE.md，仓库太大看大代码库，任务跑不稳看 Agent Loop，需要并行和隔离再看动态工作流与 Managed Agents。
