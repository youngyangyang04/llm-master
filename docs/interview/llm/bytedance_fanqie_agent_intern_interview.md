---
title: 字节Agent应用开发实习一面面经：ES混合检索、多Agent编排、三层记忆、QKV与算法题怎么答
description: 字节跳动番茄小说Agent应用开发日常实习一面面经，按真实面试对话给出ES向量检索与BM25混合检索、长文档和代码召回、三层记忆、长期记忆校验、Plan-and-Execute DAG、Planner Worker Reviewer边界、子Agent Token控制、消息编排、MCP工具、DOM压缩、Transformer QKV多头注意力和最大无重复子串的现场回答。
keywords: [字节跳动面经, 番茄小说面经, Agent应用开发实习, Agent开发面试, 日常实习面经, ES混合检索, Elasticsearch BM25, RAG检索评测, 多Agent编排, 三层记忆, Plan-and-Execute, DAG拓扑排序, Planner Worker Reviewer, 子Agent Token控制, MCP工具, DOM上下文压缩, Transformer QKV, 多头注意力, 最大无重复子串, 滑动窗口]
tags: [字节跳动面经, Agent开发, 大模型面试, RAG面试, 实习面经]
faq:
  - q: "ES和BM25混合检索到底是什么意思？"
    a: "更准确的说法是用Elasticsearch同时做BM25关键词召回和dense_vector语义召回，再通过RRF或加权打分融合结果。BM25擅长函数名、错误码等精确词，向量检索擅长需求描述和代码实现之间的语义匹配。"
  - q: "多Agent如何避免子Agent耗尽上下文？"
    a: "禁止无限递归，为整次任务设置全局Token预算和最大深度，父Agent只下发最小任务包，子Agent只返回结构化摘要与产物引用，工具输出分页截断，达到预算阈值后降级或停止派生。"
  - q: "Planner、Worker、Reviewer应该使用相同工具吗？"
    a: "不应该。Planner只负责拆任务和依赖关系，Worker按职责获得最小工具权限，Reviewer原则上只读并拥有测试与检查工具，不直接修改代码，避免自己生成又自己验收。"
---

# 字节Agent应用开发实习一面：项目被连续深挖，应该怎么答？

前面整理过一篇[字节 Agent 开发四面面经](./20260506bytedance.md)，覆盖 Prompt、RAG、MCP、微调和 Agent 架构，题目很广。

这次[知识星球](https://programmercarl.com/other/kstar.html)里录友分享了是番茄小说 Agent 应用开发日常实习的面试题。

![](https://file1.kamacoder.com/i/web/20260825111336-b561eb.png?v=20261005)

题目有十几道，难度不低。

面试官没有让他背概念，而是一直追着项目问：**为什么这样设计？效果提升了多少？数据怎么测的？到底有多少人用？出错了怎么办？**

只答“混合检索效果更好”“多 Agent 可以分工”，基本撑不过两轮追问。

本次面经问题：

-  一、为什么用 ES + BM25 混合检索？优化了多少，怎么测的，有多少人用？
-  二、需求文档和代码文件通常很大，怎么保证检索正确？怎么判断对错？
-  三、为什么要分成三层记忆？
-  四、说说记忆设计，怎么保证长期记忆和外部记忆是正确的？
-  五、Plan-and-Execute 基于 DAG 拓扑排序，说说你的设计
-  六、Planner、Worker、Reviewer 的边界怎么设计？挂载的工具一样吗？
-  七、如果子 Agent 还能调用子 Agent，怎么保证不会挤爆 Token？
-  八、基模是什么？多个 Agent 的 system、user、assistant、tool 消息怎么编排，怎么保证不乱序？
-  九、接入了几个工具？都是本地的，还是 MCP？
-  十、DOM 网页内容很大，怎么保证不撑爆上下文？
-  十一、Transformer 的 QKV 和多头注意力机制
-  十二、算法题：最大无重复子串


这位录友在分享面经的时候，有的问题没有给出对应问题的回答，我又做了适当补充。

### 一、为什么用 ES + BM25 混合检索？优化了多少，怎么测的，有多少人用？

我先纠正一下，简历里写“ES + BM25”不太严谨。我们实际是用 ES 同时做 **BM25 关键词召回和向量召回**，再通过 RRF 融合结果。

因为我们搜的是需求文档和代码。函数名、错误码这类精确词，BM25 更准；用户的自然语言和代码实现经常不是同一种表达，比如用户说“清理失效书籍”，代码里可能叫 `removeInvalidItems`，这种向量检索更容易召回。两路刚好互补。

我们用 620 条真实 Query 做了标注，420 条调参数，200 条作为独立测试集。测试集上，BM25 的 Recall@10 是 73.5%，向量检索是 78.0%，混合检索提升到 86.5%。P95 延迟从 118 毫秒增加到 189 毫秒，业务上可以接受。

当时还只是团队内测，激活用户 74 人，稳定周活约 41 人。检索结果的直接采用率从 52.8% 提升到 64.6%。样本量不算大，所以我当时只把它定义为内测阶段有效，没有包装成大规模线上结论。

![混合检索与RRF融合](https://file1.kamacoder.com/i/web/20260420205132.png?v=20261005)

### 二、需求文档和代码文件通常很大，怎么保证检索正确？怎么判断对错？

最早我们按固定 Token 切分，确实会把函数签名和函数体切开。后来需求文档按标题和段落切，代码按 AST 切到类和方法，每个 Chunk 都保留文件路径、符号、分支和 commit 等元数据。

检索采用“小块命中，大块返回”。先召回小 Chunk，RRF 合并后取 Top 30，再 Rerank 到 Top 6；放进上下文时，通过 `parent_id` 补回完整函数或需求章节，避免只拿到几行残缺代码。

正确性分两层判断。检索层看标准文件或代码片段是否出现在 Top-K，用 Recall@K、MRR 评估；任务层看 Agent 最后改的文件是否正确、测试是否通过，再加人工验收。

另外会按 `repo、branch、commit_id、权限` 做硬过滤，每段证据都带路径和行号。这样 Reviewer 能反查来源，不会只根据一段没有出处的摘要改代码。

### 三、为什么要分成三层记忆？


我们分三层不是为了把架构画得复杂，而是三类信息的生命周期、可信度和读取方式不一样。

第一层是工作记忆，放当前目标、DAG 状态和最近的工具结果，任务结束后就压缩或清理。第二层是长期记忆，放用户偏好、仓库约定和历史经验，按需检索。第三层是外部知识，也就是需求平台、代码仓库和接口文档里的实时事实。

如果都混在一起，一方面会浪费 Token，另一方面模型容易把历史推断当成当前事实。三层冲突时，以当前外部事实为准。

比如长期记忆里是 JDK 17，但当前分支的 `pom.xml` 已经升级到 JDK 21，那最终就以当前代码为准，同时把旧记忆标成过期。

### 四、说说记忆设计，怎么保证长期记忆和外部记忆是正确的？

我不敢说记忆能百分之百正确，主要从写入、读取和追溯三步控制风险。

写入时不保存全部聊天，而是抽取结构化 Memory Item，带来源、时间、作用域、置信度和证据。只有用户确认过的事实或工具验证过的结论才能进入长期记忆，模型自己的一次推断不会直接写入。

读取时先按用户、仓库、分支做硬过滤，再进行语义召回和 Rerank。带时效性的内容在使用前还要重新查外部系统；新旧记忆冲突时保留版本，把旧内容标记为过期。

每次决策都会记录使用了哪条记忆和哪个证据。出了 bad case，可以判断是记忆过期、召回错误，还是模型判断错误。长期记忆只能作为线索，关键事实仍以当前代码和需求版本为准。

### 五、Plan-and-Execute 基于 DAG 拓扑排序，说说你的设计


Planner 输出的不是自然语言步骤，而是结构化 DAG。每个节点包含任务目标、依赖、输入输出、可用工具、Token 预算和超时时间。

比如一次代码修改，会拆成读取需求、检索代码、分析影响、生成 Patch、测试和 Review。没有依赖的检索任务可以并行，生成 Patch 必须等需求和代码分析都完成。

执行前会检查节点是否重复、依赖是否存在、图里有没有环。调度器用 Kahn 拓扑排序，把入度为 0 的节点放进 ready queue；节点状态和产物都会做 checkpoint，失败后不用全部重跑。

如果执行中发现计划有问题，就把异常返回 Planner，只重规划受影响的子图。Plan-and-Execute 不等于 DAG，DAG 只是我们把计划工程化、可并行和可恢复的一种方式。

![Plan-and-Execute执行流程](https://file1.kamacoder.com/i/web/05_plan_execute.png?v=20261005)

### 六、Planner、Worker、Reviewer 的边界怎么设计？挂载的工具一样吗？


三者挂载的工具不一样，我是按职责做最小权限。简单说就是：**Planner 对计划负责，Worker 对产物负责，Reviewer 对验收负责。**

Planner 只看用户目标、仓库概要和工具说明，负责生成 DAG，不能直接改文件。Worker 只完成单个节点，检索 Worker 只有只读工具，代码 Worker 才有工作区写入权限。

Reviewer 可以读需求、看 Diff、跑测试和静态检查，但原则上不能直接修改代码。发现问题后，它把结构化 Review 结果退回 Worker，避免自己修改再自己判通过。

这些权限不是只写在 Prompt 里，而是在 Tool Gateway 按角色和任务范围鉴权。即使 Reviewer 生成了写文件调用，执行层也会拒绝。

### 七、如果子 Agent 还能调用子 Agent，怎么保证不会挤爆 Token？


这个不能只靠 Prompt 提醒。我们默认禁止任意递归，只有指定节点能创建子 Agent，而且最大深度是 2。整个任务有全局 Token 预算，父 Agent 派生子任务时必须从自己的剩余额度里分配。

预算会预留主流程、Worker 和 Reviewer 的份额，避免 Worker 把 Token 全部用完，最后没有资源验收。除了输入输出 Token，还会限制工具返回大小、调用次数和执行时间。

父 Agent 只给子 Agent 最小任务包，包括目标、必要证据、工具和输出 Schema，不复制完整聊天记录。子 Agent 也只返回摘要、关键证据和 Artifact ID，不回传全部执行轨迹。

工具结果默认分页截断，预算到 70% 后禁止继续派生，接近上限时强制收束。另外任务会记录祖先链，避免 A 调 B、B 又调 A 的递归环。

![Token预算控制状态机](https://file1.kamacoder.com/i/web/20260519175451_harness_stable_07_token_budget.png?v=20261005)

### 八、基模是什么？多个 Agent 的 system、user、assistant、tool 消息怎么编排，怎么保证不乱序？


主模型是私有部署的 Qwen2.5-72B-Instruct，通过 vLLM 提供接口。Planner 和 Reviewer 用 72B，检索改写、摘要这类简单 Worker 会路由到 14B，降低成本和延迟。

每个 Agent 都有独立 thread，不会把所有消息混进一个数组。System Prompt 包含全局安全规则和角色边界，具体任务作为 user message；assistant 发起带 `tool_call_id` 的调用，执行器再返回对应的 tool message。

并行 Agent 的事件都会带 `run_id、task_id、agent_id、seq_no、parent_event_id`。单个 Agent 内按 seq_no 保序，跨 Agent 按 DAG 依赖和 parent_event_id 判断因果关系，不按谁先返回就直接拼接。

子 Agent 完成后只返回结构化结果、状态和证据引用，不把完整的 system、assistant、tool 历史塞回主 Agent。这样既避免消息乱序，也防止不同角色的上下文互相污染。

### 九、接入了几个工具？都是本地的，还是 MCP？


当时一共给模型暴露了 11 个工具。7 个是本地工具，包括文件读取、关键词搜索、符号查询、Git Diff、Patch 和沙箱测试；另外 4 个通过 MCP 接入需求文档、代码评审、Issue 和内部知识库。

MCP 只是接入协议，不代表工具一定在远端。我们的 MCP 里有 1 个本地 stdio Server，另外 3 个走内网 Streamable HTTP。

这些工具不会全部挂给每个 Agent。Planner 只看工具摘要，Worker 根据任务拿 3 到 5 个工具，Reviewer 只有只读和检查工具。这样既节省工具 Schema 的 Token，也减少选错工具和越权调用。

### 十、DOM 网页内容很大，怎么保证不撑爆上下文？


我们不会把原始 HTML 直接放进上下文。页面抓取后先删除 script、style、广告、隐藏节点和重复导航，正文页面用 Readability 提取主体，操作页面只保留可交互元素。

保留下来的 DOM 会简化成 `node_id、role、name、state` 这样的结构，再按 section、table、form 分块。Agent 先看到页面概要，需要哪一块再局部读取。

工具单次最多返回 4K Token，单页累计最多 12K，超过就分页。连续操作时只传当前 viewport 和发生变化的节点，不重复发送整页。

每次注入前还会估算 Token，超出预算就先检索或摘要。动态页面优先读取可访问性树或合规的结构化接口，不从几万行 DOM 里直接猜状态。

### 十一、Transformer 的 QKV 和多头注意力机制


输入 Token 的表示是 `X`，分别乘三个可学习矩阵得到 Q、K、V。Q 表示当前 Token 想找什么，K 用来和 Q 计算匹配度，V 是匹配后真正被聚合的内容。

计算过程是 `softmax(QK^T / sqrt(dk))V`。除以 `sqrt(dk)` 是为了避免维度增大后点积过大，导致 Softmax 过于尖锐。

多头注意力会把隐藏维度投影到多个子空间，每个头独立计算注意力，可以学习不同类型的关系。最后把各头结果拼接，再通过 `Wo` 投影回模型维度。它在序列长度上的主要瓶颈仍然是 O(n²)。

![Self-Attention QKV流程](https://file1.kamacoder.com/i/web/1778232933.png?v=20261005)

### 十二、算法题：最大无重复子串


我的思路是滑动窗口：右指针遍历字符串，哈希表记录每个字符上次出现的位置。如果字符在当前窗口内重复，就把左边界移动到上次位置的下一位。

每个字符最多被右指针访问一次，左边界只向右，所以时间复杂度 O(n)，空间复杂度 O(字符集大小)。

```python
def length_of_longest_substring(s: str) -> int:
    last = {}
    left = 0
    ans = 0

    for right, ch in enumerate(s):
        if ch in last:
            left = max(left, last[ch] + 1)

        last[ch] = right
        ans = max(ans, right - left + 1)

    return ans
```

这里 `left` 一定要取 max。比如 `abba` 最后遍历到 `a` 时，它上次出现的位置已经在窗口外，左边界不能往回走。空字符串返回 0，全部相同字符返回 1。

### 这场一面真正卡人的地方

这场面试的题目并不偏。

RAG、多 Agent、记忆、MCP、Transformer、滑动窗口，都是常见内容。难的是面试官把每个项目名词都往下追了三层：

**为什么做 → 怎么实现 → 怎么证明有效。**

回答时有三个动作很加分：

第一，发现术语不严谨就主动纠正。比如“ES + BM25”应该说成“ES 中的向量召回 + BM25 关键词召回”。

第二，别只报最好看的数字。把测试集怎么来、指标怎么定义、延迟代价和用户规模一起说出来。

第三，少说“保证正确”“绝对不会爆”。真实系统很少有这种保证。更像工程师的说法是：**我在哪些环节降低风险，触发什么阈值以后如何降级，出了错能不能追溯。**

如果这篇里的 Multi-Agent 部分还不熟，可以继续看[多 Agent 通信与编排面试详解](./multi_agent_communication_interview.md)和[Multi-Agent Harness 面试详解](./multi_agent_harness_interview.md)；检索评测可以对照[RAG 落地最难的地方在哪](./rag_hardest_parts_interview.md)；QKV 和多头注意力再回到[Transformer 大厂面试题汇总](./transformer_interview.md)补基础。

面试不是念答案。

**把自己的项目说真、说细、说闭环，才扛得住追问。**
