---
title: "设置用户无需密码自动登录到 Windows 系统"
publishDate: 2020-03-03 08:58:42 +0800
date: 2020-03-23 11:36:48 +0800
tags: windows
position: knowledge
coverImage: /static/posts/2020-03-03-08-58-32.png
permalink: /post/automatically-sign-in-windows.html
---

你一定要为你的 Windows 用户账户设置密码，一来会安全一些，而来可以远程登录使用；但有时出于一些特殊的目的，不希望在每次开机后都必须输入密码才能进入系统。于是你可以使用本文提供的方法在每次开机的时候免密码登录到 Windows 操作系统。

---

<div id="toc"></div>

## 步骤

1. 在 Windows 搜索框中输入 `netplwiz`，然后回车打开命令；
2. 去掉“要使用本计算机，用户必须输入用户名和密码”的勾勾；
3. 点击“确定”或“应用”后，输入自动登录账号的用户名和密码。

注意，输入用户名和密码的时候，如果你使用了微软账号登录，那么需要输入你的微软账号，比如这样“walterlv@outlook.com”；而密码是你微软账号的密码，而不是 PIN 码。

## Windows 10 截图

![netplwiz](/static/posts/2020-03-03-08-58-32.png)

![高级用户账户控制面板](/static/posts/2020-03-03-08-57-14.png)

## Windows 7 截图

![netplwiz](/static/posts/2020-03-03-08-54-26.png)

![高级用户账户控制面板](/static/posts/2020-03-03-08-55-58.png)


