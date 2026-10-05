---
title: Claude Code怎么读懂大代码库？Agent搜索、CLAUDE.md、Hooks、Skills、MCP和LSP一篇讲明白
description: 基于 Anthropic 博客 How Claude Code works in large codebases，本文拆解 Claude Code 在百万行代码库、Monorepo、遗留系统和多仓库架构中的工作方式，讲清 Agentic Search 和 RAG 索引的区别，以及 CLAUDE.md、Hooks、Skills、Plugins、MCP、LSP、Subagents 如何组成 Claude Code 的工程支架，帮助团队在大代码库里稳定落地 AI 编程。
keywords: [Claude Code大代码库, Claude Code monorepo, Claude Code最佳实践, Claude Code企业落地, Agentic Search, CLAUDE.md, Claude Code Hooks, Claude Code Skills, Claude Code Plugins, MCP, LSP, Subagents, AI编程, AI Coding, 大模型编程工具, 大代码库上下文管理]
tags: [Claude Code, AI编程, 大代码库, Agentic Search, 大模型入门]
---

# Claude Code怎么读懂大代码库？Agent搜索、CLAUDE.md、Hooks、Skills、MCP和LSP一篇讲明白

录友们好，今天继续聊 Claude Code。

前面我们刚写过一篇 [CLAUDE.md到底怎么写](./claude_md.md)，讲的是一个项目怎么把命令、规范、禁区、验证流程沉淀给 Claude Code。

但如果项目变大呢？

不是一个小仓库，不是几个组件。

而是几百万行代码的 monorepo、十几年前留下来的老系统、几十个微服务、Java、C++、PHP、前端、后端、移动端全混在一起。

这时候很多人第一反应是：

**这么大的代码库，Claude Code 是不是得先建索引？是不是得先把整个仓库向量化？是不是没有 RAG 就读不懂？**

Anthropic 在 2026 年 5 月 14 日发了一篇博客：How Claude Code works in large codebases: Best practices and where to start：https://claude.com/blog/how-claude-code-works-in-large-codebases-best-practices-and-where-to-start

这篇文章讲得很有意思。

它不是在吹模型参数，也不是在说"Claude 比谁都聪明"。

它真正想说的是：

**Claude Code 在大代码库里能不能跑稳，不只看模型，还看你有没有给 Agent 铺好路。**

> 不少读者问我，如何充值Claude会员，我在这里篇单独讲一下：[国内Claude充值会员的方法](../../qita/0002.claudepay.md)

模型是发动机。

但大代码库里，光有发动机不够。

你还得有地图、路标、交通规则、工具接口、自动检查、团队维护机制。

这套东西，Anthropic 称为 harness。

其实关于harness，我在这里也讲过了： [讲透Harness Engineering](https://mp.weixin.qq.com/s/yP3Clk7EdzdmzV4ImrAxug)

今天这篇，我们就基于 Anthropic 这篇博客，把 Claude Code 在大代码库里的工作方式讲清楚。

## 目录

1. [为什么大代码库不是小项目放大版？](#一为什么大代码库不是小项目放大版)
2. [Claude Code 不是靠索引背下整个仓库](#二claude-code-不是靠索引背下整个仓库)
3. [Agentic Search：像工程师一样边搜边读](#三agentic-search像工程师一样边搜边读)
4. [大代码库的关键不是看更多，而是找得准](#四大代码库的关键不是看更多而是找得准)
5. [Harness：模型之外真正影响体验的工程支架](#五harness模型之外真正影响体验的工程支架)
6. [CLAUDE.md：大代码库的第一层导航](#六claudemd大代码库的第一层导航)
7. [Hooks：把提醒模型变成自动执行](#七hooks把提醒模型变成自动执行)
8. [Skills：不要把所有专家知识塞进上下文](#八skills不要把所有专家知识塞进上下文)
9. [Plugins、MCP、LSP、Subagents 分别解决什么问题？](#九pluginsmcplspsubagents-分别解决什么问题)
10. [大代码库应该按什么顺序配置 Claude Code？](#十大代码库应该按什么顺序配置-claude-code)
11. [团队落地：为什么需要一个 Agent 管理人？](#十一团队落地为什么需要一个-agent-管理人)
12. [普通开发者怎么开始？](#十二普通开发者怎么开始)

## 一、为什么大代码库不是小项目放大版？

小项目里用 Claude Code，很简单。

你让它修一个 bug，它搜一下文件，读几个组件，改完跑测试。

路径短，文件少，命令也少。

但大代码库不是这样。

一个大代码库里，可能会有这些问题：

- 每个子目录都有不同的构建命令
- 前端、后端、脚本、部署工具混在一个仓库
- 同名函数在不同语言里出现几十次
- 老代码没有统一目录结构
- 微服务分散在多个仓库
- 构建日志一跑就是几千行
- 全量测试根本跑不动
- 某些目录是生成代码，根本不该读

这不是"文件变多了"这么简单。

这是信息密度、历史包袱、团队约定、工具链复杂度一起上来了。

所以大代码库里，Claude Code 最大的问题不是不会写代码。

而是它一开始不知道：

**从哪里看，读哪些文件，用什么命令验证，哪些地方不能碰。**

如果这些都靠 Claude 自己猜，它会浪费大量上下文。

更糟的是，它可能猜错方向。

所以大代码库的第一原则是：

**不要让 Agent 在仓库里盲搜。**

你要给它入口。

![图1：大代码库让 Agent 迷路的根因不是文件多，而是入口太多](https://file1.kamacoder.com/i/web/20260527192245_claude_large_codebase_01_complexity_compressed.png?v=20261005)

## 二、Claude Code 不是靠索引背下整个仓库

很多 AI 编程工具喜欢讲代码索引、Embedding、RAG。

这个思路很自然。

仓库太大，模型上下文放不下，那就先把代码切块、向量化、建索引，用户提问时再召回相关片段。

这套方案在很多场景有用。

但 Anthropic 这篇博客里提到一个关键问题：

**大规模活跃代码库里，索引很容易过期。**

几千个工程师每天都在提交代码。

函数今天改名，模块明天迁移，旧服务后天删掉。

如果索引管线跟不上，模型拿到的就是旧世界的地图。

它可能召回一个两周前已经改名的函数，也可能引用一个上个 sprint 已经删掉的模块。

更麻烦的是，召回结果本身不会主动告诉你："我过期了。"

Claude Code 的思路不一样。

它更像一个真实工程师：

- 在本地文件系统里看目录
- 用 grep 搜关键词
- 读当前最新文件
- 顺着引用关系继续找
- 根据测试和报错再调整方向

它不是先把整个仓库上传到某个中心索引里。

它是在开发者机器上的实时代码库里工作。

![图2：活跃代码库里索引容易和现场错位，Claude Code 更依赖当前工作区实时搜索](https://file1.kamacoder.com/i/web/20260527192246_claude_large_codebase_02_index_drift_compressed.png?v=20261005)

这就是 Anthropic 说的 agentic search。

不是"先把答案搜出来给模型"。

而是让 Agent 自己根据当前任务，一步一步决定下一步该搜什么、读什么、验证什么。

## 三、Agentic Search：像工程师一样边搜边读

Agentic Search 听起来很高级，其实你可以把它理解成：

**不是一次性检索答案，而是边行动边缩小范围。**

比如你让 Claude Code 修一个支付失败问题。

它不应该上来读整个支付系统。

更合理的路径是：

1. 先看报错关键词
2. 用 grep 找相关接口
3. 读调用链上的关键文件
4. 找到支付状态转换逻辑
5. 看测试怎么覆盖
6. 修改最小范围
7. 跑对应测试

![图3：Agentic Search 不是一次性搜索答案，而是根据新证据一步步缩小范围](https://file1.kamacoder.com/i/web/20260527192247_claude_large_codebase_03_agentic_search_compressed.png?v=20261005)

这和工程师查 bug 很像。

工程师也不会打开全仓库从第一行读到最后一行。

他会根据线索搜索。

所以 Claude Code 的优势不是"一次看完整个仓库"。

它的优势是：

**能用工具持续探索，并把探索结果转成下一步行动。**

但这里有一个前提。

Agentic Search 需要起点。

如果你只说："帮我优化这个十亿行代码库。"

那谁也救不了。

Claude Code 也会在上下文窗口耗尽之前迷路。

所以大代码库必须给 Claude Code 几类导航信息：

- 目录地图
- 根 CLAUDE.md
- 子目录 CLAUDE.md
- 局部测试命令
- 忽略规则
- LSP 符号导航

这些东西不是锦上添花。

它们是大代码库里让 Agent 找准方向的基础设施。

## 四、大代码库的关键不是看更多，而是找得准

很多录友一听大代码库，就觉得上下文窗口要越大越好。

窗口大当然有用。

但大代码库里，真正的问题不是"放不下全部代码"。

因为无论窗口多大，你都不应该把全部代码塞进去。

真正的问题是：

**Claude Code 能不能在读很少文件的情况下，找到真正相关的那几个文件。**

这就像你查线上 bug。

你不需要把所有日志都看一遍。

你需要知道哪个服务、哪个接口、哪个配置、哪个版本最可能相关。

所以大代码库要做的是上下文治理：

- 该加载的规则提前加载
- 不相关的目录不要读
- 生成代码和构建产物要排除
- 测试命令要按子目录收敛
- 同名函数要靠 LSP 区分
- 大范围探索交给 subagent

Anthropic 文章里提到一个很重要的点：

**太多上下文会降低性能，太少上下文会让 Claude 盲搜。**

这句话很适合放到团队规范里。

大代码库不是让 Claude Code 看更多。

而是让它每一步都看得更准。

![图4：上下文治理要避免太少导致盲搜，也要避免太多稀释注意力](https://file1.kamacoder.com/i/web/20260527192249_claude_large_codebase_04_context_governance_compressed.png?v=20261005)

## 五、Harness：模型之外真正影响体验的工程支架

很多人评价 Claude Code，第一反应是看模型。

用的是 Sonnet 还是 Opus？

跑 benchmark 排名怎么样？

这些当然重要。

但 Anthropic 这篇博客说得很直接：

**Claude Code 的效果，不只由模型决定，还由模型周围的 harness 决定。**

什么是 harness？

简单说，就是 Claude Code 周围这套工程支架：

- `CLAUDE.md`
- Hooks
- Skills
- Plugins
- MCP servers
- LSP
- Subagents

模型负责推理。

harness 负责把模型放到真实工程环境里。

没有 harness，Claude Code 就像一个很聪明的新同事，但没人告诉他项目规则，也不给他工具权限，还让他自己猜构建命令。

有 harness，Claude Code 才像一个接入了团队工程体系的 Agent。

![图5：同一个任务有没有 Harness，Claude Code 的执行路径会完全不同](https://file1.kamacoder.com/i/web/20260527192250_claude_large_codebase_05_harness_paths_compressed.png?v=20261005)

接下来我们一个个讲。

## 六、CLAUDE.md：大代码库的第一层导航

`CLAUDE.md` 永远应该先做。

原因很简单：

它是 Claude Code 每次进入项目时最稳定的上下文入口。

在大代码库里，根 `CLAUDE.md` 不应该写成百科全书。

它应该 lean and layered。

也就是：

**根文件写大图和关键坑，子目录文件写局部规则。**

根文件适合写：

- 仓库整体结构
- 顶层目录各自负责什么
- 全局禁区
- 包管理器
- 全局安全要求
- 重要文档入口

子目录文件适合写：

- 当前模块怎么启动
- 当前模块怎么测试
- 当前模块有什么本地约定
- 当前模块常见坑

不要把所有细节都塞进根文件。

根文件太长，每个任务都会背着一堆无关规则。

比如你只改前端登录页，却每次都加载后端数据库迁移规则、移动端构建说明、部署平台历史问题。

这就是上下文污染。

所以大代码库更需要分层 `CLAUDE.md`。

![图6：根 CLAUDE.md 管方向，子目录 CLAUDE.md 和 Skills 负责局部知识按需加载](https://file1.kamacoder.com/i/web/20260527192251_claude_large_codebase_06_context_routing_compressed.png?v=20261005)

这里和我们上一篇文章是连着的。

上一篇讲 `CLAUDE.md` 怎么写。

这篇要再强调一句：

**大代码库里，CLAUDE.md 不只是规则文档，它是 Agent 的导航系统。**

## 七、Hooks：把提醒模型变成自动执行

很多团队刚开始用 Claude Code，会喜欢在 `CLAUDE.md` 里写：

- 修改代码后记得格式化
- 提交前记得跑 lint
- 不要忘了更新文档
- 会话结束后总结这次踩过的坑

这些有用吗？

有用，但还不够。

因为只写在文档里，本质还是提醒模型。

而 Hooks 的价值是：

**把提醒模型，变成确定执行。**

比如格式化。

你当然可以写："修改代码后请运行 formatter。"

但更稳的方式是 hook 在合适的时机自动执行格式化。

再比如 lint。

你当然可以让 Claude 记得跑。

但 hook 自动跑 lint，比让模型记住更稳定。

Anthropic 文章里还提到一个更有意思的用法：

Hooks 不只是拦截错误，还可以让配置自我改进。

比如 stop hook 在会话结束时回看这次过程：

- Claude 今天是不是绕了远路？
- 是不是因为某个目录规则不清楚？
- 是不是某个测试命令应该写进子目录 CLAUDE.md？

如果是，就提出 `CLAUDE.md` 更新建议。

这样项目规则就不是一次性写完。

而是随着团队使用不断迭代。

这才是大团队落地 AI 编程需要的能力。

![图7：Hooks 把提醒模型变成自动执行，并把经验反哺到项目规则里](https://file1.kamacoder.com/i/web/20260527192252_claude_large_codebase_07_hooks_feedback_compressed.png?v=20261005)

## 八、Skills：不要把所有专家知识塞进上下文

大代码库里，会有很多专家知识。

比如：

- 安全审查怎么做
- 支付模块怎么部署
- 文档更新流程是什么
- 某个内部平台怎么查数据
- 某类日志怎么定位

如果你把这些全部写进 `CLAUDE.md`，它会爆炸。

而且很多任务根本用不到这些知识。

这时候就该用 Skills。

Skills 的关键是按需加载。

只有当任务需要安全审查时，才加载安全审查 skill。

只有当任务涉及文档处理时，才加载文档处理 skill。

只有当任务进入支付目录时，才加载支付部署 skill。

这就是 Anthropic 提到的 progressive disclosure。

我把它翻译得口语一点：

**别一上来把所有专家都请进会议室。谁用得上，再叫谁进来。**

当然，文章里不能真的这么写得太花。

落到工程上就是：

**稳定通用规则放 CLAUDE.md，专门任务流程放 Skills。**

这条边界很重要。

很多团队把 `CLAUDE.md` 写成巨型专家手册，最后 Claude 每次启动都要读一堆无关知识。

这不是增强。

这是拖慢。

## 九、Plugins、MCP、LSP、Subagents 分别解决什么问题？

Anthropic 文章里还讲了几个组件。

这些词很多录友第一次看会晕。

我们用一句话分别讲清楚。

**Plugins：把好用配置发给全团队。**

一个团队里，某个工程师配好了 hooks、skills、MCP。

如果只存在他电脑上，那就是个人经验。

如果打包成 plugin，新人第一天装上，就能拿到同样能力。

所以 Plugins 解决的是分发问题。

**MCP：让 Claude Code 连接内部工具。**

比如内部文档、工单系统、指标平台、发布平台、结构化搜索。

Claude Code 本地文件系统看不到这些。

MCP servers 就是把这些内部能力变成 Claude 可以调用的工具。

所以 MCP 解决的是外部工具接入问题。

**LSP：让 Claude Code 按符号找代码。**

大代码库里，单纯 grep 经常会出问题。

一个 `handle()`、`init()`、`process()`，可能全仓库几千个。

Grep 返回一堆字符串匹配，Claude 还要继续读文件判断哪个是真相关。

LSP 不一样。

它能像 IDE 一样做 go to definition、find references。

也就是按符号关系找，而不是按字符串找。

对于 C、C++、Java、C# 这种大工程，LSP 非常有价值。

**Subagents：把探索和编辑拆开。**

大代码库里，探索会产生大量噪音。

读目录、查引用、看历史、对比模块，这些都很吃上下文。

更稳的做法是让子 Agent 先做只读探索，最后把结论交给主 Agent。

主 Agent 再基于结论编辑代码。

这样主上下文不会被探索过程污染。

这和我们之前讲上下文管理是同一个逻辑：

**探索噪音隔离，关键结论回传。**

![图8：Plugins、MCP、LSP、Subagents 不是同一类东西，它们分别解决分发、接入、定位和隔离问题](https://file1.kamacoder.com/i/web/20260527192254_claude_large_codebase_08_component_boundaries_compressed.png?v=20261005)

## 十、大代码库应该按什么顺序配置 Claude Code？

这里我给一个落地顺序。

不是说所有团队都必须一模一样。

但大部分团队可以按这个顺序走。

**第一步，先写根 CLAUDE.md。**

不要追求完美。

先写：

- 仓库结构
- 常用命令
- 全局禁区
- 关键文档入口
- 最容易踩的坑

**第二步，补子目录 CLAUDE.md。**

哪个目录最常被 Claude Code 修改，就先给哪个目录补。

比如 `apps/web/CLAUDE.md`、`services/payment/CLAUDE.md`。

重点写本目录命令和规则。

**第三步，配置忽略和权限。**

生成代码、构建产物、第三方库、巨型快照文件，该排除就排除。

不要让 Claude Code 把时间浪费在垃圾上下文上。

**第四步，接 hooks。**

先接最确定的：

- format
- lint
- typecheck
- 单测
- 会话结束总结

**第五步，把重复专家流程做成 Skills。**

安全审查、文档更新、发布检查、事故复盘，这些都适合抽成 skill。

**第六步，再考虑 Plugins、MCP、LSP、Subagents。**

前面的基础没做好，直接上 MCP 和一堆复杂插件，容易越配越乱。

先让 Claude Code 在本地仓库里找得准、改得稳。

再接更多外部能力。

## 十一、团队落地：为什么需要一个 Agent 管理人？

技术配置只是第一步。

大团队里，Claude Code 真正难的是组织落地。

如果每个工程师都自己配一套：

- 自己写 CLAUDE.md
- 自己写 hooks
- 自己配 MCP
- 自己总结使用技巧

那很快就会变成各玩各的。

好经验留在个人电脑里。

踩过的坑没人沉淀。

新人进来还要重新摸索。

Anthropic 文章里提到一个角色：agent manager。

你可以理解成：

**负责维护 AI 编程工程体系的人。**

这个人不一定是专职。

小团队里，一个 DRI 就够。

但他要负责：

- 统一 CLAUDE.md 层级规范
- 管理 hooks 和权限策略
- 维护团队可用 Skills
- 选择和分发 Plugins
- 管理 MCP 接入边界
- 推动代码审查和安全流程
- 定期清理过期规则

大公司里，这个角色可能在 DevEx、开发效率、基础架构团队下面。

因为它本质上是开发者工具的一部分。

不是某个开发者的个人爱好。

![图9：团队落地 Claude Code 的关键是把个人经验沉淀成可复用的工程资产](https://file1.kamacoder.com/i/web/20260527192255_claude_large_codebase_09_team_asset_loop_compressed.png?v=20261005)

这里我觉得 Anthropic 说得很现实：

**自下而上的使用会带来热情，但没有集中维护，知识会停留在小圈子里。**

这句话非常适合国内团队。

很多公司现在都是几个工程师先偷偷用。

用得好的那个人效率很高。

但团队并没有真正吃到红利。

因为方法没有标准化，也没有沉淀成项目资产。

## 十二、普通开发者怎么开始？

如果你不是团队负责人，只是一个普通开发者，也能做很多事。

不要一上来就想着搭 MCP、写 plugin、搞一套平台。

先从最小动作开始。

**第一，给当前项目写一个根 CLAUDE.md。**

哪怕只有 20 行也行。

写清楚项目结构、常用命令、禁止事项。

**第二，把你反复提醒 Claude Code 的话沉淀进去。**

凡是你连续说过三次的规则，都应该考虑写进文件。

**第三，给常改目录补子目录 CLAUDE.md。**

比如你经常改 `docs/`，那就在 `docs/` 下面写文档规则。

比如你经常改 `services/payment/`，那就写支付模块规则。

**第四，排除明显无关文件。**

生成代码、构建产物、三方依赖，别让 Agent 老去读。

**第五，把重复流程整理成 skill 或脚本。**

比如每次发版前都要检查 5 件事。

不要每次口头提醒。

把它沉淀成可复用流程。

大代码库里，AI 编程不是"换个更强模型"就完了。

真正能拉开差距的是：

**你能不能把项目知识、工具链和团队规范，变成 Claude Code 可以稳定使用的工程支架。**

## 结尾

Claude Code 在大代码库里，不是靠把整个仓库背下来。

它更像一个会使用工具的新同事。

你给它清晰目录、局部规则、测试命令、符号导航、自动检查，它就能越做越稳。

你不给它这些，它就只能盲搜、乱猜、浪费上下文。

所以这篇文章最重要的一句话是：

**大代码库里的 Claude Code，不是模型单打独斗，而是模型 + 工程支架一起工作。**

会用 Claude Code，只是开始。

能把 Claude Code 接进团队工程体系，才是真正的 AI 编程落地。

加油。

## 参考资料

- Anthropic Blog：How Claude Code works in large codebases: Best practices and where to start：https://claude.com/blog/how-claude-code-works-in-large-codebases-best-practices-and-where-to-start
- Claude Code Docs：Manage Claude's memory：https://docs.claude.com/en/docs/claude-code/memory
- Claude Code Docs：Hooks：https://docs.claude.com/en/docs/claude-code/hooks
- Claude Code Docs：Skills：https://docs.claude.com/en/docs/claude-code/skills
