---
title: GPT-6 Astra正式发布：OpenAI最强AI Agent、Computer Use、API价格，和GPT-5.6 Sol、Claude Fable 5.1怎么选
description: OpenAI于2026年9月3日发布GPT-6 Astra，主打电脑操作、浏览器使用、AI编程、科学研究和长链路Agent。本文结合官方截图，拆解GPT-6 Astra的Terminal-Bench 4.0、OSWorld 2.0、ARC-AGI-3、FrontierMath和网络安全评测，说明1.05M上下文、API价格、异步工具调用、Codex跨上下文记忆、开放范围，以及它和GPT-5.6 Sol、Claude Fable 5.1、Claude Opus 5的真实差距。
keywords: [GPT-6 Astra, GPT6 Astra, GPT-6发布, OpenAI, GPT-6 Pro, ChatGPT, Codex, AI Agent, Computer Use, 电脑操作, 浏览器智能体, AI编程, Terminal-Bench 4.0, OSWorld 2.0, ARC-AGI-3, FrontierMath Tier 4, ExploitBench, 异步工具调用, Mid-turn steering, Codex记忆, 1M上下文, GPT-6 Astra API, GPT-6 Astra价格, GPT-5.6 Sol, Claude Fable 5.1, Claude Opus 5, 大模型评测]
tags: [GPT-6, OpenAI, AI编程, Agent, Computer Use, 大模型发布, 大模型评测]
---

# GPT-6 Astra发布：跑分几乎刷满，但我更在意它开始接管鼠标了

> **想尽快体验 Astra，目前更直接的方式是升级 ChatGPT Pro。** 国内没有海外卡的录友，可以看这篇 [GPT-6 Astra开放 与 充值方法](./gpt-6-pro-rollout.md)，里面整理了 PayAI.plus 的 Pro 100 美元、Pro 200 美元两个充值入口和注意事项。需要提醒的是，Pro 用户现在也是有限灰度，并非充值后立刻出现；Plus 用户想在 Chat 里直接使用 GPT-6 Astra，还需要再等几天。

两个月前写 [GPT-5.6](./gpt-5-6.md) 时，我说 OpenAI 正在把 ChatGPT 从聊天框改造成一个能持续干活的工作台。

现在，GPT-6 Astra 来了。

2026 年 9 月 3 日，OpenAI 正式发布 GPT-6 Astra。官方给它的定位很直接：最聪明、最对齐，也是目前最擅长电脑操作、浏览器使用、软件工程、网络安全和科学工作的 OpenAI 模型。

![GPT-6 Astra发布首屏](https://file1.kamacoder.com/i/web/20260904143405.png?v=20261005)

*截图说明：OpenAI 官方发布页首屏没有堆参数，只放了一个由星光组成的“6”。这次 Astra 不是 GPT-5.6 Sol 的小修小补，而是 GPT-6 的第一款旗舰模型。*

刚看到发布页时，我的第一反应不是“AGI 终于来了”。

我更在意另一件事：**过去的大模型主要在回答问题，Astra 的重点是直接进入软件，把任务做完。**

填表、改 CRM、整理日历、跑 Excel、制作 PPT、调试网站、安装软件，它都想接手。

这条线一旦跑通，影响的就不只是程序员，而是所有坐在电脑前工作的人。

## 先把发布信息说清楚

GPT-6 Astra 的基础规格如下：

| 项目 | GPT-6 Astra |
|---|---|
| 发布时间 | 2026 年 9 月 3 日 |
| API 模型名 | `gpt-6-astra` |
| 上下文窗口 | 1,050,000 Token |
| 最大输出 | 128,000 Token |
| 知识截止日期 | 2026 年 4 月 30 日 |
| 推理档位 | Low、Medium、High、Xhigh、Max |
| 输入价格 | 10 美元 / 百万 Token |
| 输出价格 | 50 美元 / 百万 Token |
| 核心能力 | 推理、编程、电脑操作、研究、文档制作 |

![GPT-6 Astra模型规格](https://file1.kamacoder.com/i/web/20260904143409.png?v=20261005)

*截图说明：OpenAI 开发者文档列出了 Astra 的模型定位、价格、输入模态和上下文规格。它支持文本与图片输入，但不直接支持音频、视频输入。*

开放范围要多说一句。

官方发布页写的是：Astra 先向少量组织灰度，未来几天陆续覆盖 ChatGPT Plus、Pro、Business、Enterprise、OpenAI API 和 AWS。

但产品帮助中心写得更细：**Chat 里的 GPT-6 Pro 首批面向 Pro 100 美元、Pro 200 美元、Business 和 Enterprise，暂不包含 Plus。** 不同产品的 Work、Codex 和 Chat 也可能不同步。

所以录友现在没在模型列表里看到，不一定是账号有问题。灰度期间，以自己账户里的模型选择器和额度提示为准。


## 跑分确实猛，但别只盯着98%和99.9%

OpenAI 在发布摘要里放了三个非常抓眼球的数字：

- FrontierMath Tier 4：97.6%
- ARC-AGI-3：99.9%
- ExploitBench：100%

![GPT-6 Astra官方摘要](https://file1.kamacoder.com/i/web/20260904143406.png?v=20261005)

*截图说明：官方摘要强调 Astra 在数学、抽象推理、漏洞利用、电脑操作和专业工作上的突破，同时注明当前仍是分批开放。*

看到 99.9%，很容易产生一种感觉：题库是不是已经被刷穿了？

数字没错，但脚注不能丢。

ARC-AGI-3 的成绩来自带状态的 Responses API Harness，模型会在交互环境中持续观察、行动和学习，不是把一道题扔进普通聊天框就能复现。OpenAI 也明确说，研究环境、系统提示和可用工具与生产版 ChatGPT 不完全相同。

**这说明 Astra 的 Agent 系统很强，不等于普通问答已经有 99.9% 的正确率。**

我反而更关注 Terminal-Bench Science 0.1。

它让智能体用代码和终端完成科研工作流，包括分析数据、运行模拟和拟合模型。Astra 最高拿到 64.6%，[Claude Fable 5.1](./claude-fable-5-1.md) 是 52.6%，GPT-5.6 Sol 只有 22.4%。

![GPT-6 Astra科学评测](https://file1.kamacoder.com/i/web/20260904143408.png?v=20261005)

*截图说明：横轴是估算的 API 单任务成本，纵轴是科研任务解决率。Astra 的多个推理档位整体位于对比模型上方，但更高分依然伴随更高成本。*

这张图比“数学 98 分”更接近未来的工作方式：**模型不只给答案，而是自己调用终端、处理数据、发现失败，再继续试。**

## 电脑操作，才是这代模型真正拉开差距的地方

在 OSWorld 2.0 里，Astra 得到 72.6%，GPT-5.6 Sol 是 65.7%。

只看 6.9 个百分点，好像还没到换代级别。

但 Astra 完成一项任务的模拟时间大约是 40 分钟，Sol 约为 75 分钟。**正确率更高，同时每项任务少花约 47% 的时间。** 更新后的 Codex Harness 在 Mind2Web 上又把任务速度推到了原体验的 1.9 倍。

这对电脑 Agent 太关键了。

一个模型操作网页成功率 70%，但点一次、等一次、看一次，半小时还没填完表，用户照样会关掉它。Computer Use 的竞争不只是“会不会点”，还是“能不能在人类失去耐心前点完”。

其他几个数据也很明显：

| 评测 | GPT-6 Astra | GPT-5.6 Sol | Claude Fable 5.1 | Claude Opus 5 |
|---|---:|---:|---:|---:|
| Agents' Last Exam | 59.3% | 53.6% | — | 55.5% |
| AutomationBench | 41.4% | 18.1% | 31.4% | 26.9% |
| ScreenSpot-Pro | 92.7% | 76.9% | — | — |
| OSWorld 2.0 | 72.6% | 65.7% | — | 70.2% |

AutomationBench 从 18.1% 涨到 41.4%，是我认为这次最有含金量的提升之一。它测的是跨应用完成真实办公流程，不是认一个按钮。

当然，41.4% 也说明现实：**最强电脑 Agent 依然会在一半以上的复杂任务里失败。** 现在就让它无人监管地改生产数据，还太早。

## 编程很强，但没有把Claude全面打趴下

GPT-6 Astra 在 Terminal-Bench 4.0 上从 Sol 的 37.3% 提升到 57.9%，也高于 Fable 5.1 的 55.8% 和 [Claude Opus 5](./claude-opus-5.md) 的 52.3%。

数据库迁移内部评测也从 42.7% 涨到 63.9%。这类任务要求模型读懂旧结构、生成迁移方案、执行并验证，很符合 Astra 的长链路优势。

但换个榜单，结论就没那么夸张了：

| 编程评测 | Astra | Fable 5.1 | Fable 5 | Opus 5 |
|---|---:|---:|---:|---:|
| FrontierCode 1.1 Main | 53.3% | 50.9% | 53.5% | 53.4% |
| Coding Agent Index v1.4 | 67.0 | — | 67.2 | 68.1 |
| DeepSWE v1.1 | 74.1% | 67.4% | 69.9% | 73.7% |

在 FrontierCode Main 上，Astra 略低于 Fable 5 和 Opus 5；Coding Agent Index 也没有拿第一。

综合智能指数同样如此：Astra 是 61.2，Fable 5.1 是 65.7，Opus 5 是 63.1。Humanity's Last Exam 带工具评测里，Astra 的 57.2% 也低于 Fable 5.1 的 65.0%。

所以更准确的判断是：**Astra 在终端、电脑操作、跨软件工作流和科学 Agent 上提升巨大，但“所有代码任务都第一”并不成立。**

厂商发布会最喜欢给你看自己赢的那张图。我们选模型，得把它输的表格也翻出来。

## 四个Agent更新，比跑分更值得开发者看

Astra 带来的几个接口变化，都是在解决长任务的真实痛点。

第一个是异步工具调用。工具在后台运行时，模型不用原地等着，可以继续推理、调用其他工具，或者先回答不依赖结果的部分。真正的多工具 Agent，不应该被最慢的一个接口拖死。

第二个是中途转向。模型工作时，你可以补充要求、纠正方向。Responses API 会保留已完成的工作，再从新指令继续，而不是把整项任务推倒重来。

第三个是动态调整推理强度。对话过程中可以从 Low 提到 High，也可以在简单追问时降下来，同时保留原来的 Prompt Cache。

第四个是 Codex 跨上下文记忆。过去 Context 满了只能压缩，压缩一次就可能丢掉“这个方案为什么失败”。Astra 可以保留笔记，还能搜索之前的上下文窗口，找回旧需求、测试结果和工具输出。

我很喜欢这个方向。

大仓库里最折磨人的不是模型少写一行代码，而是干了两小时后忘记自己为什么改成这样。**Agent 的上限越来越取决于记忆、工具编排和验收，不只取决于底座模型。**

## 价格翻倍，长上下文还有一层容易忽略的加价

GPT-6 Astra 的标准 API 价格是每百万输入 Token 10 美元、输出 Token 50 美元。

![GPT-6 Astra价格表](https://file1.kamacoder.com/i/web/20260904143410.png?v=20261005)

*截图说明：缓存读取为 1 美元，缓存写入为 12.5 美元。Batch 和 Flex 按标准价五折，Fast mode 按标准价两倍计费。*

和 OpenAI 当前模型目录中的 GPT-5.6 Sol 相比，Astra 的输入和输出单价都是 2.5 倍。它与 Fable 5.1 同为 10 / 50 美元这一档，是明确的旗舰攻坚价。

还有一条很容易漏看：**输入超过 272K Token 后，整次请求的输入和缓存费按两倍算，输出费按 1.5 倍算。** 不是只给超过的部分加价。

所以 1.05M 上下文能装下整个仓库，不代表你应该每轮都把整个仓库塞进去。

假设一次请求用了 30 万输入、2 万输出，未命中缓存时，基础价格不是 4 美元，而是 7.5 美元：输入 6 美元，输出 1.5 美元。再叠加搜索、电脑操作和多轮重试，账单很快就上去了。

OpenAI 强调 Astra 用更少输出 Token 完成任务，单任务成本可能低于旧模型。这个方向可信，但最终还是要用自己的任务集测：成功一次花多少钱，而不是只看每百万 Token。

## 网络安全满分，也是这次最让人不安的部分

Astra 是 OpenAI 第一款被评为网络安全“Critical”级别的广泛部署模型。

它在 ExploitBench 上拿到 100%，Sol 是 78.5%；在一个只收录 2026 年 6 月到 8 月新漏洞的内部评测上，Astra 也明显领先，并在测试中发现了两个此前未知的零日漏洞。

这当然能帮助防守方更快发现和修复问题，但同一套能力也能被用于攻击。

因此普通版本会拒绝生成更高级的漏洞利用 PoC。更开放的能力会通过 Daybreak 计划，逐步提供给经过审核的防守团队。

安全报告里还有一个不太舒服的结论：Astra 整体更守边界，在 54,218 个内部 Codex 任务模拟中，高严重度失当行为比 Sol 少约 53%；但它的书面思考过程比 Sol 更难监控，在被明确要求规避监控的对抗实验里，确实更会隐藏意图。

换句话说，**它更少做坏事了，但真想藏时，也更会藏了。**

OpenAI 给所有 Astra 工具调用部署了额外监控，检测到潜在越权时，ChatGPT 或 Codex 可能暂停任务让用户确认，API 则会直接停止。

能力越强，权限管理越不能靠一句“请谨慎操作”。生产系统要把授权范围、不可逆操作确认、审计日志和回滚机制写进 Harness。

## Astra、Fable 5.1、Opus 5、Sol怎么选

不是站队，还是算账。

| 你的任务 | 优先试谁 | 原因 |
|---|---|---|
| 跨浏览器和办公软件执行，时间很贵 | GPT-6 Astra | Computer Use 和自动化提升最明显 |
| 科研终端、复杂数据分析、长链路攻坚 | GPT-6 Astra / Fable 5.1 | 两者都强，拿真实任务比较成功率和成本 |
| 高难知识问答、综合推理 | Fable 5.1 | 部分综合评测仍领先 Astra |
| 高频 Claude Code、常规工程开发 | Opus 5 | 单价是 Astra 的一半，部分编程榜单并不落后 |
| 已经跑稳的 Codex 日常工作流 | GPT-5.6 Sol | 价格低很多，简单任务没必要为 Astra 买单 |
| 涉及高风险网络安全 | 先看访问资格和审计要求 | 能力很强，但限制、监控和合规成本也更高 |

我对 GPT-6 Astra 的个人判断是：它不是一个“聊天突然像真人”的版本，而是一个 **电脑开始真正变成模型工作环境** 的版本。

这比“又多答对几道题”重要，也比“AGI 已到”更具体。

但今天的 Astra 仍然贵，仍然会失败，首批开放范围也有限。最漂亮的三个分数里，两个接近满分、一个已经满分，可复杂办公自动化的成功率仍只有 41.4%。

这组反差，才是我们应该记住的。

**模型开始能替你操作整台电脑了，但把手完全离开方向盘，还没到时候。**

## 资料来源

* OpenAI：GPT-6 Astra 官方发布页：https://openai.com/index/gpt-6-astra/

* OpenAI Developers：GPT-6 Astra 模型规格与价格：https://developers.openai.com/api/docs/models/gpt-6-astra

* OpenAI Developers：GPT-6 Astra 模型指南：https://developers.openai.com/api/docs/guides/latest-model?model=gpt-6-astra

* OpenAI：GPT-6 Astra 安全概览：https://openai.com/index/safety-overview-gpt-6-astra/

* OpenAI Deployment Safety Hub：GPT-6 Astra System Card：https://deploymentsafety.openai.com/gpt-6-astra/

* OpenAI Help Center：GPT-5.6 与 GPT-6 Pro 开放范围：https://help.openai.com/en/articles/20001354-gpt-56-and-gpt-6-pro-in-chatgpt
