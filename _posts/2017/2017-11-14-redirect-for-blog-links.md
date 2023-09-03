---
title: "为修改了链接地址的博客进行重定向"
publishDate: 2017-11-14 01:05:37 +0800
date: 2018-08-19 19:09:39 +0800
tags: site
coverImage: /static/posts/2017-11-14-00-58-47.png
permalink: /post/redirect-for-blog-links.html
---

不同于笔记，博客除了给自己带来知识的积累之外，还将知识和思想分享给了互联网上的同道中人。

于是，当我不得不修改博客地址的时候，就不得不考虑地址修改的兼容问题。

---

博客自发布的那一刻开始，就随时可能被各种奇怪的机构收录：

- 搜索引擎*（喜欢被收录，这样就有更多的人能够获益）*
- 授权的转载站点*（虽然目前还没有）*
- 各种泛滥的去除了原作者信息的盗版*（比如这里 [在Windows10系统上为WPF窗口添加模糊特效](http://www.qingpingshan.com/bc/aspnet/334582.html)）*
- 被其它内容引用*（喜欢被引用，说明这份知识是有用的）*

分享和开放的互联网行为一般会在引用或收录的时候加上原文链接，于是我的链接一旦发布，便不建议再更改。

可是，链接有问题啊！那就重定向！

我使用 Jekyll 博客，于是，我在根目录建立了一个 `redirect` 文件夹，专门存放链接的重定向。里面的内容只有两个：

- 存放原址
- 重定向到目标地址的脚本

代码如下：

```xml
---
permalink: /post/wpf-add-on-ui.html
---
<script>
  window.location.href="/post/wpf-cross-domain-ui.html";
</script>
```

可以在这个链接中尝试重定向：<https://walterlv.github.io/post/wpf-add-on-ui.html>

## 附那些盗版

![盗版](/static/posts/2017-11-14-00-58-47.png)  
▲ 某掐头去尾的盗版网站

![盗版](/static/posts/2017-11-14-00-54-54.png)  
▲ 盗版

![被翻译了的盗版](/static/posts/2017-11-14-00-54-30.png)  
▲ 被翻译了的盗版

![被翻译了的盗版](/static/posts/2017-11-14-00-51-28.png)  
▲ 被翻译了的盗版


