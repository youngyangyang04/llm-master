---
title: 深入理解 Claude Code：从 CLAUDE.md 到 Hooks、Skills、Subagents
description: "为什么写了 CLAUDE.md，Claude Code 还是会忘记测试？为什么有了 Hooks 还需要 Skills，有了 Skills 又为什么要拆 Subagents？本文沿着常驻规则、事件控制、按需能力、独立上下文四层演进，讲清 CLAUDE.md、Claude Code Hooks、Agent Skills 和子 Agent 的触发方式、上下文成本、确定性边界与组合用法，帮助 AI 编程团队搭出稳定可复用的 Agent 工作流。"
keywords: [Claude Code, Claude Code教程, CLAUDE.md, Claude Code Hooks, Claude Skills, Agent Skills, Subagents, Claude Code子Agent, AI编程, AI Coding, Agent工作流, 上下文工程, Context Engineering, Claude Code最佳实践, Agent编排, 多Agent, 大模型应用开发, Anthropic]
tags: [Claude Code, AI编程, Agent Skills, 多Agent, 上下文工程, 大模型入门]
---

# 深入理解 Claude Code：从 CLAUDE.md 到 Hooks、Skills、Subagents

大家好，今天不讲某一个 Claude Code 功能怎么配，而是把四个最容易混在一起的东西串起来：`CLAUDE.md`、Hooks、Skills、Subagents。

前面我们单独讲过 [CLAUDE.md 到底怎么写](./claude_md.md)，也拆过 [Claude Skills 的实战经验](./claude_skills.md)。但很多录友学完还是有个问题：**这些东西不都是给 Claude 加规则吗，到底为什么要分四套？**

因为它们控制的根本不是一件事。

先记住这条演进线：

**让 Claude 记住规则 → 让关键动作确定发生 → 让专项能力按需加载 → 让复杂任务交给独立角色。**

也就是：

`CLAUDE.md → Hooks → Skills → Subagents`

注意，这不是四个功能的发布时间线，而是一条**工程能力成熟线**。每往后走一步，都是因为前一层解决不了新的问题。

<!-- drawio源文件: ./drawio/claude_code_extensions_01_evolution.drawio -->

![Claude Code从CLAUDE.md常驻规则演进到Hooks事件控制、Skills按需能力和Subagents独立角色](https://file1.kamacoder.com/i/web/20260713152556_claude_code_extensions_01_evolution_compressed.png?v=20261005)

这张图回答的是：为什么项目从几条规则开始，最后会长出自动检查、专项流程和多个 Agent。后面的能力不是替换前面的能力，而是把“写给模型看的要求”一步步外化成控制点、能力包和独立执行单元。

## 第一阶段：先用 CLAUDE.md，让 Claude 知道怎么干活

假设你让 Claude Code 修改一个支付接口。

你每次都要重复交代：项目用 pnpm、支付金额统一用分、不能直接改数据库表、改完必须跑测试。

说一遍有用，但只对这次对话有用。任务一长、上下文一压缩，前面的要求就可能被淹没。

`CLAUDE.md` 解决的正是这个问题。它把跨任务都稳定的项目知识放进常驻上下文：

```md
## 项目规则

- 使用 pnpm，不要使用 npm
- 金额统一以分存储，禁止使用浮点数
- 修改支付模块后运行 pnpm test:payment
- 未经确认不要修改数据库 schema
```

Claude Code 每次进入项目都能看到，不需要你重新教育一遍。根目录写全局约束，子目录再补模块规则，模型走到对应目录时加载更具体的上下文。

所以 `CLAUDE.md` 的定位很清楚：**它是 always-on 的项目说明书。**

但它有两个绕不过去的边界。

第一，**看到了，不代表一定做到。**“修改后运行测试”仍然只是一条自然语言指令，最终要靠模型记住、理解并主动执行。

第二，**常驻上下文不是免费仓库。**发布手册、接口文档、排障流程全塞进去，每一轮都要带着一大包暂时用不到的信息，重点反而被稀释。

这就是下一层能力出现的原因：有些事不能只提醒，必须在正确的时间点自动发生。

## 第二阶段：再加 Hooks，把“应该做”升级成“触发就做”

还是“改完必须跑检查”这件事。

写在 `CLAUDE.md` 里的意思是：Claude，记得做。

写成 Hook 的意思是：只要编辑完成，这个检查点就会被触发。

Hooks 插在 Claude Code 的生命周期里。一次工具调用不是模型想完就直接结束，中间有一系列可以拦截和反馈的事件：

- `PreToolUse`：工具执行前，可以检查、修改或阻止这次调用；
- `PostToolUse`：工具成功后，可以格式化、检查并把结果喂回上下文；
- `PostToolUseFailure`：工具失败后记录错误或补充诊断；
- `Stop`：Claude 准备收工时，检查任务是否真的完成；
- `SubagentStart`、`SubagentStop`：子 Agent 启动和结束时追加约束或验收结果。

<!-- drawio源文件: ./drawio/claude_code_extensions_02_hooks_loop.drawio -->

![Hooks在Claude Code工具调用前后和结束验收时插入控制点并将失败证据反馈给Agent](https://file1.kamacoder.com/i/web/20260713152556_claude_code_extensions_02_hooks_loop_compressed.png?v=20261005)

这张图回答的是：Hooks 到底插在 Agent 循环哪里。它不是循环外的一份说明，而是卡在“准备行动、行动完成、准备结束”这些节点上的闸门；不满足条件，就把错误或证据送回 Claude，让它继续修。

一个最小的编辑后检查，大致长这样：

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "pnpm lint"
          }
        ]
      }
    ]
  }
}
```

现在的 Hooks 也不只会跑 shell。它还可以调用 HTTP、MCP 工具，或者让一次 prompt、一个临时 Agent 做判断。但无论执行形式怎么变，它的核心都没有变：**由生命周期事件触发，而不是等 Claude 想起来。**

不过别把 Hooks 神化。

Hooks 适合“每次命中这个事件都要做”的动作，例如格式化、危险命令拦截、审计记录。它不适合承载一整套需要理解业务、根据现场调整步骤的发布手册。

**确定性动作交给 Hook，需要推理的流程不要硬塞进 Hook。**

## 第三阶段：用 Skills，把专项知识从常驻上下文里搬出去

项目继续往前走，你会遇到另一类东西：

- 发版时才需要的检查清单；
- 排查线上问题时才需要的 Runbook；
- 评审支付代码时才需要的安全规则；
- 调某个内部 API 时才需要的字段说明和脚本。

这些内容很重要，但不是每个任务都要看。

塞进 `CLAUDE.md`，上下文会越来越胖；做成 Hook，又没办法表达“先根据现象判断，再选择不同排查路径”。这时候该用 Skill。

Skill 是一套**按需加载的知识和工作流**。Claude Code 启动时通常只看到它的名称和描述，判断当前任务相关时，才加载完整内容：

```text
.claude/skills/payment-review/
├── SKILL.md
├── references/
│   ├── risk-rules.md
│   └── database-schema.md
├── scripts/
│   └── check-money-unit.sh
└── assets/
    └── review-template.md
```

这里最重要的不是文件夹长什么样，而是**上下文加载方式变了**。

`CLAUDE.md` 是“每次都带上”；Skill 是“这次用到才展开”。`SKILL.md` 还可以继续指向参考资料、脚本和模板，让 Claude 用到哪一块再读哪一块。

这就是渐进式披露。关于 Skill 怎么分类、怎么写 description、怎么积累 Gotchas，我们在 [Claude Skills 实战](./claude_skills.md) 已经详细讲过，这里不重复。

现在可以把 Hook 和 Skill 分清了：

- Hook 由事件触发，强调**这件事必须发生**；
- Skill 由用户调用或任务意图触发，强调**这类事应该怎么做**；
- Hook 更像闸门，Skill 更像专项作业手册；
- Hook 追求稳定重复，Skill 允许 Claude 根据现场推理。

例如，`PostToolUse` Hook 可以在支付代码被修改后运行金额单位扫描；`payment-review` Skill 则告诉 Claude 发现问题以后，应该如何判断风险、补测试和给出修复方案。

## 第四阶段：拆 Subagents，把“一个 Claude 全包”变成角色分工

有了 Skills，Claude 已经能按需获得专项能力。但复杂任务还有一个更隐蔽的问题：**所有工作仍然挤在同一个上下文里。**

主 Agent 先读几十个文件定位问题，又改代码、跑测试，最后还让它审查自己的实现。上下文越来越脏，早期约束被压缩；更麻烦的是，让同一个 Agent 给自己的答案挑错，天然容易自我认可。

Subagents 解决的不是“再加一份提示词”，而是**再开一个独立的执行上下文**。

一个自定义子 Agent 可以这样定义：

```md
---
name: payment-security-reviewer
description: 支付模块改动完成后，独立检查金额、权限和幂等风险
tools: Read, Glob, Grep, Bash
model: sonnet
skills:
  - payment-review
---

只做安全评审，不修改业务代码。结论必须包含风险位置、触发条件和验证证据。
```

它拥有自己的系统提示、工具范围、模型、权限、Skills 和上下文窗口。主 Agent 只需要把任务委派出去，最后接收摘要，不必把子 Agent 读过的几十个文件和全部命令输出都塞回主线程。

<!-- drawio源文件: ./drawio/claude_code_extensions_03_subagent_isolation.drawio -->

![主Agent将测试安全和性能任务委派给独立上下文的Subagents并只接收带证据的结论](https://file1.kamacoder.com/i/web/20260713152556_claude_code_extensions_03_subagent_isolation_compressed.png?v=20261005)

这张图回答的是：独立上下文到底省掉了什么。测试、安全、性能三个 Agent 各自在干净窗口里工作，过程噪声留在各自上下文，只有带证据的结论回到主 Agent；执行者和审查者也被结构性地分开。

所以 Skill 和 Subagent 也不是一回事：

- Skill 是可以被复用的“方法”；
- Subagent 是拿着方法独立干活的“角色”；
- Skill 会把内容加载进当前上下文；
- Subagent 在另一个上下文里执行，只把结果带回来。

两者还可以组合。安全审查 Agent 预加载安全 Skill，测试 Agent 预加载测试 Skill，主 Agent 负责拆任务、汇总结论和决定是否继续修改。

这一步往后，就是我们在 [动态工作流](./dynamic_workflows.md) 里讲的更复杂编排了：多个 Agent 并行、对抗验证、生成与筛选。但别急着冲到那一步，两个角色能讲清楚的任务，不要为了显得高级硬开十个 Agent。

## 四层能力放在一起，到底怎么选

现在回到最容易混淆的问题：一条新要求来了，应该写到哪里？

<!-- drawio源文件: ./drawio/claude_code_extensions_04_decision.drawio -->

![根据常驻知识事件触发专项方法和上下文隔离判断一条要求应该放进CLAUDE.md、Hooks、Skills还是Subagents](https://file1.kamacoder.com/i/web/20260713152556_claude_code_extensions_04_decision_compressed.png?v=20261005)

这张图回答的是：如何根据“是否每次都要知道、是否必须自动发生、是否需要专项推理、是否需要隔离上下文”逐层判断。它不是让你四选一，而是帮你找到一条要求最主要的归属层。

可以记住四个问题：

| 判断问题 | 应该放哪里 | 典型例子 |
|---|---|---|
| Claude 是否每次都应该知道？ | `CLAUDE.md` | 技术栈、目录、构建命令、禁区 |
| 是否命中事件就必须执行？ | Hooks | 格式化、阻止危险命令、记录审计 |
| 是否只在某类任务中需要一套方法？ | Skills | 发布、评审、排障、数据分析 |
| 是否需要独立上下文或独立角色？ | Subagents | 大范围探索、安全复核、并行测试 |

还是支付模块的例子：

1. `CLAUDE.md` 写清金额单位、项目命令和数据库禁区；
2. Hook 在文件修改后跑静态检查，在危险操作前做拦截；
3. Skill 保存支付评审方法、风险规则和验证脚本；
4. Subagents 分别检查安全、测试和性能，主 Agent 汇总证据。

你会发现，四层能力不是互相竞争，而是分别控制 Agent 的四个位置：

**上下文里放什么，生命周期中卡什么，任务需要时加载什么，复杂工作交给谁。**

## 最容易踩的四个坑

**第一，把所有东西都塞进 CLAUDE.md。**

结果不是 Claude 懂得更多，而是每轮都背着一仓库资料。长期稳定规则留下，专项知识移到 Skills。

**第二，把“禁止”只写成自然语言。**

“不要改 `.env`”如果是安全红线，就别只靠模型听话。能用权限和 `PreToolUse` Hook 拦截的，直接做成硬边界。

**第三，把复杂判断写成一坨 shell Hook。**

Hook 是触发器，不是所有业务逻辑的垃圾桶。需要读资料、比较方案、根据现场调整步骤，就交给 Skill 或专项 Agent。

**第四，任务一大就疯狂开 Subagents。**

子 Agent 会消耗额外 token，也有任务描述丢信息、结果汇总失真的成本。只有过程噪声很大、角色确实应该隔离，或者工作真的能并行时再拆。

## 写在最后

很多人配置 Claude Code，一上来就抄别人的全家桶：几百行 `CLAUDE.md`、几十个 Hooks、十几个 Skills，再配一队 Subagents。

**太重了，也太早了。**

同一条规则说了两次，写进 `CLAUDE.md`；某个动作必须发生，做成 Hook；某套流程反复复制，养成 Skill；某项工作需要独立探索，再交给 Subagent。

别按功能清单搭系统。**看问题卡在哪一层，就补哪一层。**

## 参考链接

- Claude Code 官方文档（扩展能力总览）：https://code.claude.com/docs/en/features-overview
- Claude Code 官方文档（Memory 与 CLAUDE.md）：https://code.claude.com/docs/en/memory
- Claude Code 官方文档（Hooks 指南）：https://code.claude.com/docs/en/hooks-guide
- Claude Code 官方文档（Skills）：https://code.claude.com/docs/en/slash-commands
- Claude Code 官方文档（Subagents）：https://code.claude.com/docs/en/sub-agents
