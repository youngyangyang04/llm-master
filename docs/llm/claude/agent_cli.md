---
title: 为什么Agent时代大家都在做CLI？Claude Code、Codex与命令行的前世今生
description: "Claude Code 为什么一开始就选择终端，Codex CLI 又为什么紧跟这条路？本文从批处理、Unix Shell、GUI 与云原生讲到 AI Agent，拆解 CLI 如何把文件、Git、测试、日志、权限和自动化接进 Agent Loop，也讨论命令行的安全短板、Agent 友好型 CLI 应该怎么设计，以及 CLI、IDE、MCP、Computer Use 谁才是最终入口。"
keywords: [Agent CLI, Claude Code CLI, Codex CLI, AI Agent, 命令行, CLI前世今生, Unix Shell, Agent工具调用, Agent Loop, AI编程, AI Coding, Terminal Agent, MCP, Computer Use, IDE, Agent安全, 沙箱, stdout, stderr, JSON输出, CI自动化, Claude Code, OpenAI Codex]
tags: [Claude Code, Codex CLI, AI编程, Agent, CLI, 大模型入门]
---

# 为什么Agent时代大家都在做CLI？Claude Code、Codex与命令行的前世今生

录友们好，今天聊一个看起来很“复古”的现象：**AI 都发展到 Agent 了，为什么最火的一批产品，反而钻回了黑乎乎的命令行？**

严谨一点说，不是 Claude 这个产品从 CLI 起家，而是 **Claude Code 这个 Coding Agent，一开始就选择了 CLI**。2025 年 2 月，Anthropic 把它作为终端里的研究预览发布；同年 4 月，OpenAI 上线开源的 Codex CLI，5 月才推出 Codex Web。

这不是谁抄谁。两家公司同时撞上了同一个工程事实：**聊天框适合模型“说”，CLI 才方便 Agent 真正“做”。**

前面我们在 [Claude Code 作者说“不写 Prompt，写 Loop”](./claude_code_loop.md) 里讲过 Agent 闭环，又在 [动态工作流](./dynamic_workflows.md) 和 [Loop Engineering 实战](./loop_engineering_guide.md) 里继续拆了怎么编排、怎么验证。这一篇再往下挖：为什么承载这些 Loop 的第一块好地基，偏偏是几十年前的命令行。

## CLI 的前世今生：它从来不只是“黑框里敲字”

先把三个词掰清楚：Terminal 是承载输入输出的终端，Shell 是解释命令的程序，CLI 是用文本命令和软件交互的方式。Claude Code、Codex CLI 今天已经是带面板、状态和快捷键的 TUI，但它们真正依赖的底座，仍然是 Shell、文件系统和一整套命令行工具。

最早的计算机甚至谈不上命令行。人把程序打在卡片上，交给机器批处理，结果晚点再拿回来。**人和机器之间没有实时对话。**

后来分时系统和终端出现，人终于可以输入一行命令，马上看到一行结果。到了 Unix，Shell 又做了一次关键升级：它不只是启动程序的入口，还是一门可以组合程序的控制语言。

`stdin`、`stdout`、`stderr`、退出码、重定向、管道，这几个设计看起来朴素，却把一堆小工具接成了工作流。1978 年 Bourne 对 Unix Shell 的定义里，就已经同时强调了控制流、环境、重定向和进程管道。

GUI 兴起后，命令行没有消失。因为 GUI 解决的是**人的发现成本**：按钮摆在那里，不会写命令也能点。CLI 解决的是**系统的组合成本**：一个命令能进脚本、进 CI、跑远程机器，还能把输出交给下一个程序。

到了云原生时代，Git、Docker、Kubernetes、Terraform、各种云平台，几乎都把 CLI 当成一等公民。原因很简单：服务器没有必要配一块屏幕，自动化也不会拿鼠标点按钮。

Agent 时代，命令行又转了一圈回来。但这次敲命令的主角，开始从人变成模型。

![](https://file1.kamacoder.com/i/web/2026-07-16_10-48-27.jpg?v=20261005)

这张图回答的是：CLI 为什么在 GUI 时代没有死，到了 Agent 时代反而又站到前台。变化的不是命令行突然变好用了，而是“把人的意图翻译成精确语法”这份苦活，从人转移给了 Agent；CLI 原本最劝退人的门槛，恰好被大模型吃掉了。

## Agent + CLI，为什么是绝配？

很多文章会说“CLI 快、轻量、极客”。太浅了。**Agent 选择 CLI，不是因为终端酷，而是 CLI 天然提供了一套可执行、可观察、可组合的反馈系统。**

你让 Claude Code 修一个登录 Bug，它不是吐一段代码就完事，而是可以顺着仓库继续干：用搜索命令定位调用链，读取文件，修改代码，跑单测，看报错，再读 diff，直到证据说明任务完成。

这里最值钱的不是某一条命令，而是 CLI 把整个软件工具链压成了统一结构：

- **输入是文本或文件**：需求、路径、参数都能明确传入；
- **动作是命令**：Git、编译器、测试框架、数据库和云平台都能被调用；
- **反馈是证据**：标准输出、错误输出、退出码、测试报告和 diff 都能回到上下文；
- **组合靠管道和脚本**：一次成功的交互，很容易固化成可重复执行的流程；
- **边界由操作系统兜底**：工作目录、文件权限、网络策略、容器和沙箱能限制 Agent 的爆炸半径。

这就形成了一个完整闭环。

![](https://file1.kamacoder.com/i/web/2026-07-16_10-56-58.jpg?v=20261005)

这张图回答的是：Agent 为什么不能只有“大脑”。CLI 把文件系统和现成工具链接成它的“双手”，stdout、stderr、退出码和 diff 又变成“眼睛”；验证失败不是一句模糊的“不对”，而是下一轮推理能直接消费的证据。

更狠的一点是，CLI 还是**可递归的**。Agent 不只会调用 CLI，它自己也能变成 CLI 的一环：

```bash
cat build-error.txt | claude -p "定位根因并给出修复建议"
git diff | codex exec "检查这次修改是否引入并发问题"
```

交互工具因此变成了脚本组件，可以进 CI、定时任务和无人值守流程。网页聊天框很难自然做到这一点。

## GUI 输了吗？没有，只是它更适合人

GUI 最大的优势，是把功能摊开给人看。设计稿、数据大盘、复杂表格、可视化调试，这些场景让人直接看，比读几百行文本舒服得多。

但 Agent 操作 GUI，很多时候是在“猜像素”：按钮换个位置、弹窗多一层、页面加载慢一点，执行路径就变了。它当然可以靠 Computer Use 看屏幕、移动鼠标，但这种通道**延迟高、状态隐蔽、难并行、难精确复现**。

CLI 则把动作命名了。`git status` 不会因为窗口缩放就跑到右上角，退出码 `1` 也比截图里一行红字更容易判断。

所以真正的分工，不是 CLI 干掉 GUI，而是分成两层：**人用 App、IDE、Web 看计划、审 diff、做授权；Agent 在下面用 CLI、API、MCP 执行和取证。**

![](https://file1.kamacoder.com/i/web/2026-07-16_10-58-09.jpg?v=20261005)

这张图回答的是：IDE、App 和 CLI 到底谁替代谁。上层界面负责把复杂状态讲给人听，下层工具负责把确定动作交给 Agent；审批和证据在两层之间往返，而不是逼人永远盯着终端，也不是让 Agent 永远点像素。

## 但今天很多 CLI，并没有为 Agent 准备好

别因为 Agent 能跑命令，就觉得所有 CLI 都天然可靠。大量老工具默认操作者是人，会弹交互确认、输出彩色进度条、把错误混进 stdout，甚至失败了还返回 `0`。人能凭经验看懂，Agent 接进流水线就容易翻车。

真正 Agent 友好的 CLI，至少要做到几件事：

- 支持 `--json` 或稳定 Schema，不逼模型从花哨日志里猜字段；
- stdout 放结果，stderr 放诊断，退出码真实反映成功或失败；
- 提供 `--dry-run`、diff 和幂等操作，让 Agent 能先预演、失败后安全重试；
- 支持非交互模式、超时和取消，不要半夜卡在“是否继续？Y/n”；
- 权限最小化，凭证可按任务、目录、命令和时间收口；
- 输出可审计，明确谁执行了什么、改了哪里、依据是什么。

**未来 CLI 的竞争，不只是给人写得顺不顺手，还要看它能不能被 Agent 稳定调用。** 结构化输出会越来越重要，权限和沙箱也会从附加功能变成默认能力。

这也是为什么 Claude Code 和 Codex 都在权限模型上花重力气。Agent 越能干，误操作和提示注入的爆炸半径越大。没有边界的 CLI，不是自动化，是把生产事故也自动化。

## CLI 是终局吗？

我的判断很明确：**终端不是终局，CLI 也不会成为所有人的最终界面。**

当 Agent 从一次处理一个任务，变成同时管理十几个长期任务，纯终端很快就不够用了。你需要任务队列、并行状态、权限面板、diff 审核、通知和跨设备接力。OpenAI 后来做 Codex App，本身就在说明：终端适合启动和执行，但不一定适合人监督一支 Agent 队伍。

CLI 也不是所有工具的最佳协议。API 和 MCP 有明确 Schema，通常比解析自然语言日志更稳；Computer Use 则负责那些既没有 API、也没有 CLI 的最后一公里。

但 CLI 不会消失。它会更像 Agent 世界里的“窄腰层”：上面可以是 IDE、桌面 App、网页、手机，下面可以接 Git、测试、数据库、容器和云服务，中间都能通过命令和结构化结果完成交接。

![](https://file1.kamacoder.com/i/web/2026-07-16_10-58-57.jpg?v=20261005)

这张图回答的是：CLI 如果不是最终界面，为什么仍然会长期存在。未来不会由一个入口通吃，而是“人类监督层—Agent 编排层—工具执行层”三层分工；CLI 是执行层最成熟的一条通道，但会和 API、MCP、Computer Use 长期共存。

所以，真正的终局不是 CLI。

**终局是所有软件都长出一套 Agent 能调用、能验证、能审计、也能被安全关住的接口。**

CLI 只是最早准备好的那一个。

## 参考链接

- Anthropic 官方发布（Claude 3.7 Sonnet 与 Claude Code 研究预览）：https://www.anthropic.com/news/claude-3-7-sonnet
- Anthropic 官方产品页（Claude Code 跨工具链执行）：https://www.anthropic.com/product/claude-code
- Anthropic 工程博客（Claude Code 沙箱与权限边界）：https://www.anthropic.com/engineering/claude-code-sandboxing
- Claude Code 官方文档（CLI 与非交互模式）：https://code.claude.com/docs/en/cli-reference
- OpenAI 官方发布（Codex CLI 4 月上线、Codex Web 5 月上线）：https://openai.com/index/introducing-upgrades-to-codex/
- OpenAI 官方发布（Codex App 与多 Agent 监督界面）：https://openai.com/index/introducing-the-codex-app/
- OpenAI 官方仓库（Codex CLI）：https://github.com/openai/codex
- Bell Labs 论文（The UNIX Shell）：https://www.nokia.com/bell-labs/publications-and-media/publications/unix-time-sharing-system-the-unix-shell/
