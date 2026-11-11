---
title: Claude风控收紧：不只Max，Pro账号也会触发KYC刷脸认证，无法使用Claude Code怎么办？
description: 近期Claude Pro用户也反馈遭遇Persona身份验证，需要提交证件并刷脸。本文区分Anthropic官方认证规则与社区反馈，说明Pro、Max付费账号为什么也可能被验证拦住，并提供国内使用Claude Code的安装与接入指南。
keywords: [Claude风控, Claude KYC, Claude身份认证, Claude刷脸认证, Claude实名认证, Claude Pro认证, Claude Max认证, Persona身份验证, Anthropic, Claude账号无法使用, Claude Code国内使用, Claude Code安装教程, Claude API接入, AI编程]
tags: [Claude, Claude Pro, Claude Max, Claude Code, 身份认证, 大模型动态]
---

# Claude风控又收紧了？不只Max，Pro也要刷脸了

以前担心 Claude 账号被封，现在还得担心：**钱已经付了，账号却被身份认证拦住。**

最近，社区又出现了 Claude Pro 用户被要求进行 KYC 身份认证的反馈。提交证件、配合摄像头刷脸，这些要求已经不只是 Max 用户会遇到的问题。

**如果你以为买 Pro 就能避开身份验证，这个预期需要改了。**

不过，先把范围说清楚：目前能确认的是，**部分 Pro 账号也会触发认证，并非所有 Pro、Max 账号都已统一强制刷脸。** 官方也没有公开近期触发比例，不能把这些反馈直接当成全面升级风控的公告。

## 所谓KYC，不只是收个短信验证码

大家说的 KYC，在这里指的是身份核验。

![](https://file1.kamacoder.com/i/web/20261106154324.jpg?v=20261005)

个人认证需要准备有效的政府签发带照片实体证件，也可能需要用手机进行实时自拍，也就是大家常说的“刷脸”。认证服务由 Persona 提供。

官方给出的认证场景包括特定功能访问、平台完整性检查，以及安全与合规措施，并没有把范围限定在 Max 套餐。

**订阅资格和身份核验，是两件事。付费不等于免认证。**

至于什么操作一定会触发、哪种账号肯定不会触发，官方没有公开完整规则。不能因为有人没遇到，就断言 Pro 不需要认证；也不能因为有人遇到了，就说所有用户都必须刷脸。

## 无法使用Claude，可以看这里

之前我们写过 [Claude Code 账号风控问题](./claude_code_china_ip_ban.md)。这次 Pro 用户的认证反馈，又给账号可用性增加了一层不确定性。

如果目前你也无法使用 Claude，可以看这里：**[国内如何使用 Claude Code？安装与接入教程](../../ai/claude/code-china-guide.md)**。

![Claude Code国内接入教程](https://file1.kamacoder.com/i/web/20261027170310.jpg?v=20261005)

这篇指南按 Windows、macOS、Linux 写了安装、创建 API Key、配置和启动步骤，走的是中转 API 接入 Claude Code 的路线，不以登录 Claude 官网账号为前提。

它适合想继续用 Claude Code 写代码的读者；Claude 网页会员权益、历史对话和官方账号认证问题，需要另行处理。

**先把开发工具用起来，别让项目一直卡在账号上。**

## 资料来源

* Claude官方身份验证说明：https://support.claude.com/en/articles/14328960-identity-verification-on-claude
* Reddit用户反馈，Pro订阅后触发Persona认证：https://www.reddit.com/r/Anthropic/comments/1xgx9xz/serious_warning_before_subscribing_to_claude_pro/
* Reddit用户反馈，Pro使用期间被要求证件与面部验证：https://www.reddit.com/r/claude/comments/1vuaaji/claude_persona_identity_verification_account/
