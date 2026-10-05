---
title: Claude大规模封号：Claude账号被封、Claude Code不能用，国内开发者怎么在Codex和Opus 4.8之间切换
description: Claude在2026年6月28日前后出现大规模封号，国内开发者即使用美国IP、TUN模式和谨慎登录策略，也可能遇到account_banned。本文记录Claude账号被封、重新注册再次被封、被迫切换Codex和Claude Opus 4.8中转站的真实经历，给AI编程和Claude Code用户做风险提醒。
keywords: [Claude封号, Claude账号被封, Claude大规模封号, Claude account_banned, Claude Code封号, Claude Code国内不能用, Claude国内使用, Claude中转站, Claude Opus 4.8, Codex, Codex 5.5, AI编程工具, AI难民, 大模型账号风控, Anthropic账号封禁]
tags: [Claude, Claude Code, AI编程, 大模型动态, 账号风控]
---

# Claude大规模封号：Claude账号被封后，我又被迫切回中转站

<a href="https://notes.kamacoder.com/qita/0021.gpt-5-6-sol-terra-luna-domestic-guide.html">
  <img src="https://file1.kamacoder.com/i/web/2026-07-17_15-06-45.jpg?v=20261005" style="width:100%;cursor:pointer;" alt="KamaClaude">
</a>


6月28日开始，Claude开始大规模封号，之前我的两个Claude账号被封了。

后面又搞了一个，开始注意，控制网络，🪜开启 TUN模式。

我都已经养成肌肉记忆了，因为我平时有一些工作，必须要切换 非美国IP，我就会特别把 Claude CLI 和 web全部退掉，等要切回美国IP，再重新登录。

稳定的用了两个月，我以为稳了。

结果今天早上起来，果然有惊喜：

![Claude账号登录后提示账号被封禁](https://file1.kamacoder.com/i/web/2026-06-28_18-32-60.jpg?v=20261005)

上午又去注册邮箱，又去注册一个Claude，心想着，养一养吧，先别上来就冲pro。

结果过了一个小时。喜提 account_banned

![新注册Claude账号一小时后再次出现account_banned封禁提示](https://file1.kamacoder.com/i/web/2026-06-28_18-32-61.jpg?v=20261005)

绝了，妥妥的AI难民

现在又被迫用上了 中转站（我用的是[apidock.ai](https://apidock.ai/)）。。

我之前Claude被封，就切成了Codex。

有一次Codex 额度用没了，我也不能干等了。

就又切回Claude，当时感觉确实 [Opus 4.8](./claude-opus-4-8.md) 做的更好，比Codex顺手。

我现在改成 Codex 5.5 + Opus 4.8（[中转站](https://apidock.ai/)）混着用。

一些设计和技术图，还是 Opus 4.8 做的比较好。


