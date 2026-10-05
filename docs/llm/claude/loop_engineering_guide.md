---
title: Loop Engineering 实战：14 步路线图，从判断要不要做到上线后守住
description: "Loop Engineering 不是免费午餐，烧 token、要花时间搭、出问题还得 debug 一个你没亲眼看它跑的系统。本文综合 Anthropic 工程文档、Addy Osmani 关于 loop 工程的长文、以及实测数据研究，总结出 14 步完整路线图：先用四个问题判断你到底需不需要循环，再掌握 Automations、Worktrees、Skills、Connectors、Sub-agents 五块核心积木，最后搭一个最小的、不会坑你的循环。还要警惕假装干完了、理解债务、认知投降三种翻车方式，以及无人值守循环的安全红线。适合 Claude Code、AI 编程、Agent 开发和大模型应用上生产学习。"
keywords: [Loop Engineering, Claude Code Loop, Agent Loop, Automations, Worktrees, Skills, Connectors, Sub-agents, AI 编程, Agent 开发, Claude Code, Anthropic, 工作流编排, CI 自动化, Agent 验证, 大模型应用开发, AI Coding, loop 实战, Agent 安全, 循环工程]
tags: [Claude Code, AI编程, Agent, 大模型入门]
---

# Loop Engineering 实战：14 步路线图，从判断要不要做到上线后守住

录友们好，继续聊 Claude Code。

上一篇 [动态工作流详解](./dynamic_workflows.md) 我们讲了 Claude 怎么自己现写 harness、把任务拆给一队 Claude 去干。这一篇往实战方向走：**你自己搭一套 Loop Engineering，从零到上线，完整的 14 步路线图。**

这份路线图综合自 Anthropic 的工程文档、Addy Osmani 那篇全网 220w 人看过的 loop 工程长文，以及最近几篇带实测数据的研究。

## Loop 不是免费的，先问四个问题

Loop 很诱人，但它不是免费午餐。**它烧 token、要花时间搭、出了问题你还得去 debug 一个你没亲眼看它跑的系统。**

动手之前，先把这四个问题想清楚：

<!-- drawio源文件: ./drawio/loop_engineering_guide_01_decision.drawio -->

![判断是否需要 Loop](https://file1.kamacoder.com/i/web/20260708113429_loop_decision.jpg?v=20261005)

**任务是重复的吗？** Loop 的搭建成本要靠多次运行摊回来。一次性的活儿，写个好 prompt 更快。

**有自动检查机制吗？** 测试、类型检查、linter，随便哪个都行。**没有自动检查，你就得自己逐行读 diff**，loop 就并没有帮你节省时间。

**Token 预算够吗？** Loop 会反复读上下文、重试、试探，不管有没有产出都在烧 token。

**Agent 能跑自己写的代码吗？** 需要有日志、能复现、看得到哪里崩。

还有个附加题，**比上面四个都重要**：**你打算 review 它产出的代码吗？** 不打算，就别建 loop。

**谁适合上手：** 有强测试套件的团队，干 CI 失败分类、依赖升级、lint-and-fix 这类任务。

**谁不适合：** 消费级套餐的个人开发者（token 预算不够烧）、测试覆盖不够的代码库（没闸门）、瓶颈在 review 而不在打字速度的团队。

## Loop 的五个核心构件

@0xCodez 把 loop 拆成五个构件，好在每个都能单独用、单独试。

<!-- drawio源文件: ./drawio/loop_engineering_guide_02_components.drawio -->

![Loop 的五个核心构件](https://file1.kamacoder.com/i/web/20260708113435_loop_components.jpg?v=20261005)

**Automations——loop 的心跳。** 按节奏触发，跑完一轮，停下。**关键是停止条件要写死，别让它无限跑。**

**Worktrees——并行不打架。** 多个 Agent 同时干活，最怕改同一个文件。Git worktree 给每个 Agent 一份独立工作区，互不干扰。但 worktree 有 200-500ms 的 setup 开销，只在真需要并行修改文件时才用。

**Skills——把背景写下来。** 项目用什么框架、有什么约定、踩过什么坑，写成一个 skill 存着，Agent 每轮直接读。这块我们在 [Claude Skills 实战](./claude_skills.md) 里讲得很透了。

**Connectors——连上真实工具链。** 通过 MCP 接上 GitHub、Linear/Jira、Slack、Sentry，loop 才算真正接入你的工作流。关于 MCP 协议本身，可以看 [MCP 协议详解](../app/mcp_protocol.md)。

**Sub-agents——写的和验的分开。** 写代码的模型给自己打分太宽容。换一个带不同指令的第二个 Agent 来验收，能抓到第一个自我说服过去的问题。

## 14 步完整路线图

<!-- drawio源文件: ./drawio/loop_engineering_guide_03_roadmap.drawio -->

![14 步完整路线图](https://file1.kamacoder.com/i/web/20260708113442_loop_roadmap.jpg?v=20261005)

### 第一段：先想清楚要不要做（5 步）

1. **确认这活是重复的** - 一次性任务用好 Prompt 更划算
2. **确认有自动检查机制** - 测试、类型检查、linter，至少一个
3. **确认 token 预算够** - loop 不产出也照样烧钱
4. **确认 Agent 能自己验证** - 有日志、能复现、看得到哪崩了
5. **确认你真打算 review** - 不打算，就别建

### 第二段：搭一个最小能跑的 loop（8 步）

6. **先让一次手动运行稳定下来** - 顺序别跳，先手动跑一遍确认每一步都通
7. **把项目背景沉淀成一个 Skill** - 省得每轮从零解释
8. **加一个状态文件** - 记下做完了什么、下一步干啥，这是 loop 的记忆
9. **设一道硬闸门** - 测试/构建过不了就自动拒，这是你能放心走开的唯一保障
10. **配一个 Automation** - 按节奏触发，用停止条件控制什么时候停
11. **多 Agent 并行就上 Worktree** - 别让它们改同一个文件打架
12. **接上 Connectors** - 让 loop 能开 PR、更新 ticket、发 Slack
13. **拆出 Sub-agents** - 写代码的和验收的分开，验收的那个要专挑刺

### 第三段：上线之后守住（1 步，但最难）

14. **盯住每个被接受的改动成本，定期复审权限、读 diff、别让 loop 碰架构**

**盯一个指标：每个被接受的改动的成本。** 如果接受率低于 50%，这 loop 就在亏本。

**定期复审权限。** 今天加一个写权限，明天再加一个，每 30 天复审一次，砍掉不需要的。

**读 diff。** 别因为「loop 产出的」就不读了。抽查闸门，确保它真的在按你的规则工作。

**别让 loop 碰架构。** loop 能干的是重复劳动、机械校验、低风险的小改动。架构决策、重构、核心逻辑，必须人来做。

## 上线后的三种翻车方式

<!-- drawio源文件: ./drawio/loop_engineering_guide_04_pitfalls.drawio -->

![Loop 的三种翻车方式](https://file1.kamacoder.com/i/web/20260708113449_loop_pitfalls.jpg?v=20261005)

**假装干完了。** 工程师 Geoffrey Huntley 管这叫 Ralph Wiggum 循环：Agent 提前发「完成」信号，活干一半就退。原因只有一个：**没有硬闸门。** 解法：回到第 9 步，设一道硬闸门。

**理解债务。** loop 越快交付你没写过的代码，**「仓库里有什么」和「你理解什么」的差距就越大。** 有一天，你得 debug 一个团队里没人读过的系统。解法：读 diff，抽查。

**认知投降。** 你慢慢不再自己判断，loop 返回啥就收啥。**这是最隐蔽、也最致命的一种翻车。** 解法：守住第 14 步，盯指标、读 diff、不让 loop 碰架构。

## 安全红线

**无人值守的 loop，就是无人值守的攻击面。**

- **生成代码未审就上线** - 闸门里得加 SAST、依赖审计、密钥扫描
- **Skill 是注入入口** - 社区 17022 个 skill 里有 520 个会泄露凭证，自动安装前先读源码
- **凭证泄露进日志** - 生产 loop 关掉 verbose 日志
- **权限蔓延** - 今天加一个写权限，明天再加一个，每 30 天复审一次

**无人值守不等于无人负责。** loop 出了安全问题，背锅的还是你。

## 写在最后

两年来，与编码 Agent 协作的杠杆一直在提示词上。而现在，**工作流成了真正的护城河。**

loop engineering 不是银弹，它有成本、有门槛、有风险。但对于那些重复、能机器校验、有强测试套件的任务，它能把「十二个小时的活，干成二十分钟」。

录友们与其纠结理论，不如挑一个你每天都在重复、有测试覆盖、能自动验证的活——CI 失败分类、依赖升级、lint-and-fix——按这 14 步走一遍。

## 参考链接

- Anthropic 工程文档（Managed Agents 与 Dynamic Workflows）：https://claude.com/blog
- Addy Osmani 关于 loop 工程的长文：https://addyosmani.com/blog/
