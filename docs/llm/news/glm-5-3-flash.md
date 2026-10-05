---
title: GLM-5.3-Flash发布：320B总参数、18B激活、原生多模态，价格只有GLM-5.3十分之一，和DeepSeek V4、Kimi K3怎么选
description: 智谱于2026年8月26日发布GLM-5.3-Flash：320B总参数、18B激活参数，首次采用线性注意力与稀疏注意力混合架构，支持1M上下文和原生多模态，并以MIT许可证开放权重。本文结合官方发布页、跑分、架构和价格截图，拆解ox-alpha身份、GLM Coding Plan三倍额度、API促销价、国产芯片推理，以及它和GLM-5.3、DeepSeek V4、Kimi K3、Claude Opus 4.8怎么选。
keywords: [GLM-5.3-Flash, GLM5.3 Flash, GLM-5.3-Flash发布, 智谱GLM-5.3-Flash, Z.ai, ox-alpha, GLM-5.3, GLM-5.2, GLM Coding Plan, 320B, 18B激活参数, MoE, 原生多模态, 1M上下文, 线性注意力, 稀疏注意力, IndexPool, mHC, 国产AI芯片, MIT开源模型, AI编程, Coding Agent, Claude Code, Claude Opus 4.8, DeepSeek V4-Flash, Kimi K3, 多模态Agent, 大模型API价格, 本地部署]
tags: [GLM, 智谱, 大模型发布, 多模态, AI编程, Agent, 开源大模型, 国产算力]
---

# GLM-5.3-Flash 发布：价格砍到十分之一，这次还把“眼睛”补上了


前面写 [GLM-5.3](./glm-5-3.md) 时，重点是同一个基座只靠后训练，编程能力还能再涨 50%。

12 天后，智谱又发布了 GLM-5.3-Flash。

名字里带着 5.3，很容易让人以为它只是“旗舰模型加速版”。

其实不是。

**GLM-5.3-Flash 是一套重新训练的 320B MoE 基座，也是 GLM-5 系列第一个原生多模态模型。**

它真正想抢的，不是旗舰模型的最高分，而是开发者每天都要算的那笔账：能力够不够、Token 贵不贵、长上下文跑不跑得动。

下面这张图，是 2026 年 8 月 26 日 [智谱官方发布页](https://z.ai/blog/glm-5.3-flash) 的首屏截图。

![GLM-5.3-Flash发布页](https://file1.kamacoder.com/i/web/20260827152737.jpg?v=20261005)

*截图说明：官方首屏给出了四个关键信息——320B 总参数、18B 激活参数、原生多模态，以及接近 Claude Opus 4.8 的 Coding 与 Agent 能力。发布前匿名测试的 `ox-alpha`，也在这里正式认领。*

## 一、它不是GLM-5.3换个便宜套餐

先把几个最容易混淆的信息放在一起：

| 项目 | GLM-5.3-Flash |
|---|---|
| 发布时间 | 2026 年 8 月 26 日 |
| 模型规模 | 320B 总参数，18B 激活参数 |
| 训练数据 | 30T Token 多模态预训练语料 |
| 上下文 | 100 万 Token |
| 输入能力 | 文本、图片、视频、文件 |
| 核心架构 | MoE + 线性注意力 + 稀疏注意力 + mHC |
| API 模型名 | `glm-5.3-flash` |
| 开源状态 | 权重已发布，MIT License |
| 本地推理 | SGLang、vLLM、TokenSpeed、KTransformers |

GLM-5.3 沿用 GLM-5.2 的基座，提升主要来自后训练。

GLM-5.3-Flash 则从基座开始重新设计，重点是用更少的激活参数、更小的注意力开销，换更低的推理成本。

所以两者不是“同一能力、不同速度”。

**GLM-5.3 负责冲复杂任务上限，Flash 负责把足够强的能力铺到高频工作流里。**

还有个有意思的发布细节。

在正式亮身份前，智谱把它叫作 `ox-alpha`，匿名放到 OpenCode 和 OpenRouter 收集反馈。官方称它很快成了当周最受欢迎的模型，测试流量全部由国产 AI 芯片承载。

先让用户盲测，再揭晓厂牌，这比只拿自家海报证明“用户喜欢”更有说服力。

## 二、价格不是便宜一点，是直接少一个零

按照 [Z.ai 官方价格页](https://docs.z.ai/guides/overview/pricing)，GLM-5.3-Flash 的标准价是：

| 每百万 Token | 标准价 | 2026 年 9 月 9 日前促销价 |
|---|---:|---:|
| 输入 | 0.15 美元 | 0.075 美元 |
| 缓存输入 | 0.03 美元 | 0.015 美元 |
| 输出 | 0.50 美元 | 0.25 美元 |

GLM-5.3 的输入是 1.4 美元，输出是 4.4 美元。

按标准价算，Flash 确实接近它的十分之一；促销期内还要再打五折。

![GLM-5.3-Flash价格](https://file1.kamacoder.com/i/web/20260827152747.jpg?v=20261005)

*截图说明：官方价格表同时列出了 Flash、GLM-5.3 和 GLM-5.2。删除线是标准价，绿色提示里的五折活动截止到 2026 年 9 月 9 日 24:00（UTC+8）。*

举个更直观的账单。

假设一个 Agent 工作流累计用了 1000 万输入 Token、200 万输出 Token，不算缓存：

- GLM-5.3：`10 × 1.4 + 2 × 4.4 = 22.8` 美元
- GLM-5.3-Flash 标准价：`10 × 0.15 + 2 × 0.5 = 2.5` 美元
- Flash 促销价：1.25 美元

**任务量一大，这已经不是“省杯咖啡”，而是能不能把 Agent 默认打开的区别。**

GLM Coding Plan 也已经全量开放 Flash，同一订阅下可用额度是 GLM-5.3 的 3 倍；非高峰时段和周末仍按 50% 积分消耗。

## 三、跑分超过GLM-5.2，但别误读成全面追平旗舰

先看官方总表截图。

![GLM-5.3-Flash跑分](https://file1.kamacoder.com/i/web/20260827152739.jpg?v=20261005)

*截图说明：上半部分是 Artificial Analysis 的能力—成本帕累托图，Flash 在促销价下每任务 0.045 美元、指数 57；下半部分是智谱汇总的六项 Coding 与 Agent 评测。*

其中几个结果比较扎眼：

| 评测 | GLM-5.3-Flash | GLM-5.2 | Claude Opus 4.8 |
|---|---:|---:|---:|
| Terminal Bench 2.1 | 84.3 | 81.0 | 85.0 |
| DeepSWE v1.1 | 63.4 | 46.2 | 58.0 |
| Agents' Last Exam | 26.3 | 20.4 | 27.0 |
| AutomationBench | 48.8 | 26.2 | 41.0 |
| HLE w/ Tools | 55.3 | 54.7 | 57.9 |
| GDPval-AA v2 | 1773 | 1504 | 1582 |

Flash 在六项里全部超过 GLM-5.2，部分项目也超过了 Opus 4.8。

但这张表不能翻译成“0.5 美元的模型全面打赢 Opus”。

不同 benchmark 的 Harness、上下文管理、最大输出和超时条件并不完全一样；其中部分结果还是官方自行执行。

再看智谱自己的 Z.ai Code Bench，边界就更清楚了。

![GLM-5.3-Flash编程效率](https://file1.kamacoder.com/i/web/20260827152741.jpg?v=20261005)

*截图说明：紫线是 GLM-5.3-Flash。`max` 档正确率 29.0%，接近 Claude Opus 4.8 的 29.5%，但平均输出接近 14 万 Token；GLM-5.3 本体是 34.5%，而且输出更短。*

这张图至少说明三件事：

- Flash 明显强于 GLM-5.2；
- Flash 可以逼近 Opus 4.8 的官方私榜结果；
- **复杂 Coding 上，GLM-5.3 本体依然更强，也更省输出 Token。**

所以 Flash 的便宜来自架构效率，不代表每个任务的总 Token 都更少。

真正上线前，还是拿自己的仓库测一次成功率、总输出 Token、完成时间和人工接管次数。

## 四、长上下文为什么能便宜：不是只砍参数

GLM-5.3-Flash 的 18B 激活参数当然重要，但只看 MoE 还解释不了 1M 上下文的成本。

真正的变化在注意力结构。

![GLM-5.3-Flash架构](https://file1.kamacoder.com/i/web/20260827152743.jpg?v=20261005)

*截图说明：官方架构图左侧是三层线性注意力与一层稀疏注意力交替，右侧对比了 1M 上下文下的 KV Cache 和注意力计算量。相对 GLM-5.3，Flash 分别降低约 4.44 倍和 3.01 倍。*

线性注意力负责用状态建模抓住局部依赖，计算量不会随着上下文长度快速膨胀。

稀疏注意力再通过轻量 Indexer，从全局上下文里挑真正相关的 KV Block。

为了让 100 万 Token 下的索引也别太重，IndexPool 会把 4 组 Indexer Key 加权压成 1 组。

模型还用了 mHC，也就是 Manifold-Constrained Hyper-Connections，目标是让深层网络扩展更稳定、更高效。

官方对比里，Flash 的注意力计算量最低；但 KV Cache 仍略大于 Kimi K3 和 DeepSeek V4-Flash。

这句“不如别人”的说明反而值得保留。

**架构图讲的是服务成本，不是告诉你个人电脑能轻松跑。**

320B 权重即使做 8 Bit 量化，光权重也在 300GB 量级，还没算 KV Cache 和运行时开销。MIT 开源解决的是许可证和可控性，不会自动解决显存。

## 五、GLM-5系列终于有了原生多模态

GLM-5.3-Flash 支持图片、视频和文件输入，不是外挂一个 OCR 再交给文本模型。

它把视觉能力直接放进 Coding Loop：写前端、做游戏或搭 3D 场景时，模型可以看渲染结果，发现布局、交互和视觉问题，再继续改代码。

![GLM-5.3-Flash视觉闭环](https://file1.kamacoder.com/i/web/20260827152745.jpg?v=20261005)

*截图说明：官方案例左边是存在布局问题的初版，右边是模型经过视觉自检后的版本。重点不是“会识图”，而是观察、修改、再次验证形成闭环。*

这和前面实测的 [DeepSeek V4-Flash-Vision-Exp](./deepseek-v4-flash-vision-exp.md) 有点像：视觉不只是回答“图里有什么”，而是把截图变成下一步工具行动。

不过 GLM-5.3-Flash 发布时权重已经开放，模型名里也没有 `Exp`。它更适合直接进入本地部署、Agent 和产品选型名单。

## 六、API怎么接，有两个参数别漏

模型 ID 是 `glm-5.3-flash`，文本参数基本沿用 GLM-5.3。

图片放进 `messages[].content[]`，类型使用 `image_url`：

```json
{
  "model": "glm-5.3-flash",
  "messages": [{
    "role": "user",
    "content": [
      {"type": "image_url", "image_url": {"url": "https://example.com/ui.png"}},
      {"type": "text", "text": "检查这个页面的布局问题，并给出修改方案"}
    ]
  }],
  "thinking": {"type": "enabled", "clear_thinking": false},
  "reasoning_effort": "max",
  "temperature": 1,
  "top_p": 0.95,
  "stream": true,
  "tool_stream": true
}
```

注意两点：

- `thinking.type` 只支持 `enabled`，不能关闭思考；
- 做复杂 Agent 任务时，官方推荐同时开启 `stream` 和 `tool_stream`。

如果只是批量 OCR、分类或字段提取，不要直接照抄 `max`。

从 `low` 开始跑一批真实样本，再比较成功率和 Token。**便宜模型默认拉满思考，也可能把省下来的钱重新吐回去。**

## 七、和GLM-5.3、DeepSeek V4、Kimi K3怎么选

不站队，直接按任务算账。

| 你的任务 | 优先试谁 | 原因 |
|---|---|---|
| 最难的跨模块工程、长程排障，失败成本高 | [GLM-5.3](./glm-5-3.md) | 官方私榜上限更高，输出 Token 也更克制 |
| 高频 Coding Agent、批量工具调用、预算敏感 | GLM-5.3-Flash | 标准价约为 5.3 的十分之一，Coding Plan 额度 3 倍 |
| 截图驱动前端、Office、视频或 GUI 操作 | GLM-5.3-Flash / [Kimi K3](./kimi-k3.md) | 两者都要用真实视觉任务比完成度，别只看文本榜 |
| 已经围绕 DeepSeek API 和 Harness 搭好链路 | [DeepSeek V4](./deepseek-v4.md) | 迁移成本也是成本，先看 Flash 能否明显提高成功率 |
| 数据不能出内网、需要改权重 | GLM-5.3-Flash | MIT 权重已发布，但要先核算 320B 模型的硬件和吞吐 |
| 极难任务，一次失败代价远高于 Token | Claude Opus 4.8 或 GLM-5.3 | 这时买的是一次成功率，不是最低单价 |

我对 Flash 的判断很明确：

**它不是拿来替代所有旗舰模型，而是有资格成为大部分日常 Agent 任务的默认起点。**

默认用 Flash 跑，结果能自动验收就继续；跨模块失败、连续重试或人工接管变多，再升级到 GLM-5.3、Opus。

这比一上来所有任务都用最贵模型，更接近真实团队的成本控制。

## 写在最后

GLM-5.3-Flash 最狠的地方，不是某个柱子超过了谁。

而是把原生多模态、1M 上下文、可用的 Coding Agent 能力和 MIT 权重，压进了 18B 激活参数与十分之一价格里。

先拿它干一周真实的活。

账单和提交记录，比跑分海报诚实。

资料核对：

- [GLM-5.3-Flash 官方发布页](https://z.ai/blog/glm-5.3-flash)
- [GLM-5.3-Flash 模型文档](https://docs.z.ai/guides/vlm/glm-5.3-flash)
- [Z.ai API 价格](https://docs.z.ai/guides/overview/pricing)
- [Hugging Face 模型权重](https://huggingface.co/zai-org/GLM-5.3-Flash)
