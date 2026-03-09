---
title: OpenClaw连接Discord中遇到的问题
published: 2026-03-09
description: OpenClaw连接Discord过程中遇到的问题与解决经验
category: Learning
tags:
  - Experience
---
> AI正在重塑开发者的工作流：一次精准的Prompt，往往抵得上过去数小时的信息检索与反复试错。然而，面对OpenClaw这类新兴项目时，AI的短板也显露无疑：一方面，AI生成的内容同质化严重，面对冷门Bug缺少解决方案；另一方面，相比PyTorch、OpenCV等成熟生态，社区尚未沉淀出体系化的教程，而官方教程对于新手来说相对困难。这篇文章记录了我配置OpenClaw channels的经验，希望能帮后来者少踩几个我踩过的坑。
# 为什么选择Discord
![alt text](<../../assets/images/Pasted image 20260309092840.png>)
![alt text](<../../assets/images/Pasted image 20260309092910.png>)（上图出处：https://open-claw.me/zh/channels）
- 在目前Openclaw支持的Channel中，可以看出，Discord的配置难度相对较低，功能丰富，支持文本、富文本等多种格式。
- Telegram当然也是一个很好的选择，但由于现实的原因，其配置相对困难。

> [!note]
>  [参考教程：How to connect OpenClaw to Discord: Integration guide](https://lumadock.com/tutorials/connect-openclaw-to-discord?language=romanian)
 大部分设置可以根据**参考教程**一步步来，下文是我在配置过程中遇到的问题与经验
# 如何查看Channel连接情况
- 使用`openclaw channels status`命令
	![[Pasted image 20260309125039.png]]
	- reachable：OpenClaw 网关服务本身在线，本地配置和基础网络没问题。
	- enabled：允许 OpenClaw 启动 Discord 集成
	- configured：找到了配置文件（token 等参数已填写）
	- running：Discord 适配器进程已启动
	- disconnected：实际未连接到 Discord 服务器
	- token:config：Token 来源是配置文件（非环境变量）
# 是否设置allowlists
![alt text](<../../assets/images/Pasted image 20260309110700.png>)
- 教程中有一步需要获取Sever ID（Guide ID）和Channel ID，目的是设置Bot可访问哪些Channel。如果是个人使用，则没必要设置allowllists，直接设置为open即可（如下图）。
- 当然，设置一下肯定也没有坏处。
	 - ![!\[\[Pasted image 20260309110913.png\]\]](<../../assets/images/Pasted image 20260309110913.png>)
# 设置DISCORD_BOT_TOKEN
![!\[\[Pasted image 20260309111010.png\]\]](<../../assets/images/Pasted image 20260309111010.png>)
- 这里本人使用环境变量会导致Discord not configured，即OpenClaw找不到环境变量里的Token，目前还没找到解决方案。最后是通过将DISCORD_BOT_TOKEN写入.openclaw/openclaw.json解决（如下图）。
	![!\[\[Pasted image 20260309112705.png\]\]](<../../assets/images/Pasted image 20260309112705.png>)
# Discord中BOT离线的解决方法
**问题**：完成 OpenClaw 配置后，Discord Bot 始终离线，无法响应指令。
**根因**：Discord 网关依赖 WebSocket 长连接，对网络环境有特定要求。常规代理模式（HTTP/HTTPS 代理）仅覆盖浏览器或显式配置的应用层流量，而 OpenClaw 的底层连接可能未落入该范围。
**解决方案**：启用 TUN 模式（虚拟网卡层代理）。该模式在操作系统网络栈层面拦截流量，确保包括 WebSocket 在内的所有协议均被正确转发。
**验证**：开启 TUN 后执行 `openclaw channels status`，`disconnected` 应变为 `connected`，Bot 状态同步更新为在线。

