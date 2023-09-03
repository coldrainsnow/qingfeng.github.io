---
title: "所有 WPF 程序的界面渲染完全糊掉，竟是戴尔电脑惹的锅？"
publishDate: 2021-11-05 20:47:55 +0800
date: 2021-12-21 16:18:51 +0800
tags: wpf
position: problem
version:
  current: 中文
versions:
  - 中文: #
  - English: /post/wpf-renders-wrong-because-of-nahimicosd-en.html
coverImage: /static/posts/2021-11-05-20-07-45.png
permalink: /post/wpf-renders-wrong-because-of-nahimicosd.html
---

近期收到了多起来自用户的反馈，说我们软件界面糊成一团，完全没办法看到按钮在哪里。我一看，这可棘手了，完全不是我们软件能渲染出来的样子啊！

本文将先给出结论帮助大家解决问题，随后再展示我们的调查过程。

---

This post is written in **multiple languages**. Please select yours:

{% include post-version-selector.html %}

<div id="toc"></div>

## 现象

大家看看下面的界面：

* 不同的界面元素大小不一，参差不齐地随处摆放
* 鼠标划过界面各部分时，划过的矩形区域会闪烁，闪成不同的模样
* 虽然渲染杂乱无章，但功能依旧还在，你能在界面本来应该在的部位点击获得本该有的正常的界面功能

![糊掉的界面](/static/posts/2021-11-05-20-07-45.png)

很明显，这是渲染炸掉了。我们软件的各处逻辑功能什么的都非常正常。

另外，评论区有童鞋反馈搜不到这篇答案，所以我多贴一些关键字在这里：

```
WPF花屏
Blend界面花屏
WPF控件错乱
wpf模糊
wpf控件显示异常
wpf重影
wpf按钮异常显示
wpf异常放大
wpf渲染问题
wpf渲染错误
wpf render abnormal
```

## 原因

*正常情况下造成 WPF 渲染炸掉的原因其实有很多，但大多数只破坏一台计算机。而真正让产品在全球大范围炸掉的，只有近期戴尔外星人推送的一枚新的更新。因此，本博客只说戴尔外星人造成的问题。至于其他原因，你可以从本原因中获得灵感自行查找。*

**请检查一下出问题的电脑上是否有 NahimicOSD.dll 这个文件**，如果这是一台戴尔电脑，那么它最有可能出现在下面这个路径下：

* C:\ProgramData\A-Volute\DellInc.AlienwareSoundCenter\Modules\ScheduledModules\NahimicOSD.dll

如果这是其他品牌的电脑，那么他很有可能出现在下面这个路径下（因为插上了外星人耳机，自动安装了驱动）：

* C:\ProgramData\AWHeadset\DellInc.AlienwareSoundCenter\Modules\ScheduledModules\NahimicOSD.dll

当然，也有其他反馈说在别的路径下的：

* C:\ProgramData\A-Volute\A-Volute.28054DF1F58B4\Modules\ScheduledModules\NahimicOSD.dll
* C:\ProgramData\A-Volute\A-Volute.Nahimic\Modules\Scheduled\NahimicOSD.dll
* C:\ProgramData\A-Volute\Modules\ScheduledModules\NahimicOSD.dll

同时也存在非戴尔设备的情况：

* C:\Program Files\Nahimic\Nahimic2\UserInterface\Nahimic2OSD.dll

NahimicOSD 是一个用于在应用程序最终渲染结果上叠加另一个显示层的库，这个库会在应用程序运行时注入进程，并在目标进程调用 DirectX 渲染时将叠加层加入渲染。

## 解决方法

### 用户端解决方法（你自己遇到问题时使用）

由于这个文件已经注入到了很多应用中，所以解决方法是[将 NahimicOSD.dll 文件重命名](http://blog.walterlv.com/post/rename-executable-self-when-running.html)（因为你直接删删不掉）。等重启电脑后，这个文件也就不会注入到任何程序里去了，这时你想怎么处置都可以。

另外，这个文件可能有 x86 和 x64 两个版本；可能都需要处理，适用于不同架构的进程。已知有问题的应用：

1. dnSpy (x64, .NET)
1. [WPF Performance Suite](http://blog.walterlv.com/post/wpf-rendering-dirty-region.html) (x64/x86, .NET Framework)

我们已经尝试了各种方式与外星人技术人员取得联系，但是都失败了。现在相关的问题戴尔技术论坛上也有人在讨论，不过官方技术人员目前还没有任何处理。现在有两个帖子在讨论，我在后面有回复：

* [Solved: Area-51m R2, blurred text and images in some apps - Dell Community](https://www.dell.com/community/Alienware/Area-51m-R2-blurred-text-and-images-in-some-apps/m-p/8073108#M47622)
* [m17 R2, WPF apps distorted, disappearing window contents - Dell Community](https://www.dell.com/community/Alienware/m17-R2-WPF-apps-distorted-disappearing-window-contents/m-p/8069137#M47499)

也写了一篇无奈的吐槽：

* [The WPF UI is blurring and broken while the Alienware NahimicOSD.dll injects · Issue #5708 · dotnet/wpf](https://github.com/dotnet/wpf/issues/5708)

### 开发端解决方法（修改你的程序以应对这种情况）

#### 方法一：软渲染（无奈之举）

如果你的程序足够简单，那么采用软渲染来规避这个问题也未尝不可。方法是在你第一个窗口显示出来之前，调用一下下面这句话：

```csharp
System.Windows.Media.RenderOptions.ProcessRenderMode = System.Windows.Interop.RenderMode.SoftwareOnly;
```

如果想避免影响范围太大，还可以判断一下是否存在这个文件：

```csharp
bool hasInjected = Process.GetCurrentProcess().Modules.OfType<ProcessModule>().Any(x =>
{
    return x.FileName.Equals(@"C:\ProgramData\A-Volute\DellInc.AlienwareSoundCenter\Modules\ScheduledModules\NahimicOSD.dll", StringComparison.OrdinalIgnoreCase)
    || x.FileName.Equals(@"C:\ProgramData\AWHeadset\DellInc.AlienwareSoundCenter\Modules\ScheduledModules\NahimicOSD.dll", StringComparison.OrdinalIgnoreCase);
});
if (hasInjected)
{
    RenderOptions.ProcessRenderMode = RenderMode.SoftwareOnly;
}
```

#### 方法二：黑名单（需要管理员权限）

NahimicOSD 有一个黑名单文件 BlackApps.dat，在列表里的程序将不会注入破坏其渲染。

路径在：`C:\ProgramData\A-Volute\DellInc.AlienwareSoundCenter\Modules\ScheduledModules\Configurator\BlackApps.dat`

虽然扩展名是 `dat` 但这是一个标准的文本文件，截取开头的一部分如下：

```plaintext
firefox.exe
chrome.exe
iexplore.exe
opera.exe
steam.exe
gameoverlayui.exe
vlc.exe
skype.exe
SkypeApp.exe
RzSynapse.exe
MovieMaker.exe
Photoshop.exe
blender.exe
googleearth.exe
XBMC.exe
BOXEE.exe
```

你的程序可以用管理员权限修改这个文件，在末尾把你的进程名加上。

顺便吐槽一下，这个文件截至目前已有 398 行了，包含了大量常用软件。所以既然开发商早已知道问题这么多，竟然还不修机制，而是通过黑名单来避免冲突，也是扯！

## 调查过程

实际上找到这个原因并没有花太多时间，但多少有些机缘巧合。

1. 一开始，我们查看了用户电脑的型号、CPU/GPU 型号（都是戴尔，十代 CPU）
1. 一开始怀疑的，必然是显卡驱动之类。但反馈此问题的用户中，有一部分是双显卡，换 Intel 的换 NVidia 的都不能修掉此问题，重新去官网下载安装最新版本的驱动亦不能解决掉该问题。所以直接排除此原因。
1. 随后，我们将一些其他的 WPF 程序放到用户的电脑上运行（比如 dnSpy、[WPF Performance Suite](http://blog.walterlv.com/post/wpf-rendering-dirty-region.html)），结果都会糊掉。
1. 随后，我们又将一个 DirectX 9 Demo 放到用户电脑上运行，然而 Demo 画面一直在动，我们又是远程调查的，所以难以判定这种糊到底是因为远程软件导致还是确实已经糊了，所以此步骤没有收获。

在我们即将放弃之时，又一个新用户反馈了问题，与之前多个用户反馈不同的是，这是一台联想的电脑，八代 CPU，这样看来似乎又没有什么共性。

林德熙提醒我可能跟 [Button renders wrong after mouse leave · Issue #707 · dotnet/wpf](https://github.com/dotnet/wpf/issues/707) 问题有关。于是我们立刻拿下 DUMP 文件，果然发现了可疑文件：

![可疑文件](/static/posts/2021-11-05-20-33-36.png)

一台联想的电脑上出现戴尔的模块，并且注入到了进程里面，实在是另人怀疑。并且 OSD 全称为 on-screen display，即屏幕叠加显示，更加值得怀疑。

所以我们就在用户电脑上删除了 NahimicOSD 文件，重启程序，果然一切恢复正常。

再联系用户，才发现原来用户近期插了一部外星人耳机，于是自动安装上了驱动。

## 其他讨论

在看到这篇讨论（[Button renders wrong after mouse leave · Issue #707 · dotnet/wpf](https://github.com/dotnet/wpf/issues/707)）后我惊呆了，因为我两年前其实也参加了讨论，但是竟然一点印象都没有，还在这里调查了这么久。如果大家有其他需要讨论的地方，也可以去那里讨论。

---

**参考资料**

- [Button renders wrong after mouse leave · Issue #707 · dotnet/wpf](https://github.com/dotnet/wpf/issues/707)


