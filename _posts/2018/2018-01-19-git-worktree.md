---
title: "再也不用克隆多个仓库啦！git worktree 一个 git 仓库可以连接多个工作目录"
publishDate: 2018-01-19 09:20:06 +0800
date: 2018-09-17 18:45:12 +0800
tags: git
permalink: /post/git-worktree.html
---

我在 `feature` 分支开发得多些，但总时不时被高优先级的 BUG 打断需要临时去 `develop` 分一个分支出来解 BUG。git 2.6 以上开始提供了 worktree 功能，可以解决这样的问题。

阅读本文将了解使用 git worktree 高效进行并行开发的方法。

---

git worktree 从一个仓库中可以创建多个工作目录，方便多开编辑器并行开发。

## 快速上手

```bash
git worktree add -b <新分支名> <新路径> <从此分支创建>
```

例如，你正在某个 `feature` 分支开发，希望从 master 分出一个分支来解决某个紧急的 BUG：

```bash
git worktree add -b t/walterlv/bugfix-100 ../Demo.bugfix master
```

这样，原本的仓库文件夹的同级目录下会出现一个 Demo.bugfix 文件夹（当然名字随便取）。这个仓库里只有一个 .git 文件用来记录这是主仓库的一个工作目录。

自此，这两个工作目录在工作上看起来就像两个独立的仓库一样，都可以运行各种命令，包括切换分支。

另外，你也可以不使用 `-b`，以便直接使用现有的分支，而不创建新的分支：

```bash
git worktree add <新路径> <从此分支创建>
```

例如，你正在某个 `feature` 分支开发，希望回到 master 分支解决某个紧急的 BUG：

```bash
git worktree add ../Demo.bugfix master
```

相比于克隆多个仓库，使用这种方法创建的多个目录，有诸多好处：

1. 只有一个仓库会占用版本库的空间，其它只占用工作目录的空间，对大型项目而言非常节省空间。
1. 因为所有工作目录共享一个仓库，所以一个更新意味着整个更新（A 目录里对分支做的改动，B 目录里切到此分支也是改动后的；避免到时候找不到某个未推送的改动改到了哪个仓库）

## 注意事项

使用 git worktree 创建的多个目录，不能有任何两个目录在同一个分支下——原因应该不言自明。

如果要删除其中一个工作目录，直接删除文件夹即可。随后使用命令清除多余的已经被删的工作目录：

```bash
git worktree prune
```

