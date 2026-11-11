---
title: Claude Code与AI原生SDLC实战：从intent.md、spec.md到Agent测试、审查、部署和运维闭环
description: "深度解读Anthropic《The AI-Native SDLC Playbook》，讲清AI写代码变快后为什么计划、测试、部署和治理成为新瓶颈，以及团队如何用intent.md、spec.md、plan.md、CLAUDE.md、Skills、Hooks和持续评估重构软件开发生命周期。"
keywords: [AI原生SDLC, AI-Native SDLC, Agentic SDLC, Claude Code, Anthropic, AI编程, Agentic Coding, 软件开发生命周期, intent.md, spec.md, plan.md, CLAUDE.md, Claude Skills, Claude Hooks, Plan Mode, AI代码审查, 持续评估, CI/CD, DevOps, 人机协作]
tags: [Claude Code, AI编程, Agentic SDLC, 软件工程, Anthropic]
---

# Claude Code写代码快到飞起，为什么你的研发流程反而更堵了？

Anthropic 发布了一篇博客《The AI-Native SDLC Playbook》。

![](https://file1.kamacoder.com/i/web/20260915111138-quc5m0.png?v=20261005)

翻译过来就是「AI 原生软件开发生命周期手册」。

这份手册讨论的不是怎么让 Claude Code 再多写几行代码，而是当 AI 已经能快速写代码以后，整个研发流程应该怎么重做。

![代码生产与流程拥堵](https://file1.kamacoder.com/i/web/20260915101818.jpg?v=20261005)

现在很多团队已经用 AI 把几天的编码工作压缩到了几小时，但需求澄清、评审排队、安全检查和上线审批依然按照过去的速度运行。

结果就是代码写得越快，PR 堆得越多，安全团队和测试团队越容易成为新的瓶颈。

**只给编码阶段装上 Agent，等于给高速列车接了一段泥巴路，产出越快，堵得越明显。**

前面我们写过[从Prompt走向Agent Loop](./claude_code_loop.md)和[Loop Engineering实战](./loop_engineering_guide.md)，讲的是单个任务怎么跑稳，而这份手册解决的是整个软件开发生命周期怎么跟上 Agent 的速度。

## AI原生SDLC的核心，不是把六个阶段都交给AI

![版本化工件接力](https://file1.kamacoder.com/i/web/20260915101820.jpg?v=20261005)

Anthropic 给出的关键不是一张“AI 替代岗位”的表，而是一条可提交、可追溯的工件链：`intent.md → spec.md → plan.md → 代码与测试 → PR与审查结果 → 事故记录`。

每一个阶段先读取上一棒留下的工件，再把自己的结果提交到版本控制，被接受的工件会自动触发下一阶段。

这样 Git 历史就能记录谁提出需求、Agent 做了什么、谁最终批准，**人也从搬运信息转向守住真正需要判断的门。**

## 第一阶段Plan：先保存原始意图

![业务意图沉淀](https://file1.kamacoder.com/i/web/20260915101821.jpg?v=20261005)

传统需求会经过工单、用户故事、估点和多轮会议，传到工程师手里时，常常已经离提出者的原话很远。

AI 原生的做法是让提出者直接和 Claude 把问题、用户、约束、目标与未决问题聊清楚，生成`intent.md`，再由产品负责人纠正和确认。

这里最值钱的不是 Markdown 格式，而是**让人和 Agent 第一次共享同一个可执行的事实源。**

## 第二阶段Design：把规则前移，不要等评审找茬

![设计阶段应用规则](https://file1.kamacoder.com/i/web/20260915101824.jpg?v=20261005)

被接受的`intent.md`进入设计阶段后，Claude 会在品牌、安全、合规和 UX 等 Skills 约束下生成`spec.md`。

过去很多问题要到安全评审或设计验收时才暴露，那时代码可能已经写了一半，而现在这些规则会在规格形成时直接参与决策。

**Skill 负责让政策更容易被遵守，但绝对不能违反的要求，还要交给确定性的 Hook 和流水线检查。**

## 第三阶段Build：先审plan.md，再开自动模式

![计划评审与Agent执行](https://file1.kamacoder.com/i/web/20260915101825.jpg?v=20261005)

工程师不应该拿到`spec.md`就让 Agent 直接改代码，而要先在 Plan Mode 里让 Claude 列出修改文件、实施顺序、风险和验证方式。

计划应该具体到一个没看过聊天记录的人也能照着`plan.md`完成任务，随后 Claude 再依据[CLAUDE.md](./claude_md.md)、[Skills](./claude_skills.md)和 Hooks 执行。

计划越扎实，普通改动越适合自动执行和多 worktree 并行，但并行上限永远是人还能不能认真审完这些工件。

## 第四阶段Test：Agent必须先拿证据，再说完成

![Agent自我验证闭环](https://file1.kamacoder.com/i/web/20260915101827.jpg?v=20261005)

Anthropic 在测试阶段最强调的一件事，就是永远给 Claude 一条能自行运行的反馈回路。

测试、构建、Lint、浏览器操作和截图对比都应该由 Agent 在交付前反复执行，修 Bug 时还要防止它偷偷改弱测试。

**没有可重复的验证命令，AI 产出越多，人工验收债务越大。**

## 第五阶段Deploy：Agent走到生产门口就停

![生产部署人工门禁](https://file1.kamacoder.com/i/web/20260915101828.jpg?v=20261005)

部署阶段可以让 Claude 执行多轮 PR 审查、归类问题、处理评论和修复失败检查，并在沙箱里完成所有发布准备。

但写代码的 Agent 不能批准自己的 PR，生产发布也必须由指定责任人授权。

**治理不是多开一个周会，而是用`REVIEW.md`、分支保护和 Hooks，把控制要求变成 Agent 行动时就会生效的代码。**

## 第六阶段Maintain：让生产反馈自动回到起点

![生产告警回流闭环](https://file1.kamacoder.com/i/web/20260915101830.jpg?v=20261005)

闭环真正形成于维护阶段，因为确定性脚本会持续观察测试失败率、5xx 比例和交付周期等指标，越过控制带后再调用 Claude 诊断。

Agent 会把异常证据、受影响系统和建议结果写成新的`intent.md`，重新经过计划、设计、构建、测试和审查，而不是在凌晨绕过流程直接修改生产。

**检测要保持确定性，诊断可以使用模型，行动必须受到权限和门禁约束。**

## Carl的判断：别先追求全自动，先修最堵的一段

![人守住目标风险验收](https://file1.kamacoder.com/i/web/20260915101832.jpg?v=20261005)

这份 Playbook 最容易被误读成“买了 Claude Enterprise，就能让 Agent 包办整个研发流程”。

真正可落地的顺序，是先挑最堵的一段，把完成标准变成可执行的验证命令，再把反复出现的错误写进`CLAUDE.md`，把组织政策做成 Skill，把不可妥协的边界做成 Hook。

不要一上来替换 Jira、Figma 和现有审计系统，只要先为每类工件明确唯一事实源，并在另一边保存链接或 Commit SHA。

**AI 原生 SDLC 不是无人研发，而是让人从盯每一行代码，升级为设计目标、风险、证据和责任边界。**

## 参考链接

- Anthropic官方博客（The AI-Native SDLC Playbook）：https://claude.com/blog/the-ai-native-sdlc-playbook
