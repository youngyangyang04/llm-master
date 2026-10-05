---
title: Claude Fable 5发布：最强模型来了，但先别急着把Opus 4.8换掉
description: Anthropic于2026年6月9日发布Claude Fable 5，官方称这是目前最强的公开发布模型，主打高难度推理和长链路Agent。本文从定价翻倍、新tokenizer多算30%、adaptive thinking常开、refusal拒答与fallback、effort思考强度、30天数据留存几个角度，拆解Fable 5到底是给谁用的，以及和Claude Opus 4.8怎么选。
keywords: [Claude Fable 5, Fable 5发布, Anthropic最强模型, Claude Mythos 5, Project Glasswing, adaptive thinking, effort思考强度, refusal拒答, fallback回退, fallback credit, 新tokenizer, 1M上下文, 长链路Agent, AI编程, Claude Opus 4.8对比, claude-fable-5]
tags: [Claude, 大模型发布, AI编程, Agent]
---

# Claude Fable 5发布：最强模型来了，但先别急着把Opus 4.8换掉

> 不少读者问我，如何Claude账号被封，如果用上Claude opus4.8，或者Fable5，充值Claude会员，我在这里单独讲一下：[没有Claude账号，怎么用上Opus模型](../../qita/0020.no-claude-account-use-opus-4-8-fable-5.md)


6月9日，Anthropic 发布了 Claude Fable 5。

![](https://file1.kamacoder.com/i/web/2026-06-10_14-34-30.jpg?v=20261005)

现在就已经可以用上了：

![](https://file1.kamacoder.com/i/web/2026-06-11_11-19-01.jpg?v=20261005) 


官方的定位很直接：**目前最强的公开发布模型**，专门给高难度推理和长链路 Agent 任务用。

官方介绍在这里：https://www.anthropic.com/news/claude-fable-5-mythos-5

Anthropic 表示，Fable 5 堪称其有史以来最强大的模型，而从下面的跑分表中可以看出出来，确实是碾压。

![](https://file1.kamacoder.com/i/web/2026-06-11_11-19-02.jpg?v=20261005)

不过他的价格也是最贵的。

![](https://file1.kamacoder.com/i/web/20260611143412_position.png?v=20261005)

这次和上次 Opus 4.8 的"温和更新"不一样。

Opus 4.8 是把老问题往前推一步，Fable 5 是换了一个梯队。

但卡哥要先泼一盆冷水：

**Fable 5 很强，可它不是用来替换你手里 Opus 4.8 的。**

它更像一台攻坚机器，平时不开，关键时刻才上。

为什么这么说？往下看。

## 一、先看价格，这是绕不过去的事

很多文章一上来就吹能力，卡哥习惯先看账单。

这是官网价格：https://claude.com/pricing#api，大家可以去看。

![](https://file1.kamacoder.com/i/web/2026-06-11_11-25-15.jpg?v=20261005) 

| 模型 | 输入价格 | 输出价格 |
|------|----------|----------|
| Claude Fable 5 | $10 / 百万 token | $50 / 百万 token |
| Claude Opus 4.8 | $5 / 百万 token | $25 / 百万 token |

一眼就能看出来：**Fable 5 的单价，正好是 Opus 4.8 的两倍。**

但真实成本的涨幅，不止两倍。

因为 Fable 5 换了新的 tokenizer。

同样一段内容，在 Fable 5 上切出来的 token 数，比 Opus 那一代大约多 **30%**。

这两个因素叠在一起，**同一个任务的实际花费，可能是 Opus 4.8 的 2.5 倍甚至更多。**

![](https://file1.kamacoder.com/i/web/20260611143412_cost.png?v=20261005)

所以那种"既然更强就全换 Fable 5"的想法，钱包会第一个反对。

录友要记住一句话：

**Fable 5 是按"难度"付费的，不是按"数量"付费的。**

## 二、它到底强在哪

价格摆完，再说能力，这样心里有数。

Fable 5 的提升不在"会不会"，而在"扛不扛得住"。

它最值得用的几类场景：

**第一，长链路 Agent。**

就是那种一口气跑很久、中间要不停读文件、调工具、自我验证的任务。

这类任务最考验模型的"耐力"和"连续性"，Fable 5 在这上面是目前最稳的。

**第二，一次成型的复杂实现。**

需求说清楚之后，让它从零搭一个结构完整的系统，而不是来回补丁。

**第三，难啃的代码审查和 debug。**

尤其是那种藏得深、偶发的 bug，以前模型容易"跑一次没复现就说修好了"，Fable 5 在这点上更老实。

**第四，密集视觉和长上下文。**

1M 上下文是默认值，单次最多输出 128K token。配合高分辨率视觉，处理大文档、截图、图表更从容。

但注意，**这些都是"上限"提升，不是"日常"提升。**

你让它写个普通脚本、解释段代码，它不会比 Opus 4.8 强到哪去，钱反而花得更多。

## 三、接入方式变了，老代码可能直接报错

这一节最重要，做开发的录友一定要看。

Fable 5 不是把模型名一换就完事，它的 API 行为有几处硬变化。

**第一，思考是常开的，关不掉。**

在 Fable 5 上，adaptive thinking 是唯一的思考模式。

你不传 `thinking` 参数，它就默认在思考。

你想像以前那样 `thinking: {"type": "disabled"}` 关掉？直接报 400。

控制深浅只能靠 `effort`：

```python
# Fable 5：不要再传 thinking 的开关，用 effort 控深浅
response = client.messages.create(
    model="claude-fable-5",
    max_tokens=64000,
    output_config={"effort": "high"},  # low / medium / high / xhigh / max
    messages=[...],
)
```

我的建议和之前一样：

- 常规任务：`high` 起步
- 编程、Agent 长任务：`xhigh`
- 真的很难、又不在乎慢和贵：才上 `max`

别无脑拉满。Fable 5 本身底子高，低一档往往就够，还省钱。

**第二，原始思考过程拿不到了。**

Fable 5 永远不会把原始的思维链返回给你。

你能拿到的，是 `thinking.display` 控制的两种形态：`"summarized"` 给你一段可读的摘要，`"omitted"`（默认）则是空的。

如果你之前靠读模型的思考内容做产品展示，这里要改。

**第三，模型会"拒答"，你的代码要接得住。**

这是 Fable 5 最容易踩坑的地方。

它带了安全分类器，某些请求会被拒。被拒的时候，**返回的是 HTTP 200，不是报错**，但 `stop_reason` 是 `"refusal"`。

意思是：如果你的代码上来就读 `response.content[0]`，碰到拒答会直接崩。

正确姿势是先判断 `stop_reason`：

```python
response = client.messages.create(model="claude-fable-5", max_tokens=1024, messages=[...])
if response.stop_reason == "refusal":
    handle_refusal()   # 出口前拒答：content 是空的，不计费
else:
    print(response.content[0].text)
```

被拒之后想换个模型重试，官方给了三条路：服务端的 `fallbacks` 参数、SDK 的客户端中间件、自己手搓。配合 fallback credit，重试时还能把切模型的缓存成本退回来，不至于双倍付钱。

![](https://file1.kamacoder.com/i/web/20260611143412_refusal_fallback.png?v=20261005)

**第四，数据留存有要求。**

Fable 5 要求 30 天数据留存，**不支持零留存（ZDR）**。

如果你们组织是零留存配置，那 Fable 5 的请求会直接 400。接入前先确认这条，不然会以为是请求写错了，白排查半天。

## 四、还有个 Mythos 5，别搞混

发布里还提了一个 Claude Mythos 5。

简单说：**Mythos 5 和 Fable 5 能力、价格、规格完全一样，区别是 Mythos 5 没有那套安全分类器。**

但它不是谁都能用，只在 Project Glasswing 里向通过审核的客户限量开放。

普通录友够不着，用 Fable 5 就行，能力是一样的。

## 五、和Opus 4.8到底怎么选

这才是大多数人真正关心的问题。

卡哥的判断还是那条老规矩，只是这次门槛更高了：

**便宜模型跑量，顶级模型攻坚，Fable 5 攻的是"硬骨头里的硬骨头"。**

| 场景 | 推荐 | 原因 |
|------|------|------|
| 普通问答、文章初稿、代码解释 | Opus 4.8 / Sonnet 4.6 | 够用，便宜得多 |
| 多文件改动、常规 bug 修复、PR 审查 | Claude Opus 4.8 | 性价比甜区，绝大多数开发任务它就够 |
| 超长链路 Agent、一次成型复杂系统、偶发深层 bug | Claude Fable 5 | 耐力和连续性最强，难任务里更稳 |
| 大仓库迁移、安全审计、跨模块重构 | Fable 5 + 子 Agent 并行 | 任务能拆、能并行、能自我验证 |

![](https://file1.kamacoder.com/i/web/20260611143412_model_choice.png?v=20261005)

注意，Opus 4.8 没有被淘汰。

对绝大多数开发任务，**Opus 4.8 仍然是默认选择**，性价比明显更好。

只有当一个任务满足这几条，才值得把它升到 Fable 5：

- 失败成本很高，错了代价大
- 上下文极复杂，需要持续判断
- 链路特别长，要扛很久
- 普通模型反复试都搞不定

满足不了这几条，用 Fable 5 就是在烧钱。

## 写在最后

Fable 5 是 Anthropic 目前最强的公开模型，这一点没有水分。

但"最强"不等于"该上"。

它定价翻倍、tokenizer 多算 30%、接入还得改 refusal 和思考参数。

这些成本，只有真正的硬任务才扛得起。

所以卡哥的建议很简单：

**Opus 4.8 继续当主力，把 Fable 5 留给那几个你一直没搞定的难题。**

先拿小任务试水，别一上来就让它扫整个代码库。

自己测，按难度选。

