---
title: GPT-6.1 Sol正式发布：价格、跑分、Codex与GPT-6 Astra、Claude Opus 5.5怎么选
description: OpenAI发布GPT-6.1 Sol，距离GPT-6 Sol上线仅7天。本文结合官方截图和数据，分析GPT-6.1 Sol的DeepSWE、OSWorld 2.0、AutomationBench表现，API价格、缓存费用、Codex与ChatGPT Work开放范围，并比较GPT-6 Astra、Claude Opus 5.5和GPT-6 Luna怎么选，附国内充值ChatGPT Plus方法。
keywords: [GPT-6.1 Sol, GPT6.1 Sol, GPT-6 Sol, GPT6 Sol, GPT-6 Astra, Claude Opus 5.5, GPT-6 Luna, OpenAI, Codex, ChatGPT Work, ChatGPT Plus, ChatGPT充值, AI编程, Agent, Computer Use, DeepSWE, OSWorld 2.0, AutomationBench, API价格, 提示词缓存, 105万上下文]
tags: [GPT-6, GPT-6.1 Sol, OpenAI, Codex, ChatGPT, AI编程, Agent, 大模型发布, 大模型评测]
---

# GPT-6 Sol才发布7天，OpenAI就用GPT-6.1 Sol把它替了

> 国内没有海外信用卡，又想开通会员体验新模型的录友，可以直接看这篇：[ChatGPT充值方法推荐：国内开通Plus怎么选？](/ai/chatgpt/recharge.md)

9 月 22 日，OpenAI 刚发布 GPT-6 Sol。

9 月 29 日，**GPT-6.1 Sol 又来了。**

中间只隔了 7 天。

这次升级最值得关注的，不是名字多了一个“.1”，而是 OpenAI 做了三件非常实际的事：

- DeepSWE、电脑操作和专业文档任务继续提分；
- 输入、输出单价不变，把缓存读取价格再砍一半；
- 直接进入 ChatGPT Work、Codex 和 API，开始替代 GPT-6 Sol 的日常主力位置。

![GPT-6.1 Sol发布公告](https://file1.kamacoder.com/i/web/20261004111233.jpg?v=20261005)

OpenAI 给它的定位很直接：**用 Astra 五分之一的标准 Token 单价，做接近 Astra 的复杂工作。**

我先说结论。

GPT-6.1 Sol 不是新的能力天花板，最难任务还是 Astra 和 [Claude Opus 5.5](./gpt-6-sol-luna-claude-opus-5-5.md) 更值得测。

但如果你每天都在跑 Codex、代码 Agent、网页操作和长链路任务，**6.1 Sol 很可能是当前最适合当主力的 GPT 模型。**

## GPT-6 Sol没降价，但缓存便宜了一半

先看价格。

| 模型 | 输入 / 百万Token | 缓存读取 / 百万Token | 输出 / 百万Token | 定位 |
|---|---:|---:|---:|---|
| GPT-6 Astra | 10美元 | 1美元 | 50美元 | 能力上限 |
| GPT-6.1 Sol | 2美元 | 0.10美元 | 10美元 | 复杂任务主力 |
| GPT-6 Sol | 2美元 | 0.20美元 | 10美元 | 上一版Sol |
| GPT-6 Luna | 0.10美元 | 0.01美元 | 0.50美元 | 高频批处理 |

6.1 Sol 的普通输入和输出价格没有变化，仍然是 2 美元和 10 美元。

真正调整的是缓存读取：**从每百万 Token 0.20 美元降到 0.10 美元。**

这个数字对偶尔调用一次 API 的人感知不强，对 Agent 却很重要。

Agent 会反复带上系统提示词、仓库规范、工具说明和历史上下文。如果前缀能稳定命中缓存，同一份上下文每跑一轮都重新付全价，和只付 5% 的输入价格，量一大完全是两回事。

所以这次不是简单的“模型更聪明了”。

**OpenAI 是在逼大家把 6.1 Sol 当作可以长期跑的执行模型。**

不过还有一个容易被忽略的价格规则：单次输入超过 272K Token 后，整次请求的输入和缓存费率按 2 倍计算，输出按 1.5 倍计算。

105 万上下文能装很多东西，但别把整个仓库不加筛选地塞进去。**能装，不等于应该装。**

## DeepSWE提升6.4个百分点，而且成本更低

DeepSWE v1.1 测的是模型在真实代码仓库里处理长链路软件工程任务的能力。

GPT-6.1 Sol 在 high 推理档位拿到 **75.2%**，GPT-6 Sol 的最好成绩是 max 档位的 **68.8%**。

也就是说，6.1 Sol 不仅高了 6.4 个百分点，达到最好成绩时使用的推理档位还更低。

![DeepSWE成本得分曲线](https://file1.kamacoder.com/i/web/20261004111235.jpg?v=20261005)

这张图不要只看最上面的点。

横轴是单任务成本，纵轴是得分。6.1 Sol 的意义在于：**它把更高的得分，往成本曲线左边拉了。**

对录友来说，这比“某一个 max 档多两分”更实用。

日常写功能、修 Bug、补测试，很少有人愿意每次都用最贵档位。如果 high 已经能超过上一代 max，说明同一份预算能多跑几轮，也更敢让 Agent 自己检查和返工。

但也别把一张官方图理解成“6.1 Sol 已经全面等于 Astra”。DeepSWE 只是软件工程任务的一部分，不代表架构判断、前端审美、复杂研究和所有真实仓库都已经追平。

## 电脑操作距离Astra只差2.1个百分点

OSWorld 2.0 测的是模型操作真实电脑应用、完成多步工作流的能力。

在离线集、max 推理档位下：

- GPT-6.1 Sol：71.4%；
- GPT-6 Sol：约 64.4%；
- GPT-6 Astra：73.5%。

6.1 Sol 比上一版提高 7 个百分点，距离 Astra 只差 2.1 个百分点；OpenAI 给出的单任务成本大约是 Astra 的七分之一。

![OSWorld成本得分曲线](https://file1.kamacoder.com/i/web/20261004111236.jpg?v=20261005)

这项提升对 ChatGPT Work 比对普通聊天更重要。

聊天模型答错一句，你再问一次就行。电脑操作一旦点错按钮、选错文件、漏掉确认步骤，前面十分钟可能全部白跑。

所以我判断 Agent 模型，不只看它会不会写答案，更看三个指标：

- 能不能连续完成多步操作；
- 出现异常时会不会停下来检查；
- 失败一次到底要花多少钱。

6.1 Sol 这次真正补的，就是第三项和前两项之间的平衡。

## 其他跑分也在涨，但别被官方榜单带着跑

除了 DeepSWE 和 OSWorld，OpenAI 还公布了几组数据：

- AutomationBench：medium 档比 GPT-6 Sol 同档高 4.8 个百分点，比 Opus 5.5 高 2.2 个百分点，单任务成本约为后者三分之一；
- GDP.pdf：复杂 PDF 专业问答超过带回退机制的 Opus 5.5，任务成本不到一半；
- Terminal-Bench Science 0.1：max 档得分超过 GPT-6 Sol 两倍，平均单任务成本 5.47 美元；
- 事实性评测：low 档回答包含事实错误的比例，从 GPT-6 Sol 的 11.4% 降到 7.7%。

这些数字证明 6.1 Sol 不是只修了一个编程榜单。

但边界也必须说清楚：**上面主要是 OpenAI 自己发布的评测，运行环境、系统提示词和工具都可能与生产环境不同。**

特别是“超过 Opus 5.5”这种结论，只能限定在对应基准、对应推理档位和成本条件里，不能外推成“综合能力全面反超”。

模型选型最终还是要拿你自己的任务测。

如果你在意前端设计、复杂重构和长时间自主推进，把 6.1 Sol、Opus 5.5、Astra 用同一份需求、同一套验收标准跑一遍，比看十张官方海报有用。

## 105万上下文，工具能力基本拉满

GPT-6.1 Sol 的 API 模型名是 `gpt-6.1-sol`。

主要规格如下：

| 项目 | GPT-6.1 Sol |
|---|---|
| 上下文窗口 | 1,050,000 Token |
| 最大输出 | 128,000 Token |
| 知识截止时间 | 2026年4月30日 |
| 推理档位 | low、medium、high、xhigh、max |
| 图片输入 | 支持 |
| 音频输入 | 不支持 |
| 工具 | Web Search、File Search、Code Interpreter、Hosted Shell、Computer Use、MCP、Skills等 |
| 微调 | 不支持 |

这里有两个迁移坑。

第一，6.1 Sol 不支持 `none` 和 `minimal` 推理档位。从 GPT-6 Sol 切换时，如果原来写死了这两个值，先改成 `low` 再做回归测试。

第二，要让模型调用工具，官方建议使用 Responses API。Chat Completions 可以调用模型，但不支持 6.1 Sol 的工具调用。

别只把模型 ID 替换完就上线。**推理参数和 API 入口也要一起检查。**

## 已经进入Codex，但普通Chat里还没有

GPT-6.1 Sol 已经向 Plus、Pro、Business、Enterprise 和 Edu 用户开放，入口是 **ChatGPT Work 和 Codex**，同时也能通过 API 调用。

它目前还没有进入普通 ChatGPT 对话。

![GPT-6.1 Sol模型入口](https://file1.kamacoder.com/i/web/20261004111237.png?v=20261005)

部分账号的模型选择器已经能看到 GPT-6.1 Sol。OpenAI 采用分批推送，如果你的付费账号暂时没有，先检查桌面端是否更新，再等灰度，不要反复切地区或者重新付款。

官方还预告了 GPT-6.1 Sol 的 Ultrafast 模式，生成速度最高可达到标准速度的 8 倍。速度档通常会对应更高成本，正式使用前还是要看自己账号显示的价格和额度规则。

## 6.1 Sol、Astra、Opus 5.5和Luna怎么选？

别站队，直接按任务算账。

| 你的任务 | 优先测试 | 原因 |
|---|---|---|
| 日常Codex开发、修Bug、补测试 | GPT-6.1 Sol | 编程能力和成本最平衡 |
| 高频Agent、反复复用长上下文 | GPT-6.1 Sol | 缓存读取只有0.10美元/百万Token |
| 大规模分类、抽取、简单Worker | GPT-6 Luna | 单价比Sol再低一个数量级 |
| 最难推理、科学研究、失败代价极高 | GPT-6 Astra | 仍是GPT-6家族能力上限 |
| 大仓库重构、前端设计、长时间自主任务 | Claude Opus 5.5与Astra一起测 | 官方单项跑分不能代替真实项目验收 |

我的实际建议是分层：

**Luna 跑便宜的前置任务，6.1 Sol 扛日常主力，真失败了再升级到 Astra 或 Opus 5.5。**

这样做的价值，不是每次都选到排行榜第一，而是让绝大多数任务不必按旗舰价格结算。

## 国内怎么充值ChatGPT会员？

想在 ChatGPT Work 或 Codex 里使用 6.1 Sol，首先要有符合开放范围的付费套餐。

有稳定海外信用卡的录友，优先走 ChatGPT 官网订阅。没有海外卡、不想折腾美区 Apple ID 或虚拟信用卡，可以参考 `qita` 目录里这篇完整教程：

👉 **[国内充值ChatGPT Plus完整方法](/ai/chatgpt/recharge.md)**

教程里使用的是 第三方卡密代充：先用人民币购买对应套餐，再按照订单页教程，在自己的浏览器中完成充值。

![ChatGPT充值页面](https://file1.kamacoder.com/i/web/2026-07-31_15-45-58.jpg?v=20261005)

下单前注意三件事：

- 本文推荐的充值网站是第三方充值平台，不是 OpenAI 官方网站；
- 价格、库存和微信、支付宝等付款方式，以实际订单页为准；
- 不要把 ChatGPT 密码、验证码或临时 Session 发给客服和其他人。

如果充值过程遇到问题，保留订单号和完整报错截图，通过平台公开客服入口处理。

![ChatGPT充值客服](https://file1.kamacoder.com/i/web/2026-07-31_15-55-52.jpg?v=20261005)

还要再提醒一句：**ChatGPT Plus 和 OpenAI API 是两套独立计费。**

开通 Plus，可以获得套餐包含的 ChatGPT Work、Codex 使用权益；如果你要在自己的程序里调用 `gpt-6.1-sol`，API 余额仍然需要单独充值。

GPT-6 Sol 发布 7 天就被 6.1 Sol 抢走位置，说明现在模型迭代已经快到“刚写完评测，主力型号就换了”。

但对真正使用模型的人，这是好事。

**能力上限还在涨，能长期跑起来的价格，终于也开始往下掉了。**

## 资料来源

* OpenAI：GPT-6.1 Sol官方发布页：https://openai.com/index/introducing-gpt-6-1-sol/

* OpenAI Developers：GPT-6.1 Sol模型规格：https://developers.openai.com/api/docs/models/gpt-6.1-sol

* OpenAI Help Center：ChatGPT Work与Codex开放范围：https://help.openai.com/en/articles/20001275-chatgpt-work-and-codex

