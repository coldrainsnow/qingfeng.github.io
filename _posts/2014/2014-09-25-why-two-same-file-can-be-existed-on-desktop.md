---
title:  "为什么桌面上可能有两个同名文件？"
date:   2014-09-25 18:11:00 +0800
tags: windows
permalink: /windows/2014/09/25/why-two-same-file-can-be-existed-on-desktop.html
coverImage: /static/posts/2014-09-25-two-same-files.png
---

是否见过桌面上同时存在两个同名的文件？这其实是可能的。

---

我们见过这样的情况：

![两个 desktop.ini](/static/posts/2014-09-25-two-same-files.png)

当打开隐藏文件和系统文件时，桌面上会看到这样两个同名的文件。

虽然 Windows 不允许在一个文件夹中同时存在两个同名的文件，但我们知道“桌面”不是普通的文件夹。
在默认的 Windows 系统设置中，桌面上显示的图标不仅来自于当前用户帐户专有的“桌面”配置文件夹，
也来自于所有用户帐户共有的“公共桌面”配置文件夹。前者提供的图标仅在当前用户帐户的桌面上显示；
后者提供的图标在所有用户帐户的桌面上显示。由于这两个“桌面”配置文件夹都有自己的 Desktop.ini，
所以当我们允许显示隐含的文件时，两个 Desktop.ini 都将出现在桌面上。

![属性面板](/static/posts/2014-09-25-attributes.png)

请注意“对象名称”，其中指示两个文件存在于不同路径。

