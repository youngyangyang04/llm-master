---
title: GPT-6 Luna开始向免费用户推送：ChatGPT Free入口、GPT-5.6替换范围与交互界面变化
description: OpenAI正向免费用户推送GPT-6 Luna。本文核对Free与Go账号的桌面端入口、ChatGPT Work和Codex的开放范围，解释GPT-5.6 Luna是否已被替换，梳理Intelligent UI交互界面，并介绍OpenAI数学手稿发布后的撤稿与验证情况。
keywords: [GPT-6 Luna, GPT6 Luna, GPT-6免费, ChatGPT免费用户, ChatGPT Free, ChatGPT Go, GPT-5.6 Luna, GPT-5.6, OpenAI, ChatGPT Work, Codex, ChatGPT模型切换, Intelligent UI, ChatGPT可视化, 交互式图表, OpenAI数学手稿]
tags: [GPT-6, GPT-6 Luna, OpenAI, ChatGPT, Codex, AI数学, 大模型动态]
---

# GPT-6 Luna开始给免费用户推送了，但你打开ChatGPT可能还看不到

> **如果你还没有体验过 GPT顶尖模型的能力，建议冲一个GPT会员，可以看[国内充值ChatGPT会员](../../ai/chatgpt/recharge.md)**


最近 OpenAI 的帖文截图给出了新信号：GPT-6 和新的交互界面开始扩大推送范围，Free、Go 用户也被列入其中。

![OpenAI发布GPT-6推送消息](https://file1.kamacoder.com/i/web/20261011110413.png?v=20261005)

不过，“开始推送”和“所有免费账号已经切换”，差得很远。想知道自己能不能用，先看**账号、客户端、入口**这三件事。

## 免费用户能用到的，是哪个 GPT-6？

OpenAI 的帖文截图写得很明确：付费档对应 GPT-6 Sol，Free、Go 档对应 **GPT-6 Luna**；免费档从后一阶段开始扩展。

![GPT-6免费用户开放范围](https://file1.kamacoder.com/i/web/20261011110355.jpg?v=20261005)

但截至本文核对时，OpenAI Docs 对实际可用范围的描述更窄：**Free、Go 用户可在桌面应用的 Work 和 Codex 中使用 Luna，具体取决于分批推送；普通 Chat 对话尚未列为 GPT-6 Luna 的可用入口。**

所以，录友如果打开网页版 ChatGPT，发现模型列表里还没有 Luna，先别以为账号出了问题。到桌面应用里的 Work 或 Codex 看模型选择器，更符合目前官方文档写明的开放路径。

## GPT-5.6 已经被全面取代了吗？

**还不能这么说。**

OpenAI Docs 仍把 GPT-5.6 Luna 列为 Free、Go 用户普通 Chat 对话的默认模型；GPT-6 Luna、Sol 在 ChatGPT 中的开放范围仍写为 Work 和 Codex。模型文档也说明，GPT-5.6 系列在新模型推送期间继续可用。

换句话说，这次更准确的表述是：**GPT-6 Luna 正向免费用户开放新入口，GPT-5.6 仍可能出现在你原来的聊天入口。**同一个账号里，不同界面看到不同模型，是当前推送阶段可能出现的正常情况。

这里还要分清订阅和 API。ChatGPT Free 能用桌面端的 Luna，不等于 OpenAI API 免费层可以调用 `gpt-6-luna`；模型规格页目前写的是 API Free 层不支持。

## 另一个变化：回答开始变成能操作的界面

OpenAI 同时展示了 ChatGPT 回答中出现图片、图表和可操作组件。下面这张对比截图里，同一个聚餐问题，旧版主要用文字组织答案，新版把菜品照片也放进了回答。

![GPT-6图文回答对比](https://file1.kamacoder.com/i/web/20261011110357.png?v=20261005)

图片多了，不等于回答一定更准，但在做菜、旅行、认识物体这类问题上，读者不必只靠文字想象。

另一个对比是网页搜索。截图展示了 GPT-5.6 即时模式与 GPT-6 即时模式回答同一问题的过程：新版更早开始组织可读的回复。**这类演示说明的是产品体验方向，不能只凭截图推断每个问题都会更快。**

![GPT-6网页搜索回答对比](https://file1.kamacoder.com/i/web/20261011111300.png?v=20261005)

比图片和速度更值得聊的，是**回答本身开始带操作入口**。比如给出一个储蓄目标，让读者自己调整输入，直接观察结果如何变化。

这个方向确实值得关注。以前问“每月存多少钱能达到目标”，模型给一段文字和公式；交互式界面可以把金额、年限等参数变成控件，**你改一个数，结果就跟着变**。对于学习概念、比较方案、探索数据，这比来回重写提示词省事。

但也别把它理解成“只要切到 Luna，所有回答都会自动生成交互工具”。OpenAI 的 Visualizations 文档把可视化列为**单独逐步开放的预览功能**，是否出现还与平台、账号和工作区设置有关。可用时，可以在支持的聊天中输入 `@Visualize` 选择插件；找不到这个入口，就说明当前账号尚未获得相应能力。

OpenAI 对这类界面的设想也很直接：用户说清楚想完成什么，软件围绕任务生成合适的交互方式。这个方向比“聊天框里多放几张图”走得更远，但眼下仍要看功能是否推送到你的账号，以及生成的控件算得对不对。

![ChatGPT交互界面理念](https://file1.kamacoder.com/i/web/20261011111254.png?v=20261005)

## 免费用户现在怎么确认？

打开最新版 ChatGPT 桌面应用，进入 Work 或 Codex，在输入框下方查看模型选择器。能看到 GPT-6 Luna，就可以从一个范围清楚的小任务试起：让它整理资料、提取字段，或者做一次明确要求的代码修改。

如果普通 Chat 里还是 GPT-5.6 Luna，也不用为了“换代”反复刷新。**这两个入口的推送范围，官方目前写得并不一样。**

模型名称会继续变，真正值得留意的是：免费用户能在哪个入口完成什么任务，以及新界面有没有让问题更容易被理解和操作。

## 同期还有一件大事：OpenAI公开数学手稿

这次更新之外，OpenAI 还开放了一个 [数学研究仓库](https://github.com/openai/math)。仓库收录由内部模型产出的研究手稿、部分证明材料与形式化文件，内容涉及多个数学方向。

![OpenAI数学研究仓库](https://file1.kamacoder.com/i/web/20261011111246.png?v=20261005)

最初公布的是 **722 篇手稿**。但截至本文核对时，仓库首页已经改为 **719 篇、372 个成果系列**：仓库 10 月 7 日的修订记录显示，一处符号错误导致三篇相互依赖的手稿被撤回，另有多篇做了修订。

这比单看“722”更值得关注。**AI 能批量提出研究结果，不代表每一份证明都已经通过核查。**仓库也明确说，不是所有结果都有 Lean 形式化证明，未形式化的内容可能存在问题。

数学家 Alex Kontorovich 对其中一个准黎曼猜想方向的结果给出了极高评价。截图里的激动是可以理解的，但它是对手稿的即时反应，不能直接当作独立验证结论。

![数学家评价准黎曼猜想手稿](https://file1.kamacoder.com/i/web/20261011111248.png?v=20261005)

随后他又提到 Siegel zero 相关结果，用“两枚菲尔兹奖”继续表达震惊。读这两张截图时，重点应放在**数学界为什么愿意认真检查这些结果**，而不是把社交媒体上的感叹当成奖项预测。

![数学家评价Siegel零点结果](https://file1.kamacoder.com/i/web/20261011111250.png?v=20261005)


所以我更在意的是后续：哪些结果能被复核，哪些会修订或撤回，以及研究者要花多少时间读懂这些由 AI 生成的证明。仓库公开是起点，检查才刚开始。

## 资料来源

* OpenAI Docs：ChatGPT与Codex更新日志：https://learn.chatgpt.com/docs/changelog
* OpenAI Docs：模型与开放范围：https://learn.chatgpt.com/docs/models
* OpenAI Docs：ChatGPT可视化功能：https://learn.chatgpt.com/docs/visualizations
* OpenAI Developers：GPT-6 Luna模型规格：https://developers.openai.com/api/docs/models/gpt-6-luna
* OpenAI数学研究仓库与修订记录：https://github.com/openai/math
