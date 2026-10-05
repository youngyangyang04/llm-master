---
title: "Claude Fable 5.1发布：缓存价降75%，和Opus 5、GPT-5.6 Sol怎么选"
description: "Anthropic于2026年9月1日发布Claude Fable 5.1和Mythos 5.1。新模型强化AI编程、知识工作、长链路Agent和科学研究能力，API输入输出单价不变，但缓存读取降至0.25美元/百万Token，官方估算典型任务成本降低25%、重Agent任务最多降低45%。本文结合官方发布页截图，拆解Fable 5.1跑分、安全回退、EFS零数据留存、API模型名，以及它和Fable 5、Opus 5、GPT-5.6 Sol的选型边界。"
keywords: [Claude Fable 5.1, Fable 5.1发布, Claude Mythos 5.1, Anthropic, claude-fable-5-1, Claude API, Claude Code, AI编程, Coding Agent, 长链路Agent, Prompt Cache, 缓存读取, API价格, EFS, Enterprise Frontier Safeguards, 零数据留存, ZDR, 安全回退, 漏洞发现, Terminal-Bench 4.0, Terminal-Bench-Science 0.1, CursorBench 3.2, OSWorld 2.0, AutomationBench, Claude Fable 5, Claude Opus 5, GPT-5.6 Sol, 大模型评测]
tags: [Claude, 大模型发布, AI编程, Agent, Claude Code, 大模型评测]
---

# Claude Fable 5.1发布：缓存价降75%，Anthropic这次终于开始管账单了

前面写 [Fable 5](./claude-fable-5.md) 时，我的判断是：能力很强，但更像一台只在关键时刻启动的攻坚机器。

中间又来了 [Claude Opus 5](./claude-opus-5.md)，把大部分日常 Coding 和 Agent 任务接了过去。

9 月 1 日，Anthropic 发布 Claude Fable 5.1 和 Mythos 5.1。这次不是简单补丁：**能力继续往上推，缓存价格直接砍掉 75%，安全误判和数据留存也一起改了。**

![Fable 5.1发布首屏](https://file1.kamacoder.com/i/web/20260902104543.jpg?v=20261005)

*截图说明：Anthropic 官方发布页首屏，发布日期为 2026 年 9 月，页面本身标注由 Fable 5.1 制作。*

先说结论：**Fable 5.1 仍然不是便宜模型，但它第一次有机会从“偶尔攻坚”走进一部分高价值日常工作流。**

## 一、5.1到底更新了什么

先把最重要的信息放在一起：

| 项目 | Claude Fable 5.1 |
|---|---|
| 发布时间 | 2026 年 9 月 1 日 |
| API 模型名 | `claude-fable-5-1` |
| 可用平台 | Claude 全平台、Claude API、AWS、Google Cloud、Microsoft Azure |
| 输入价格 | 10 美元 / 百万 Token |
| 输出价格 | 50 美元 / 百万 Token |
| 缓存读取 | 0.25 美元 / 百万 Token，比 Fable 5 低 75% |
| 默认 Effort | Claude Code 为 High；Cowork 和 Claude.ai 为 Medium |
| 核心场景 | Coding、知识工作、长时间 Agent、科学研究 |
| 数据留存 | 默认仍是 30 天；符合条件的企业客户可用 ZDR 过渡方案，后续接入 EFS |

输入和输出单价一分钱没降。

所以标题里的“降价”，准确说是 **Prompt Cache 读取降价**，不是所有 Token 打二五折。

但对 Agent 来说，这个变化很实在。长任务会反复带上系统提示、工具定义、仓库上下文和历史结果，缓存命中占比越高，降价越明显。

Anthropic 用 2026 年 8 月四周真实用量估算：典型 Fable 工作负载总成本下降约 25%，上下文和工具调用密集的 Agent 工作负载最多下降约 45%。

![Fable 5.1成本结构](https://file1.kamacoder.com/i/web/20260902104547.jpg?v=20261005)

*截图说明：左边是典型任务，成本指数从 100 降到 75；右边是高 Agent 化任务，从 100 降到 55。阴影部分代表缓存读取成本。*

这张图也提醒录友一件事：**只有缓存真的命中，账单才会降。**

如果你的 Agent 每轮都改系统提示、打乱工具定义、塞入全新的上下文，缓存复用率很低，那 45% 和你没多大关系。上线前别只看单价，要记录 `cache_read_input_tokens`、总 Token、任务成功率和人工接管次数。

## 二、这次不只是更便宜，能力也拉开了

官方把 Fable 5.1 定位成目前最强的通用 Claude，重点不是一次回答多聪明，而是能否在长任务里持续推进、发现根因、失败后恢复。

最夸张的一项是 Terminal-Bench-Science 0.1：

- Fable 5.1 Max：52.6%
- Fable 5 Max：24.7%
- Opus 5：29.0%
- GPT-5.6 Sol：22.4%

![Fable 5.1科学任务曲线](https://file1.kamacoder.com/i/web/20260902104544.jpg?v=20261005)

*截图说明：横轴是单任务平均成本，纵轴是正确率。Fable 5.1 从 Low 到 Max 的整条曲线都在 Fable 5 上方。*

更值得看的不是 Max 那个点。

Fable 5.1 的 Low 档是 26.3%，已经略高于 Fable 5 Max 的 24.7%，单任务平均成本则从 44.1 美元降到 11.1 美元。

这说明 5.1 的进步不是“多烧 Token 换高分”。**同一类任务，它能用更低 Effort 接近甚至超过上一代的最高档。**

不过别把一张厂商曲线翻译成“所有科研任务都翻倍”。官方也写了，Terminal-Bench-Science 每个模型的标准误差约为 3.5 到 4.5 个百分点，而且不同 Harness、工具和重试策略都会影响成本。

真正接入时，先拿你自己的 20 到 50 个任务跑 Low、Medium、High 三档。Max 不是默认答案。

## 三、完整跑分很强，但有一个脚注不能删

官方总表里，Fable 5.1 在列出的项目上全部领先 Fable 5，也在多数项目上高于 Opus 5 和 GPT-5.6 Sol：

![Fable 5.1官方跑分](https://file1.kamacoder.com/i/web/20260902104545.jpg?v=20261005)

*截图说明：官方对比覆盖科研 Agent、终端编程、知识工作、电脑操作、综合推理、业务工作流和 Cursor 编程评测。绿色列为 Fable 5.1。*

几个对真实 Agent 比较有参考价值的数字：

| 评测 | Fable 5.1 | Fable 5 | Opus 5 | GPT-5.6 Sol |
|---|---:|---:|---:|---:|
| Terminal-Bench 4.0 | 55.8% | 42.0% | 52.3% | 37.3% |
| OSWorld 2.0 strict | 41.7% | 36.1% | 39.6% | — |
| AutomationBench | 31.4% | 17.1% | 26.9% | 19.6% |
| CursorBench 3.2.0 | 73.4% | 70.5% | 70.0% | 67.2% |

这里必须把脚注带上。

Fable 5.1 是带生产安全策略跑的。遇到安全策略介入时，部分网络安全任务会交给 Opus 4.8，生物任务会交给 Opus 5；OSWorld 的部分任务则直接计零分。

所以这张表测到的不只是一个裸模型，也包含 **Fable 5.1 + 安全分类器 + 回退模型** 这套生产系统。

这对普通用户未必是坏事，但对评测解读很重要。你的任务如果容易触发网络安全或生物安全边界，实际体验可能和表里的平均分差很多。

## 四、安全回退终于没那么容易误伤

Fable 5 发布时最让开发者头疼的，不只是贵，还有安全分类器误判。

Fable 5.1 的网络安全策略把误介入次数平均减少约 60%。现在模型可以用于在源代码中发现漏洞，但渗透测试、漏洞利用生成、基于二进制的漏洞扫描，仍可能被转交给 Opus 模型。

生物安全策略对基础生物和医疗类正常问题的误判，相比 Fable 5 首发时减少约 85%。研究和开发级生命科学问题仍会进入更严格的回退或受控访问流程。

录友写 API 接入时，不要假设“请求了 Fable 5.1，执行的一定还是 Fable 5.1”。生产环境至少要记录：

- 最终模型和停止原因；
- 是否触发回退；
- 回退后的质量、延迟和费用；
- 哪一类业务问题最容易被误判。

**安全回退不是报错兜底，而是一条会改变模型能力的正常业务分支。**

## 五、30天留存没彻底取消，但企业多了一条路

Fable 5.1 默认仍要求 30 天数据留存，用于安全监控。

变化在 Enterprise Frontier Safeguards，也就是 EFS。数据会保存在客户自己控制的云基础设施里，默认由客户而不是 Anthropic 做人工审查，在提供安全监控的同时实现接近零数据留存协议的隐私边界。

EFS 会从 2026 年秋季开始分阶段推出。在它正式可用前，符合资格的企业客户可以先用 Fable 5.1 的零数据留存过渡方案。

这不是“所有 API 用户自动获得 ZDR”。涉及源码、医疗、金融、法务和客户数据的团队，接入前仍要确认自己的组织资格、云平台支持和合同条款。


## 六、Fable 5.1、Opus 5、GPT-5.6 Sol怎么选

还是看失败成本和任务量。

| 你的任务 | 优先试谁 | 原因 |
|---|---|---|
| 多天运行、跨应用、复杂根因分析，失败一次代价很高 | Fable 5.1 | 能力上限最高，长链路和自我验证更强 |
| 高频 Claude Code、常规代码审查和功能开发 | [Opus 5](./claude-opus-5.md) | 输入输出单价是 Fable 的一半，日常任务更容易算账 |
| 多智能体并行、成本敏感、已经在 Codex 工作流里 | [GPT-5.6 Sol](./gpt-5-6.md) | 工具链和多 Agent 模式不同，不要只按 Anthropic 私榜迁移 |
| 旧 Fable 5 工作流，缓存命中率高 | 直接 A/B Fable 5.1 | 同单价、缓存更便宜，Low 或 Medium 就可能超过旧版高档 |
| 涉及高风险网络安全或生命科学 | 先确认回退和访问资格 | 安全策略可能换模型，Mythos 5.1 只向审核组织开放 |

Fable 5.1 最值得关注的，不是又多拿了几个第一。

而是它开始把 **成功率、Token 效率、缓存成本和长任务稳定性** 放在同一张账单里。

普通问答和小改动没必要上 Fable。真正值得用它的，是那些你愿意花更多钱，但不能接受模型做到一半就丢下的任务。

**先用真实任务测 Medium，再决定要不要为 Max 买单。**

## 资料来源

* Anthropic：Fable 5.1 与 Mythos 5.1 官方发布页：https://www.anthropic.com/claude-fable-and-mythos-5-1

* Anthropic：Claude Fable 模型页与价格说明：https://www.anthropic.com/claude/fable

