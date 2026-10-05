---
title: DeepSeek终于能看图了：V4-Flash-Vision-Exp识图、图表、ASCII和SVG实测，Harness多模态怎么用？
description: DeepSeek发布实验性多模态视觉理解模型DeepSeek-V4-Flash-Vision-Exp，DeepSeek Harness同步支持图文输入。本文实测复杂网页OCR、Benchmark表格识别与计算、ASCII重绘和SVG生成，记录默认思考模式跑满12000 Token却无正文的真实失败，并说明图片Token、API价格和使用边界。
keywords: [DeepSeek V4 Flash Vision Exp, DeepSeek-V4-Flash-Vision-Exp, DeepSeek多模态, DeepSeek视觉模型, DeepSeek看图, DeepSeek图片识别, DeepSeek OCR, DeepSeek图表识别, DeepSeek Harness多模态, dsh 0.1.1-rc.2, DeepSeek图片Token, DeepSeek API价格, 截图生成SVG, AI编程, 多模态Agent, Opus 4.8]
tags: [DeepSeek, 多模态, 视觉模型, DeepSeek Harness, AI编程, Agent, 大模型评测]
---

# DeepSeek终于能看图了：复杂截图、图表、ASCII和SVG实测

大家好，我是卡哥。

本周四，DeepSeek Harness 先迎来了一波更新，其中最关键的一项，就是补上了多模态输入。

普通对话可以传图，`/goal`、`/plan` 这些常用命令也开始接收图文输入。

刚看到更新时，我还以为 Harness 只是把入口准备好，让我们接第三方视觉模型。

结果第二天，DeepSeek 就发布了自己的实验性视觉理解模型：`DeepSeek-V4-Flash-Vision-Exp`。

好家伙，DeepSeek 终于能看图了。

现在回头看，Harness 这次更新明显是在给自家模型铺路。前面那篇 [DeepSeek Harness 安装教程](./deepseek-harness-guide.md) 讲的是怎么把 Agent 跑起来，这篇直接实测它的新眼睛。

## 先确认一件事：这次不是第三方套壳

我测试时，npm 上最新的 Harness 已经是 `0.1.1-rc.2`。官方 `/models` 接口也直接返回了 `deepseek-v4-flash-vision-exp`。

![Vision实测环境](https://file1.kamacoder.com/i/web/20260823165020.png?v=20261005)

所以这次走的是 DeepSeek 官方 API，不是外挂一个视觉模型识图，再把文字转给 [DeepSeek V4-Flash](./deepseek-v4-flash-official.md)。

官方 8 月 21 日的[发布说明](https://api-docs.deepseek.com/zh-cn/news/news260821)也把定位写得很清楚：这是一个实验模型，纯文本能力与 V4-Flash 正式版持平，新增的是视觉理解能力。

当前支持 Chat Completions、Anthropic Messages 和 Responses API。图片可以用三种方式传入：

- base64 内联；
- 外部图片 URL；
- 先传到免费的 Files API，再用 `file_id` 引用。

JPEG、PNG、GIF、WebP 都支持。只有 Vision-Exp 接受图片，普通 Flash 和 Pro 收到图片仍会返回 400。

## 官方跑分很漂亮，但先别急着下结论

官方公布了同一套模型在文本 Agent 和多模态 Agent 基准上的成绩。

![官方多模态跑分](https://file1.kamacoder.com/i/web/20260823165024.png?v=20261005)

从图里看，Vision-Exp 的文本 Agent 能力确实没有因为加视觉而掉下去。

Terminal Bench 2.1 从 82.7 到 83.9，DeepSWE 从 54.4 到 59.3，Toolathlon-Verified 从 70.3 到 75.9。只有 CyberGym 从 76.7 降到 75.3。

多模态项目上提升更明显：ApexBench 从 26.2 到 36.5，Agents' Last Exam 从 25.2 到 27.3。

不过官方脚注也说明，文本 Code Agent 测试使用 DeepSeek Harness 极简模式、`max` 档位、`temperature=1.0`、`top_p=0.95`。这能说明模型上限，不能替代你自己的仓库和验收。

官方还展示了 PPT、Harness 官网二次创作和前端 Mini Demo 三个案例。

![官方PPT生成案例](https://api-docs.deepseek.com/zh-cn/img/v4_260821_case1.png)

![Harness网页改造](https://api-docs.deepseek.com/zh-cn/img/v4_260821_case2.gif)

![黏土怪物Demo](https://api-docs.deepseek.com/zh-cn/img/v4_260821_case3.gif)

看着都不错，但官方 Demo 永远是挑过的。接下来还是自己跑。

## 第一轮：复杂网页截图，OCR 到底稳不稳

我先打开自己的 [卡码笔记](https://notes.kamacoder.com/)，截了一张 1680×952 的首屏。

这张图不只是一个大标题。里面有中英文混排、小字号导航、十个导航入口、八个蓝色链接，以及 `C++`、`agent`、`rag`、`transformer` 这种 OCR 很容易抄错的技术名词。

![卡码笔记网站首屏](https://file1.kamacoder.com/i/web/20260823172734.png?v=20261005)

提示词里我明确要求它不要搜索，只看图识别站点、导航、主副标题、八个链接标题和三个易错细节。

站点名称、主标题、副标题和导航语义都识别对了。八个蓝色链接里，七个完整：

- 计算机基础面试题
- 大模型应用开发面试题
- C++面试题
- Java面试题
- Go面试题
- 代码随想录
- 卡码投递表

唯一的漏字是把「真实面经（含大模型）」截成了「真实面经」。`C++` 和说明文字里的 `agent`、`rag`、`transformer` 都抄对了。

![网页OCR结果](https://file1.kamacoder.com/i/web/20260823172735.png?v=20261005)

这轮我的结论是：**普通网页截图里的中文、小字号和中英文技术名词已经能用，但括号里的补充信息仍可能漏掉。**

关闭思考后，这个任务输入 477 Token、输出 555 Token，没有为一段 OCR 结果绕很远的路。

## 第二轮：让它读官方跑分图，再自己算差值

只会 OCR 还不够。

我把前面的官方跑分图原样丢回 Vision-Exp，要求它抄出全部 11 行数据，再计算 Vision-Exp 相比 Flash 的差值，找出超过 Opus-4.8 的项目。

这轮它不但把 11 行全部抄对，减法也全部正确。

![图表识别结果](https://file1.kamacoder.com/i/web/20260823165028.png?v=20261005)

它算出的最大提升是 ApexBench，`36.5 - 26.2 = 10.3`。

超过 Opus-4.8 的项目有三个：DeepSWE、Agents' Last Exam、ZeroBench。与 Opus 差距不到 1 分的项目是 Toolathlon-Verified 和 Chartography。

还有个细节我挺喜欢：图里的 `**` 没有解释含义，它明确说“不知道，不做猜测”。

**识别图表不难，抄对数字、做对计算、遇到缺失信息不脑补，才是这轮真正有用的地方。**

## 第三轮：看着 K Logo，能不能画成 ASCII

接下来测一点更抽象的。

我把卡码笔记首页的 K Logo 单独拿出来，让它限制在 48 个等宽字符内，只用纯 ASCII 重新画。

![卡码笔记K Logo](https://file1.kamacoder.com/i/web/20260823172737.jpg?v=20261005)

它确实画出了一块代码窗口，括号、斜杠和下方横线都能辨认。

但如果只看相似度，这轮明显不及格。原图最有辨识度的 K 形斜面直接丢了，输出更像一个普通的矩形代码图标。

![ASCII重绘结果](https://file1.kamacoder.com/i/web/20260823172738.png?v=20261005)

所以“看懂图片”和“用字符重新表现图片”，明显不是同一个难度。

DeepSeek 能抓住内部元素，但压缩成 ASCII 后，连品牌图形最关键的外轮廓都可能保不住。

## 第四轮：它到底能不能制作图片

这里要先把边界说清楚。

`DeepSeek-V4-Flash-Vision-Exp` 是视觉理解模型，不是文生图模型。它不会像图片生成模型那样直接返回一张 PNG。

但它可以看图后生成 SVG、HTML、CSS 或 Canvas 代码，再由浏览器或渲染器把代码变成图片和界面。

我继续使用同一张 K Logo，要求它只用 SVG 原生图形重画，不得嵌入原图、base64、外链和文字。

最终生成的是一份 1.7 KB 的纯 SVG。

![SVG重绘对比](https://file1.kamacoder.com/i/web/20260823172739.png?v=20261005)

深蓝色 K 形主体、白色代码窗口、括号、斜杠和两条横线都保住了，说明它确实理解了原图结构和配色。

问题主要在比例：右侧斜面更宽，代码窗口更大，底部留白和原图也不完全一致。

所以这轮更准确的评价是：**能根据图片制作可编辑矢量图，但还不是像素级复刻。**

拿它做图标草稿、示意图和网页占位图没问题；品牌 Logo、人物肖像和要求严格的视觉资产，仍然要设计师或更强的图像生成模型收尾。

## 一个很烧钱的坑：画图任务别默认开思考

前面的 K Logo ASCII 和 SVG，我其实都跑了两遍。

第一遍使用默认思考模式。两个请求都消耗了 11999 个 reasoning Token，几乎把我设置的 12000 输出 Token 全部用完，`finish_reason` 是 `length`，最终正文却是空的。

第二遍关闭思考模式，ASCII 总计 597 Token，SVG 总计 1081 Token，很快就正常返回。

这个差异太夸张了。

**视觉任务不是越会想越好。** OCR、格式转换、SVG 重绘这类目标明确的任务，先关思考；只有图表推理、复杂界面分析和多步 Agent 任务，再考虑打开思考。

否则模型可能把预算全部花在内部推理上，用户连一个字符都拿不到。

## 一张图片最多384 Token，价格还是Flash档

官方[图像理解指南](https://api-docs.deepseek.com/zh-cn/guides/vision#token-usage)说明，图片会先按比例缩放，再换算成 Token。

小图会放大，大图会缩小到总像素约等于 800×800。**每张图片最多占 384 Token。** 2000×2000 和 5000×5000 的大图缩放后，图片 Token 相同。

单个请求最多传 600 张图；图片单边最长 8192 像素，达到 15 张及以上时降为 4096 像素。

Vision-Exp 与 V4-Flash 同价。按 2026 年 8 月 23 日官方[价格页](https://api-docs.deepseek.com/zh-cn/quick_start/pricing)：

| 计费项 | 空闲时段 | 高峰时段 |
| --- | ---: | ---: |
| 缓存命中输入 | 0.05 元/百万Token | 0.10 元/百万Token |
| 缓存未命中输入 | 1.5 元/百万Token | 3.0 元/百万Token |
| 输出 | 4.5 元/百万Token | 9.0 元/百万Token |

高峰时段是工作日 9:00–12:00、14:00–18:00，其余为空闲时段。

我这次四组有效实验，加上两次接近 12000 Token 的失败请求，按高峰价粗算约 0.27 元。

便宜是真便宜，但默认思考模式如果放进批量图片任务，浪费也会被并发放大。

## 写在最后

这次不是 DeepSeek 多了一个“上传图片”按钮。

真正补上的，是 AI 编程和 Agent 工作流里很关键的一块能力：**看见截图、理解截图，再把截图里的信息变成文字、计算结果和可执行代码。**

实测下来，网页 OCR 和图表理解已经很能打，SVG 重绘能用，ASCII 和高相似度视觉还原还有明显提升空间。

模型名字里还带着 `Exp`，这个结果很合理。

但从这次开始，DeepSeek 至少不再是一个只能靠别人描述世界的文本模型了。

它终于有自己的眼睛了。
