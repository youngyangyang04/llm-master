---
title: ChatGPT、Claude、Grok集体宕机事件复盘：故障重叠93分钟，Azure、Cloudflare是元凶吗
description: 2026年9月3日，ChatGPT、Claude、Grok在同一时间窗口发生服务故障，三家官方事件窗口重叠约93分钟。本文结合OpenAI、Anthropic、xAI状态页关键截图，还原北京时间线、影响范围与已知原因，澄清Gemini是否幸存、Azure和Cloudflare是否导致集体宕机，并给出AI工作流容灾方案。
keywords: [ChatGPT宕机, Claude宕机, Grok宕机, ChatGPT Claude Grok集体宕机, AI集体宕机, OpenAI故障, Anthropic故障, xAI故障, Codex宕机, Claude Code宕机, Gemini宕机, Azure故障, Cloudflare故障, DownDetector, AI服务中断, AI工作流容灾, 大模型高可用]
tags: [ChatGPT, Claude, Grok, OpenAI, Anthropic, xAI, AI基础设施, 大模型动态]
---

# ChatGPT、Claude、Grok同时宕机：重叠93分钟，但“同一个元凶”没有证据

2026年 9 月 3 日晚上，不少录友经历了很魔幻的一幕。

Claude 报错，切到 Grok，还是报错；再打开 ChatGPT，连 Codex 也开始出问题。

看起来就像三家约好了，一起拔网线。

但我把三家官方状态页、公开回应和故障统计对了一遍，结论和网上流传最广的版本不太一样：

**三家的故障窗口确实重叠了约 93 分钟，但没有证据证明它们是被同一个 Azure 或 Cloudflare 故障带走的。**

而且，Claude 和 Grok 都持续了接近 3 小时甚至更久。把这件事概括成“集体宕机 90 分钟”，也不准确。

## 先把时间线对齐：不是一起开始，也不是一起恢复

下面全部换算成北京时间。

| 北京时间 | 发生了什么 |
| --- | --- |
| 9 月 3 日 20:37 | Claude Sonnet 5 先出现一轮短暂高错误率，约 19 分钟后恢复 |
| 21:26 | Claude 多款模型出现高错误率，Claude.ai、API、Claude Code、Cowork 均受影响 |
| 21:30 | Grok 官方开始调查模型故障 |
| 22:43 | OpenAI 称路由错误开始影响 ChatGPT 和 Codex |
| 23:17 | OpenAI 实施缓解措施，进入恢复监控 |
| 9 月 4 日 00:16 | Claude 确认用户影响结束 |
| 00:55 | OpenAI 状态页将事件标记为已解决 |
| 01:05 | Grok 宣布流量恢复健康，官方记录持续 3 小时 35 分钟 |

按官方事件窗口计算，三家从 22:43 到次日 00:16 同时处于故障事件中，重叠约 **93 分钟**。

但要注意：**“事件窗口重叠 93 分钟”，不等于每个用户都连续 93 分钟完全无法使用。**

故障可能只影响部分地区、模型和功能。OpenAI 也明确说，是“部分用户”无法使用，而不是所有请求全部失败。

## ChatGPT和Codex：15个ChatGPT组件、4个Codex组件受影响

OpenAI 状态页显示，这次事件影响了 ChatGPT 的 15 个组件和 Codex 的 4 个组件。

![ChatGPT故障状态](https://file1.kamacoder.com/i/web/20260904120620.jpg?v=20261005)

这是 OpenAI 事件页的最终状态。截图使用北京时间显示：事件在 9 月 4 日 00:55 被标记为已解决。

OpenAI 对媒体给出的解释是：**北京时间 22:43 左右发生了一次路由错误，导致部分用户无法使用 ChatGPT 和 Codex；23:17 左右已经实施解决方案。**

也就是说，明确披露的用户影响与缓解之间约 34 分钟；后面较长的一段时间，是恢复监控和最终关闭事件。

第三方报障数据看起来更吓人。Forbes 援引 Downdetector 数据称，ChatGPT 在美国的报告峰值超过 3.7 万条。

不过这个数字只能说明“报告量突然暴涨”，不能直接等同于受影响人数，更不能拿它和 Claude、Grok 的数字简单比较。三家用户规模不同，用户主动去 Downdetector 报告的比例也不同。

## Claude：不是只挂了网页，API和Claude Code也在范围内

Claude 这次影响更早，也更久。

Anthropic 最初只列出 Mythos 5.1、Fable 5.1 和 Opus 5，随后补充确认 Mythos/Fable 5、Opus 4.8、Opus 4.6 等模型也受到影响。

![Claude故障状态](https://file1.kamacoder.com/i/web/20260904120621.jpg?v=20261005)

官方事件页最后写得很清楚：受影响的不只是 Claude.ai，还包括 **Claude API、Claude Code 和 Claude Cowork**。

主事件从北京时间 21:26 开始，影响在次日 00:16 结束，约 **2 小时 50 分钟**。

Anthropic 表示已经找到原因并部署修复，但没有公开根因细节。

所以我们目前只能确认“Claude 自己找到了原因”，不能进一步写成“AWS 挂了”“Azure 挂了”或者“被 Grok 的机房拖下水”。这些说法都缺少官方证据。

## Grok：三家里持续最久，已知和孟菲斯计算中心有关

Grok 的官方记录最直白。

![Grok故障状态](https://file1.kamacoder.com/i/web/20260904120622.jpg?v=20261005)

北京时间 21:30 开始调查，次日 01:05 宣布流量恢复，持续 **3 小时 35 分钟**。

xAI 的状态页没有写技术原因，但 SpaceX 随后的公开说法是：**Grok 的问题来自当天早上孟菲斯计算中心的一次中断。**

至少就已经公开的信息看，Grok 有一个具体的计算中心故障；OpenAI 说的是路由错误；Anthropic 只说找到原因，没有披露是什么。

这三份口径放在一起，更不能直接得出“同一个云厂商把三家同时搞挂了”。

## 最大的误传：Azure、Cloudflare把三家一起带崩了？

这次故障刚发生时，很多报道把矛头指向 Azure、AWS 或 Cloudflare。

这个猜测并不离谱。

三家竞争对手在相近时间出问题，第一反应当然是找共同依赖：云平台、CDN、DNS、身份认证、网络骨干，任何一层出故障，都可能让不同产品同时不可用。

但**合理猜测不等于已经证实。**

WIRED 在事后核查中写到，OpenAI 和 Anthropic 都没有把故障归因于外部供应商；AWS、Azure 和 Cloudflare 当时也没有报告与这次窗口对应的重大事故。

目前能站得住的表述只有三句：

- OpenAI：路由错误；
- Claude：原因已定位，但未公开；
- Grok：孟菲斯计算中心中断。

至于为什么时间这么巧，可能是独立故障碰巧重叠，也可能存在尚未公开的关联；还可能是一家出问题后，大量用户和自动化流量切去另一家，给备用服务继续加压。

最后一种在工程上叫“故障转移风暴”，但这次是否真的发生，**目前没有公开数据可以下结论。**

## Gemini也不是“唯一幸存者”

旧说法里还有一句很抓眼球：三家全挂，只有 Gemini 活着。

实际情况更模糊。

Google 没有在官方状态页确认 Gemini 发生事故，这一点没错；但 Downdetector 同期也出现了数百条 Gemini 报告，第三方 API 探针还记录到多个地区的短时请求失败。

所以更准确的说法是：

**Gemini 没有出现一场被 Google 官方确认、规模与三家相当的故障，但也不能证明它在整个窗口里完全没受影响。**

“唯一幸存者”适合当段子，不适合当结论。

## 真正值得警惕的，不是90分钟没法问AI

如果 ChatGPT 挂了，你手动切 Claude；Claude 也挂了，再切 Grok。

这叫有三个账号，不叫有容灾。

真正的容灾，要看三件事：**故障是不是独立、上下文能不能迁移、业务能不能降级。**

很多 AI 工作流的问题是，模型虽然准备了三家，状态却只存在某一个云端会话里：需求、代码分析、工具执行记录、待办列表全在里面。服务一挂，换模型也得从头讲。

这和前面写过的 [Claude与Codex额度风波](./claude-codex-usage-limits-reset.md) 是同一个提醒：当 AI Agent 进入日常生产，额度、服务状态、后台任务和会话上下文，都已经是工作流依赖，不只是“聊天工具体验”。

个人用户至少可以做这几件事：

- 把需求、关键结论和下一步动作落到本地文档，不要只留在聊天记录里；
- 收藏三家的官方状态页，遇到连续报错先看状态，别反复重试；
- 准备一个不同厂商的备用模型，但提前验证它能否接手你的提示词、代码仓库和工具；
- 本地构建、测试、查日志这些工作继续做，别让“AI 不能回答”变成“人也完全停工”。

如果是线上 AI 产品，还要再往前一步：

- 在接入层做模型路由，不让业务代码绑死某一家 SDK；
- 对失败请求加熔断、指数退避和随机抖动，避免故障时疯狂重试；
- 给队列任务做幂等和检查点，恢复后能接着跑，而不是全部重来；
- 备用模型先限流接管，别把主服务的全部流量一秒钟灌过去；
- 对低风险任务准备本地或开源模型降级，高风险任务宁可暂停，也不要悄悄换模型后直接产出。

## 三家都挂过，状态页比段子更重要

这次最值得记住的，不是“AI 打工人被迫用脑”这种段子。

而是当三张报错页面同时出现时，网上会迅速长出一个听起来很完整的故事：同一个云厂商故障、三家一起倒下、Gemini 独自幸存。

可官方记录告诉我们的，其实是三个不同口径的事故，只是在约 93 分钟里发生了重叠。

**先把已知、未知和推测分开，再谈基础设施脆弱性。**

对普通用户，多备一个模型有用；对真正依赖 AI 交付的团队，只备账号远远不够。

## 资料来源

* OpenAI 状态页：ChatGPT 与 Codex 错误率升高：https://status.openai.com/incidents/2rm6gqeh
* Claude 状态页：多款模型错误率升高：https://status.claude.com/incidents/461yvfrzpwtt
* xAI 状态页：Grok in X 模型故障：https://status.x.ai/grok-in-x/INC429d651a
* WIRED：三家故障原因仍不明确，以及 OpenAI、SpaceX 的公开说明：https://www.wired.com/story/nobody-is-saying-why-openai-and-anthropic-had-outages-today/
* Ars Technica：四家 AI 服务故障窗口与 Gemini 报告：https://arstechnica.com/ai/2026/09/four-major-ai-models-suffer-rare-overlapping-downtime/
* Forbes：Downdetector 报告峰值统计：https://www.forbes.com/sites/maryroeloffs/2026/09/03/widespread-ai-outage-hitting-open-ai-claude-and-others/
* LLM Latency：多地区 API 探针记录：https://llmlatency.dev/incidents
