---
title: Git 教程 - 起步
date: 2018-04-01 10:10:00
categories: Git
tags: [Git 教程, Git 安装, Git 基础]
---

Linux 内核开源项目有着为数众广的参与者。 绝大多数的 Linux 内核维护工作都花在了提交补丁和保存归档的繁琐事务上（1991－2002年间）。 到 2002 年，整个项目组开始启用一个专有的分布式版本控制系统 BitKeeper 来管理和维护代码。

到了 2005 年，开发 BitKeeper 的商业公司同 Linux 内核开源社区的合作关系结束，他们收回了 Linux 内核社区免费使用 BitKeeper 的权力。 这就迫使 Linux 开源社区（特别是 Linux 的缔造者 Linus Torvalds）基于使用 BitKeeper 时的经验教训，开发出自己的版本系统。 他们对新的系统制订了若干目标：

- 速度
- 简单的设计
- 对非线性开发模式的强力支持（允许成千上万个并行开发的分支）
- 完全分布式
- 有能力高效管理类似 Linux 内核一样的超大规模项目（速度和数据量）

自诞生于 2005 年以来，Git 日臻成熟完善，在高度易用的同时，仍然保留着初期设定的目标。 它的速度飞快，极其适合管理大项目，有着令人难以置信的非线性分支管理系统。

<!--more-->

## 安装
在你开始使用 Git 前，需要将它安装在你的计算机上。 即便已经安装，最好将它升级到最新的版本。

### Linux
使用 yum：`sudo yum install git`
使用 apt-get：`sudo apt-get install git`

### Mac 上安装
可以使用二进制安装程序。 官方维护的 OSX Git 安装程序可以在 Git 官方网站下载，网址为 http://git-scm.com/download/mac。

### Windows 上安装
官方版本可以在 Git 官方网站下载。 地址为 http://git-scm.com/download/win
简单的方法是安装 GitHub for Windows。 地址为 https://desktop.github.com/


## Git 基础
在开始学习 Git 的时候，请努力分清你对其它版本管理系统的已有认识，如 Subversion 和 Perforce 等；这么做能帮助你使用工具时避免发生混淆。 Git 在保存和对待各种信息的时候与其它版本控制系统有很大差异，尽管操作起来的命令形式非常相近，理解这些差异将有助于防止你使用中的困惑。

### 直接记录快照，而非差异比较
相比传统的版本管理系统（CVS、Subversion、Perforce、Bazaar 等等），它们将保存的信息看作是一组基本文件和每个文件随时间逐步累积的差异。
反之，Git 更像是把数据看作是对小型文件系统的一组快照。每次你提交更新，或在 Git 中保存项目状态时，它主要对当时的全部文件制作一个快照并保存这个快照的索引。 为了高效，如果文件没有修改，Git 不再重新存储该文件，而是只保留一个链接指向之前存储的文件。 Git 对待数据更像是一个快照流。

### 近乎所有操作都是本地执行
因为你在本地磁盘上就有项目的完整历史，所以在 Git 中的绝大多数操作都只需要访问本地文件和资源，一般不需要来自网络上其它计算机的信息。
例如：
- 你可以不需外连到服务器去获取历史，浏览项目的历史。
- 离线或者没有 VPN 时，几乎可以进行任何操作。

### Git 保证完整性
Git 中所有数据在存储前都计算校验和，然后以校验和来引用。这意味着不可能在 Git 不知情时更改任何文件内容或目录内容。 这个功能建构在 Git 底层，是构成 Git 哲学不可或缺的部分。 若你在传送过程中丢失信息或损坏文件，Git 就能发现。

### Git 一般只添加数据
你执行的 Git 操作，几乎只往 Git 数据库中增加数据。 很难让 Git 执行任何不可逆操作，或者让它以任何方式清除数据。一旦你提交快照到 Git 中，就难以再丢失数据。

### 三种状态
Git 有三种状态，你的文件可能处于其中之一：
- 已提交（committed），表示数据已经安全的保存在本地数据库中。
- 已修改（modified），表示修改了文件，但还没保存到数据库中。
- 已暂存（staged），表示对一个已修改文件的当前版本做了标记，使之包含在下次提交的快照中。

Git 项目的三个工作区域的概念：
- Git 仓库，是 Git 用来保存项目的元数据和对象数据库的地方。
- 工作目录，工作目录是对项目的某个版本独立提取出来的内容。
- 暂存区域，是一个文件，保存了下次将提交的文件列表信息。

工作目录中保存着的特定版本文件，就属于已提交状态。
如果作了修改并颐和园已放入暂存区域，就属于已暂存状态。
如果自上次取出后，作了修改但还没有放到暂存区域，就是已修改状态。


## 环境初始化
Git 有多种使用方式。 你可以使用原生的命令行模式，也可以使用 GUI 模式。 但只有在命令行模式下你才能执行 Git 的 所有 命令。

### 获得帮助
使用 Git 命令行时有三种方法可以找到 Git 命令的使用手册：

```
$ git help <verb>
$ git <verb> --help
$ man git-<verb>
```

例如，如果想要获得 config 命令的手册，可以: `$ git help config`

### 初次运行配置
初次安装 Git 的系统上，首先需要做几件事来定制你的 Git 环境。 每台计算机上只需要配置一次。

Git 自带一个 git config 的工具来帮助设置控制 Git 外观和行为的配置变量。 这些变量存储在三个不同的位置：

- /etc/gitconfig 文件，包含系统上每一个用户及他们仓库的通用配置。
- ~/.gitconfig 或 ~/.config/git/config 文件，只针对当前用户。 可以传递 --global 选项让 Git 读写此文件。
- 当前使用仓库的 Git 目录中的 config 文件（就是 .git/config），针对该仓库的配置。

每一个级别覆盖上一级别的配置，所以 .git/config 的配置变量会覆盖 /etc/gitconfig 中的配置变量。

### 用户信息
当安装完 Git 应该做的第一件事就是设置你的用户名称与邮件地址。 这样做很重要，因为每一个 Git 的提交都会使用这些信息，并且它会写入到你的每一次提交中，不可更改：

```
$ git config --global user.name "Veinin Guo"
$ git config --global user.email veininguo@gmail.com
```

### 检查配置信息
可以使用 git config --list 命令来列出所有 Git 当时能找到的配置。

```
$ git config --list
rebase.autosquash=true
credential.helper=manager
user.name=veinin
user.email=veininguo@gmail.com
...
```

你也可以通过输入 `git config <key>` 来检查 Git 的某一项配置。

```
$ git config user.name
veinin
```