---
title: DeepSeek开源Super-agent架构原型：Router与Worker、事件驱动、上下文管理、BYOM和Watchdog到底怎么工作
description: DeepSeek开源dsh-superagent-prototype。本文从多任务调度切入，拆解Super-agent的Router/Worker双层架构、Trigger与Thread路由、授权和主动接单、上下文召回、异步交互、云端执行设想与Watchdog，并区分设计文章和公开研究原型的实际能力。
keywords: [DeepSeek Super-agent, DeepSeek超级Agent, dsh-superagent-prototype, dsh-superagent, DeepSeek Harness, DSH, Router Worker架构, Agent架构, Agent Harness, Trigger, Thread, Watchdog, BYOM, 主动式Agent, 持续运行Agent, 多Agent, GitHub Agent, 飞书Agent, AI编程, Agent权限控制, Agent任务调度, 上下文管理]
tags: [DeepSeek, Super-agent, AI Agent, Coding Agent, Agent工程, 大模型动态]
---

# DeepSeek开源Super-agent：Agent会干活以后，谁来管这一堆Agent？

前面写 [DeepSeek Harness使用教程](./deepseek-harness-guide.md) 时，我们讲的是单个 Agent 怎样读仓库、跑命令、改代码。

可当一个团队同时有 Bug、需求和代码评审在推进，问题就变了：**谁来接住这些任务，谁来记住它们各自的进度，什么时候才需要叫人？**

DeepSeek 新开源的 dsh-superagent-prototype，正是在探索这层调度系统。

Github地址：https://github.com/deepseek-ai/dsh-superagent-prototype

![](https://file1.kamacoder.com/i/web/20261102155926.jpg?v=20261005)


## 一、为什么需要 Super-agent？

你可以同时开好几个 Coding Agent 会话：一个修测试，一个改接口，一个等 CI。

每个会话单独看都能干活，但人要在窗口之间切换、找回上下文、判断谁在等谁。

如果把所有事塞进同一个会话，长命令又会阻塞后续消息，几个任务的细节也会混在一起。**

模型能并行完成的工作，最后卡在人的注意力和会话管理上。**

![多Agent会话注意力瓶颈卡通图](https://file1.kamacoder.com/i/web/20261102112948.jpg?v=20261005)

## 二、Super-agent 能替团队做什么？

Super-agent 想替人接管的是会话和任务的调度。

团队成员在熟悉的工作入口提出问题，系统把它转成可跟踪的工作项，后台用独立会话推进，最后回到原讨论交付结果。

公开的 [dsh-superagent-prototype](https://github.com/deepseek-ai/dsh-superagent-prototype) 已经把这个思路用于**一个 GitHub 仓库**：监听 Issue 和 PR，可选接入飞书，受理后交出答复或 Pull Request。**

这是研究原型，不是正式产品**；它与设计文章中的内部版本并不完全相同。

![Super-agent研究原型边界卡通图](https://file1.kamacoder.com/i/web/20261102110232.jpg?v=20261005)

从使用者的角度，最直接的变化是不用盯着一排 Agent 窗口。

请求进入系统后，你主要在需要补充决定、审核改动或处理失败时介入；平时的读代码、跑测试和进度更新由后台完成。

原型的面板也先展示“等待人处理”的事项，回复尽量留在发起任务的讨论里。**

省下的不是一次工具调用，而是人反复找窗口、追进度、接上下文的时间。**

![Agent减少消息打扰卡通图](https://file1.kamacoder.com/i/web/20261102104333.jpg?v=20261005)

## 三、双层架构：一个入口，多个执行者

### Router 负责分派

Super-agent 的核心分工是 Router 和 Worker。

Router 像值班总台，负责读新事件、判断任务归属、维护跨任务的全局视野，并统一对外说话；它不需要亲自改代码。

原型里主会话接住候选事件，执行接单、附加、停止或忽略等决定。**

总台越少卷入具体代码，越有余力处理下一条消息。**

设计文章还讨论了把短时间涌入的事件合批处理，以减少每个决定分摊的推理成本。

![Router与Worker分工卡通图](https://file1.kamacoder.com/i/web/20261102110237.jpg?v=20261005)

### Worker 负责交付

一个被接下的工作会变成 Ticket，再分配给自己的 Worker 会话和目录。

准备命令可以为它建立 git worktree，让修测试与改接口落在不同工作区；并发数由 Worker slot 控制。

这样 Worker 可以深入代码、跑测试、开 PR，而 Router 只看任务级结果。**

隔离的是任务上下文和工作目录，不是说它们天然拥有操作系统级沙箱。**

![Agent独立工作区卡通图](https://file1.kamacoder.com/i/web/20261102110240.jpg?v=20261005)

## 四、事件驱动：消息怎样变成工作

### 先接住环境事件

传统聊天 Agent 等你在输入框敲下一句；这套系统把 GitHub Issue、PR 评论、CI 结果以及可选的飞书消息看作事件入口。

Source 收到事件后把它记为 Trigger，并归入相应的 Thread。

如此一来，Agent 面对的是持续变化的工作现场，而不是一段孤立的提示词。**

但能看见事件，只说明有了线索，还不代表应该立刻动手。**

![Super-agent事件接入卡通图](https://file1.kamacoder.com/i/web/20261102110233.jpg?v=20261005)

### 先判断有没有权做

原型会先检查来源、接收范围、发送者和规则，候选任务才进入评估。

接单后，Ticket 带着 Mandate，记下授权依据与工作范围。

比如 Issue 评论里有人追加“顺手改隔壁仓库”，这句话不能直接扩大原任务的权限；Worker 要把超范围请求交回总台重新判断。**

把权限写成宿主程序能检查的状态，比相信模型会自行克制更可靠。**

![Agent任务授权边界卡通图](https://file1.kamacoder.com/i/web/20261102110235.jpg?v=20261005)

### 后续消息直达原任务

Thread 的价值在后续消息到来时才明显。

同一条 Issue 或 PR 的评论，可以直接送给订阅它的 Ticket；Worker 在原任务范围内处理追问，必要时把新请求交回 Router。

这样就不用每来一条评论都重新问模型“这是谁的事”。

设计文章的模拟实验里，去掉这种直通后，Router 的决定次数增加约 30%，缓存读取 token 增加约 37%；这个数字说明的是**调度开销**，并非代码质量提高了 37%。

![任务讨论直达Worker卡通图](https://file1.kamacoder.com/i/web/20261102112950.jpg?v=20261005)

### 旧信息按需找回

一个长期运行的 Router 不可能把几个月的消息全放进模型窗口。

设计文章提出把上下文当缓存：高频使用的信息直接放进当前窗口，较长的原文留在外部，只带摘要和引用，需要时再按 ID 找回相关 Trigger、Thread 或 Worker。

公开原型也能按 Thread 读取分页消息和相关 Ticket，但它的存储与召回实现不等于设计文章描述的完整长期记忆。**

关键原则是先给模型足够判断的线索，同时保留继续追查的路。**

![Agent上下文整理卡通图](https://file1.kamacoder.com/i/web/20261102112952.jpg?v=20261005)

### 主动工作先试运行

没人明确点名时，某条 CI 失败或群消息也可能符合团队写好的规则。

原型把这类事件列为“推断候选”，有 off、observe、auto 三种接纳模式；默认 observe **只记录本来会接什么，不创建任务**。

团队先看一段时间的判断记录，再决定是否打开 auto。

这个顺序很重要：主动性不是让 Agent 对每条消息都抢答，而是让它在可审查的规则内行动。

![Agent观察模式卡通图](https://file1.kamacoder.com/i/web/20261102110238.jpg?v=20261005)

### 提前试修仍是设计方向

比自动接单再往前一步，是后台先做低成本尝试：例如看到问题反馈，先静默复现，确实找到小范围修法再把结果交给团队；试不出来就不发一条空洞的“我正在处理”。

这是 DeepSeek 设计文章介绍的内部实践和架构方向。**

不能因为原型有 observe 和自动接单，就推断它已经完整实现这种投机执行策略。**

两者都追求少打扰人，但机制不同。

![Agent后台试修卡通图](https://file1.kamacoder.com/i/web/20261102112954.png?v=20261005)

## 五、边干活边接新指令

一个测试命令跑十分钟时，Agent 如果必须等命令结束才能读下一条消息，接任务的总台也会跟着卡住。

设计思路是把长操作转成后台作业：先返回“已启动”，完成结果稍后再送入会话。

公开原型确实把前台 shell 命令在短时间后移到后台作业，并在有后续结果时唤醒 Worker；DeepSeek Harness 更通用的异步任务机制，设计文章也说明当时尚未默认开启。**

这里要看的是能否继续接新消息，而不是命令有没有跑完。**

![Agent前后台并行交互卡通图](https://file1.kamacoder.com/i/web/20261102112957.jpg?v=20261005)

## 六、长期运行：状态、环境和巡检

### 云端与本地环境拆开

设计文章想把三件事拆开：云端 Agent Loop 负责推进任务，数据库保存会话状态，用户电脑或其他设备只提供实际执行环境。

这样电脑暂时离线时，Agent 的“脑子”还能在线，等设备回来再跑必须在本机执行的命令；这种连接自有机器的设想叫 BYOM。**

公开原型并没有随仓库交付这套云端 Loop 加本地 daemon 的架构。**

它的部署文档写的是在一台机器上运行专用 DSH profile 和 tmux 服务。

![Super-agent云端与本地环境卡通图](https://file1.kamacoder.com/i/web/20261102112958.jpg?v=20261005)

### 重启后从任务账本恢复

虽然没有实现上面的 BYOM，公开原型仍有值得读的持久化设计：Trigger、Ticket、Assignment、订阅和待发送消息存进 SQLite；重启后，忙碌中的任务带着恢复提示重新启动，Worker 先核对目录、Git 历史和 GitHub 状态，避免重复执行外部动作。

暂时的网络与模型故障按规则重试，无法继续的任务停在需要人处理的状态。**

“能恢复”首先要有可信的任务账本。**

![Agent重启恢复卡通图](https://file1.kamacoder.com/i/web/20261102110242.jpg?v=20261005)

### 还要发现无效空转

设计文章里的外部 Watchdog 像夜班巡检：不仅看服务有没有报错，也看 Worker 是否长期空转、是否偏离目标，然后尝试纠偏。

公开原型有健康状态、失败重试、恢复流程和人工面板，但它的架构文档没有把独立的 Watchdog Session 列为已实现模块。**

进程活着与任务真的向前，是两种不同的健康指标。**

这也是持续运行 Agent 比普通脚本难的地方。

![Agent巡检与纠偏卡通图](https://file1.kamacoder.com/i/web/20261102113000.jpg?v=20261005)

## 写在最后

原型部署文档说得很明确：Router 和 Worker 预设以运行它的系统用户执行，并使用 danger-full-access。

给 GitHub CLI 换成 Agent 专用凭据，并不能隔离本机进程；Worker 仍可能访问该系统用户的主目录。

所以试跑时应使用独立系统账号和专用 GitHub 账号，先把能碰到的仓库和凭据缩到最小。**

接入的事件越多、运行时间越长，权限错误的影响就越大。**

![Agent专用账号隔离卡通图](https://file1.kamacoder.com/i/web/20261102110244.jpg?v=20261005)

先从一个练习仓库开始，保持默认 observe，记录它哪些任务想接、哪些该忽略。

再沿着 **Source → Trigger → Eval → Ticket → Worker → Channel** 追一条请求，看看授权如何留下、后续评论如何回到原任务、失败又怎样恢复。

最后再读设计文章里的 BYOM 和 Watchdog，问一句：如果要把它们补进公开原型，现有的状态和权限边界要怎样调整？**

把“想法”和“代码已经做到”分开，才是真正读懂 Super-agent。**

![Super-agent练习部署卡通图](https://file1.kamacoder.com/i/web/20261102110246.jpg?v=20261005)

## 资料来源

* DeepSeek Super-agent 设计文章：https://x.com/DeepSeekHarness/article/2116857257245987236
* DeepSeek dsh-superagent-prototype README：https://github.com/deepseek-ai/dsh-superagent-prototype
* DeepSeek 原型设计原则：https://github.com/deepseek-ai/dsh-superagent-prototype/blob/main/docs/000-philosophy.md
* DeepSeek 领域模型：https://github.com/deepseek-ai/dsh-superagent-prototype/blob/main/docs/001-domain.md
* DeepSeek 授权机制：https://github.com/deepseek-ai/dsh-superagent-prototype/blob/main/docs/002-authorization.md
* DeepSeek 执行与恢复：https://github.com/deepseek-ai/dsh-superagent-prototype/blob/main/docs/003-execution.md
* DeepSeek 部署与权限说明：https://github.com/deepseek-ai/dsh-superagent-prototype/blob/main/docs/006-deployment.md
* DeepSeek 原型实现架构：https://github.com/deepseek-ai/dsh-superagent-prototype/blob/main/docs/007-architecture.md
