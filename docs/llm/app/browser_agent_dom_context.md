---
title: Browser Agent怎么读取大型网页？DOM清洗、可访问性树、局部读取与Token预算
description: Browser Agent面对长文章、后台表格和无限滚动页面时，为什么不能把原始DOM全部塞进上下文？本文讲清HTML清洗、Readability正文提取、可访问性树、DOM语义分块、viewport增量读取、游标分页、页面版本校验和Token硬预算，适合Agent工程落地与面试准备。
keywords: [Browser Agent, 浏览器Agent, 网页Agent, 大型网页读取, DOM清洗, HTML清洗, Readability, 可访问性树, Accessibility Tree, DOM分块, viewport增量读取, 无限滚动, 工具分页, 页面Token预算, Browser Use, Playwright, Agent上下文工程, AI Agent面试, 大模型应用开发]
tags: [大模型应用, AI Agent, Browser Agent, 上下文工程, 工具调用, 大模型面试]
---

# Browser Agent怎么读取大型网页

<a href="https://programmercarl.com/other/project_kamaClaude.html">
  <img src="https://file1.kamacoder.com/i/web/2026-06-16_14-36-54.jpg?v=20261005" style="width:100%;cursor:pointer;" alt="KamaClaude">
</a>

上一篇[《多Agent上下文、消息和Token怎么治理》](./multi_agent_context_governance.md)讲了一个原则：大结果留在外部，Agent之间只传任务需要的摘要、状态和引用。Browser Agent也一样。

一个电商后台可能有几千行表格，一个文档站可能有几十万字，页面里还混着脚本、样式、隐藏节点和重复导航。**浏览器能加载完整网页，不等于模型应该读取完整网页。**

这篇只解决一个问题：大型网页怎么变成Agent当前一步真正能用的上下文。

## 面试官会怎么问

“Browser Agent打开一个很大的网页，原始DOM超过上下文窗口，你怎么处理？”

很多录友会回答：“先把HTML转成文本，太长就截断，或者换一个更大上下文的模型。”方向不算错，但缺了工程闭环。

直接转文本会丢掉按钮、表单、层级和状态；从尾部截断可能刚好删掉目标区域；换大窗口则把导航、样式类名和隐藏节点一起放大。结果是Token更贵了，Agent仍然找不准，也点不稳。

应该补上的，是**语义投影、区域索引、增量读取、版本校验和系统级预算**。

## 简要回答

- **先按任务选择页面表示。** 读文章、找事实，优先抽取正文；点按钮、填表单，优先读取可访问性树和可操作节点；样式、图表或语义缺失时，再局部读取DOM或截图。
- **先给地图，不给全文。** 第一次只返回标题、landmark、heading、表格和列表摘要、可操作控件，以及稳定区域引用。
- **按语义边界分块。** 文章按标题层级切，后台页面按region、form、table、dialog切，不能按HTML字符数从中间硬砍。
- **工具必须支持局部读取。** Agent根据`region_ref`、`cursor`、`limit`继续展开，滚动后只返回新增或变化区域。
- **页面引用要带版本。** URL、导航序号、DOM或语义快照版本发生变化后，旧引用必须失效或重新解析，不能盲点旧节点。
- **Token由工具层硬限制。** 同时约束单次返回、累计页面Token、分页次数和滚动次数，并给模型输出留空间。

**Browser Agent读取大网页的本质，不是把网页压成一段更短的文本，而是把页面变成一张可逐层展开的语义地图。**

<!-- drawio源文件: ./drawio/browser_agent_dom_context_01_progressive_reading.drawio；待用户审核后导出、上传并插入图片 -->

这张图回答的是：同一个大型页面如何根据阅读或操作任务生成不同的语义视图，再通过区域索引和预算闸门，只把当前所需部分送进Agent上下文；原始DOM直塞为什么会走向噪声和Token溢出。

## 为什么不能把原始DOM直接塞给模型

DOM是浏览器的实现结构，不是给模型准备的阅读稿。一个用户眼里只有“商品名称、价格、加入购物车”的卡片，DOM里可能还有十几层`div`、CSS类名、埋点属性、SVG路径、响应式副本和暂时隐藏的弹窗。

直接把`document.documentElement.outerHTML`交给模型，会同时出现四个问题：

1. **Token被结构噪声吃掉。** 标签、属性、脚本和重复组件占了大量位置，真正可见文本反而很少。
2. **页面语义被实现细节淹没。** 模型看见很多`div`，却不一定知道哪个是主内容、哪个按钮属于哪个商品。
3. **隐藏内容制造冲突。** PC和移动端副本、未展开菜单、模板节点可能同时进入上下文。
4. **节点引用很快过期。** 页面一重渲染，旧DOM路径或序号就可能指向另一个元素。

![原始DOM噪声困境](https://file1.kamacoder.com/i/web/20260911094725.jpg?v=20261005)

基础清洗可以先移除`script`、`style`、`noscript`、注释、纯装饰节点和重复模板，再规范空白与URL；但链接目标、表头、label、ARIA属性、控件状态和iframe边界要保留。也不能把“当前viewport之外”直接等同于“无关”，虚拟列表的目标内容可能要滚动后才出现。

所以第一步不是“截多少HTML”，而是先问：**当前任务需要阅读页面，还是操作页面？**

## Readability和可访问性树为什么不能二选一

两者解决的问题不同。

| 页面表示 | 最适合 | 保留什么 | 容易丢什么 |
|---|---|---|---|
| Readability正文 | 新闻、博客、文档详情页 | 标题、正文、作者、摘要等阅读内容 | 按钮状态、复杂表单、后台布局 |
| 可访问性树 | 登录、下单、审批、搜索等交互页面 | role、accessible name、层级和部分控件状态 | CSS视觉关系、画布内容、无障碍标注差的节点 |
| 局部DOM | 需要属性、结构或精确调试的区域 | 节点属性、父子结构、业务标记 | Token开销仍高，容易绑定实现细节 |
| 局部截图 | 图表、Canvas、视觉位置判断 | 最终渲染效果和空间关系 | 精确文本、完整状态和稳定定位 |

[Mozilla Readability](https://github.com/mozilla/readability)能从DOM中提取处理后的正文HTML、纯文本、标题和作者等字段，适合“这篇文章讲了什么”。但它不是后台系统解析器，拿它处理多列表格和复杂表单，关键交互很可能被当成噪声删掉。

可访问性树则是DOM派生出的语义树。[Chrome官方文档](https://developer.chrome.com/docs/devtools/accessibility/reference)说明，它只保留对辅助技术有用的页面节点；按钮、标题、输入框等节点通常带有role和accessible name。[Playwright的`getByRole()`](https://playwright.dev/docs/locators#locate-by-role)也是利用这类面向用户的语义定位元素。

但可访问性树也不是“更干净的完整DOM”。网站无障碍标注差、内容画在Canvas里，或者信息主要靠颜色与位置表达时，它可能不完整。

**正确方案不是选一个格式打天下，而是默认用最省Token的语义表示，缺信息时再局部升级。**

## 第一次打开页面，为什么只应该返回“地图”

假设Agent要在一份300页的API文档里找“重试策略”。第一次工具调用没必要返回300页正文，它更需要这样的页面地图：

```json
{
  "page_version": "nav-18:sem-7",
  "title": "支付平台API文档",
  "regions": [
    {"ref": "r1", "role": "navigation", "summary": "12个一级目录"},
    {"ref": "r7", "role": "main", "headings": ["鉴权", "错误处理", "重试策略"]}
  ]
}
```

Agent看到`重试策略`位于`r7`，再调用`read_region(ref="r7", query="重试策略")`。这叫**先概览，再下钻**。

页面地图至少保留四类锚点：

- landmark：`main`、`navigation`、`form`、`dialog`等大区域；
- heading path：当前内容位于哪个标题层级；
- content summary：表格行数、列表项数、正文摘要；
- action ref：按钮、链接、输入框等可操作节点的临时引用。

这和[《长文档与代码怎么检索》](./long_document_code_retrieval.md)里的Parent-Child思路很像：小表示负责定位，大内容按需返回。区别在于网页会实时变化，所以引用还必须绑定页面版本。

## DOM分块为什么不能按字符数硬切

固定每4000字符切一块，看起来简单，实际会把语义切碎。

一张表可能表头在第一块、目标行在第二块；一个表单可能label在上一块、input在下一块；文章的标题和解释也可能分家。模型拿到的每一块都“有字”，却不知道这些字之间是什么关系。

更稳的分块顺序是：

1. 先按`main`、`nav`、`form`、`table`、`dialog`等区域切；
2. 正文区域再按`h1`到`h6`的标题树切；
3. 表格保留表头，并按行分页；
4. 列表保留列表语义，并按item游标分页；
5. 单块仍超预算时，才在段落或行边界继续切。

每个块都要带最小元数据：`region_ref`、标题路径、块类型、序号、总量估计和`page_version`。这样模型拿到第3页表格时，仍然知道它属于哪个页面、哪张表、前面还有多少内容。

## 工具分页应该怎么设计

“返回全部结果，框架自动截断”是最差的工具契约。截断发生后，Agent甚至不知道结果是否完整。

一个可控的读取工具，参数和返回值都要显式分页。下面省略了正文，只展示控制字段：

```json
{
  "request": {"region_ref": "r7", "query": "重试策略", "cursor": null, "max_tokens": 1800},
  "page_version": "nav-18:sem-7",
  "returned_tokens": 1460,
  "truncated": true,
  "next_cursor": "r7:p3"
}
```

这里的`truncated`和`next_cursor`很关键。它们告诉Agent：“这不是全文，只是当前页。”

工具还应该支持`find_in_page(query)`先返回命中区域和短片段，让Agent决定读哪一块，而不是从第1页机械翻到第80页。

## 无限滚动和SPA为什么必须增量读取

大型页面经常不是一次加载完成的。消息列表滚到底才加载下一页，表格只渲染viewport附近几十行，单页应用点击标签后URL甚至不变。此时“一次DOM快照”只代表某个时刻、某个视口的状态。

![动态页面旧引用风险](https://file1.kamacoder.com/i/web/20260911094726.jpg?v=20261005)

增量读取要记录三样东西：

- 当前`page_version`和导航序号；
- 已见区域或item的指纹集合；
- 本次滚动、点击后新增、删除和变化的区域。

操作完成后，工具优先返回diff，而不是再次返回整页。如果目标没出现，Agent再决定是否继续滚动；达到`max_scrolls`、连续两次无新增，或页面Token预算不足时，必须停止。

[Playwright官方文档](https://playwright.dev/docs/locators)也提醒，动态列表变化时直接取得所有元素会产生不稳定结果；面向交互的locator会在每次动作前重新定位当前DOM。工程上可以借鉴这个原则：**动作引用用于表达意图，执行前必须在最新页面状态上重新解析并校验唯一性。**

## 页面Token预算怎么设成硬约束

预算不能只写在Prompt里说“请节省Token”。模型决定是否继续读，工具层决定它最多能读多少。先算本轮可用页面预算：

```text
页面可用Token = 模型窗口
              - 系统规则与工具Schema
              - 当前任务和必要历史
              - 输出预留
              - 安全余量
```

再把它拆成四道限制：

- `max_tokens_per_read`：一次读取最多返回多少；
- `max_page_tokens`：本次任务累计能读取多少页面内容；
- `max_pages`或`max_scrolls`：最多分页、滚动多少次；
- `min_output_reserve`：无论如何都不能挪用的回答与验收预算。

预算快耗尽时，优先级应该是：精确命中区域 > 当前可视区域 > 相邻上下文 > 低相关页面扩展。订单号、按钮状态、表头和证据引用不能为了省Token被摘要掉。

关于窗口为什么要预留输出空间，可以先看[《Context Engineering入门》](./context_engineering.md)。

## 一套可落地的读取链路

可以把Browser Agent的读取层收成两个入口：

```python
def observe(page, task, budget):
    wait_for_stable_enough(page)
    version = fingerprint(page)
    view = extract_readable(page) if task.is_reading else extract_accessible(page)
    outline = build_region_index(view, version)
    return fit_budget(outline, budget.overview_tokens)

def read_more(page, ref, cursor, budget):
    assert_current_version(page, ref.page_version)
    return semantic_page(resolve_ref(page, ref), cursor, budget.remaining_page_tokens)
```

`wait_for_stable_enough`不是固定`sleep(3)`。它应该围绕任务等到目标区域出现、关键请求结束或loading状态消失，同时有超时上限。广告轮播和心跳请求可能永远不停，等“整个页面绝对静止”会把Agent卡死。

这条链路也要遵守[《工具设计决定Agent上限》](./agent_tool_design.md)的边界：工具返回的不只是内容，还要返回完整性、版本、下一步入口和错误类型，让Agent知道能否继续行动。

## 怎么评估读取策略是不是真的有效

只看Token下降不够。把正文砍没了，当然最省。

至少同时记录四组指标：

- **定位质量**：目标区域召回率、目标控件唯一定位率；
- **任务效果**：阅读问答准确率、网页任务完成率、误点击率；
- **资源消耗**：每步页面Token、累计页面Token、读取和滚动次数；
- **状态稳定性**：过期引用率、重定位成功率、动态页面重复读取率。

测试集要覆盖长文章、分页表格、虚拟列表、无限滚动、iframe、弹窗和无障碍标注较差的页面。还要故意让页面在观察后重渲染，验证旧引用会被拒绝，而不是悄悄点到别处。

**真正要优化的是每1000个页面Token带来的任务成功率，不是把页面压得越短越好。**

## 知识拓展

**Q1：只给Agent截图，能绕过DOM太大的问题吗？**

不能。截图适合补充布局、图表和Canvas，但长页面仍要滚动，也会产生视觉Token和定位成本。阅读文本、填写表单时，语义树通常更稳定。

**Q2：可访问性树比DOM稳定吗？**

通常更接近用户语义，也减少了样式节点，但不是永久ID。页面重渲染后仍应重新定位，尤其不能长期保存“第17个按钮”这种位置引用。优先使用role、accessible name、所属region和业务ID组合校验。

**Q3：Readability提取完，还需要做安全处理吗？**

需要。Mozilla明确说明Readability不负责净化不可信输入；重新渲染HTML时应配合sanitizer和CSP。给模型读取时，也要防止网页文字冒充系统指令。

**Q4：什么时候必须回退到局部DOM？**

当可访问性树缺少必要属性、控件没有可辨识名称、需要读取`data-*`业务标记，或要理解表格跨行跨列结构时，再读取目标region的DOM。范围应该由已定位区域限定，不能因为一个节点缺信息就回退整页HTML。

**Q5：面试时怎么用一句话收住？**

可以说：先根据阅读或操作任务生成正文视图或可访问性视图，首次只返回页面地图；Agent再通过区域引用和游标局部读取，动态页面用版本与diff更新，工具层限制单次和累计Token，并用任务成功率、过期引用率和单位Token收益验证。

把整个DOM塞给模型，只是把浏览器的负担转嫁给上下文窗口。

真正能跑稳的Browser Agent，手里拿的不是网页全文，而是一张随页面变化、可以继续下钻的地图。
