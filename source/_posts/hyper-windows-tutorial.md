---
title: 玩转终端软件 Hyper
date: 2018-11-30 01:20:00
categories: Hyper
tags: [Hyper, Shell, Zsh, CMD]
---

相信有很多人开发环境是我 Windows 上面，有时候又不得不面临在 Windows 上面开发 Linux 环境的程序。还好 Windows 10 系统为我们提供了一个叫 WSL（Windows Subsystem for Linux） 的子系统。但是，尽管微软已经很努力得在改善 Windows 开发环境了，但我们却始终得不到一个比较简单易用得终端软件，最起码你用 Window 自带得终端来操作 Linux 系统还是会觉得缺了点什么。

目前市面上很多终端软件都比 Windows 自带终端好太多，比如这篇文章就介绍了 20 个比较好用终端软件：

[THE BEST 1 OF 20 OPTIONS](https://www.slant.co/topics/1552/~best-terminal-emulators-for-windows)

之前用过一个不错得软件叫做 [Cmder](http://cmder.net/)，但它不是今天得主角，我们今天要介绍得是另一款更符合我口味得终端软件 Hyper。

## 获取并安装 Hyper

你可以在一下几个地方获取到 Hyper：

- 官方网站：[入口](http://cmder.net/)
- Github：[入口](https://github.com/zeit/hyper)

下载到二进制安装包后，直接安装就可以运行，本文运行环境是 Windows。

## 基础插件安装

Hyper 本身是支持插件系统的，在安装扩展插件时，你需要先安装 Hyper 命令行程序，你可以点击菜单，选择 `Plugin` 选项，点击 `Install Hyper CLI command in PATH`。

![install](/images/hyper/insall-path.png)

其官方在首页推荐了4个比较易用的插件，推荐直接全部安装。安装地址请点击 [链接](https://hyper.is/plugins)。其中4款插件分别为：

- [hypercwd](https://hyper.is/plugins/hypercwd)，它可以让你的终端在新建标签页时，保持上一个终端的目录地址。安装命令： `hyper i hypercwd`。
- [hyper-search](https://hyper.is/plugins/hyper-search)，它可以让你搜索整个终端的所有文本内容。安装命令： `hyper i hyper-search`。
- [hyper-pane](https://hyper.is/plugins/hyper-pane)，增强窗口导航，对于支持多个页签的软件来说这是个利器。安装命令： `hyper i hyper-pane`。
- [hyperpower](https://hyper.is/plugins/hyper-pane)，这是一个让你的终端变得更加绚丽的插件，当然这么炫酷的效果是不是会影响你的工作呢，还是要视个人情而定。安装命令： `hyper i hyperpower`。

![hyperpower](https://cloud.githubusercontent.com/assets/13041/16820268/13c9bfe6-4905-11e6-8fe4-baf8fc8d9293.gif)

<!--more-->

## 主题插件安装

Hyper 支持插件系统，自然就少不了主题了，官方首页推荐了4种同颜色的主题，你可以直接在[主题地址](https://hyper.is/themes)查看并安装。

![install](/images/hyper/themes.png)

当然，上面的满足不了你，你也可以去在上面的主题地址中的页签 `NEWEST` 找到更多主题，或者你也可以直接去 GitHub 搜索，抑或是你自己写一个主题插件。总之，你最终会找到一款符合你的主题皮肤。

## 其他插件

GitHub 有一个项目叫做 [awesome-hyper](https://github.com/bnb/awesome-hyper)，其列出了 Hyper 社区中精选的软件包、主题和相关有趣的资源列表，比如下面这款当你输入 `git push` 成功执行后，终端会出现火箭动画：

![git-push](https://user-images.githubusercontent.com/6589909/28026422-9dc92218-655b-11e7-8852-3ee8d57c87d5.gif)

## Hyper 集成 WSL

通常我们在使用 WSL 工作时，希望启动终端时就进入 WSL bash 内部，Windows 中有一个 base.exe 程序可以让我们在启动终端后输入 `bash` 后进入 WSL 终端。
但这一步实在太繁琐了，有了 Hyper，我们只需要改动一下配置文件，上面这步就可以直接省略。

现在，你只需打开 Hyper 设置页面文件，这一步需要从菜单 Edit -> Preferences 进入，然后编辑 `shell` 和 `shellArgs` 参数：

``` shell
shell: 'C:\\Windows\\System32\\wsl.exe',
shellArgs: [],
```

再次从新打开 Hyper，你会发现默认已经进入了 base 终端页面。

## 使用 zsh 代替 bash

通常 Linux 服务器上面默认使用的命令模式是 `bash`，除了 `bash` 之外，还有很多其他诸如 `zsh`、`csh`、`fish`等，这其中，我认为 `zsh` 增强功能（标签完成和拼写错误修正）会很吸引人。

### 安装 zsh 和 oh-my-zsh

虽然 `zsh` 配置很繁琐，但这一步已经有人帮你完成了，GitHub 上面有一个项目叫做 [oh-my-zsh](https://github.com/robbyrussell/oh-my-zsh) 就是专门用来管理配置 `zsh` 的一个框架。

在使用 `oh-my-zsh` 之前，你需要安装 `zsh`，各个平台安装的方式可能不一样，你可以通过这个 [链接](https://github.com/robbyrussell/oh-my-zsh/wiki/Installing-ZSH) 查看 `zsh` 安装方式。

有了 `zsh` 后，要安装 `oh-my-zsh`，你还需要用 `wget` 或者 `curl` 来辅助安装：

- 通过 curl 安装，`sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"`
- 通过 wget 安装，`sh -c "$(wget https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"`

安装完成后，你可以通过命令 `chsh -s /bin/zsh` 设置 `zsh` 为默认的 shell。

### 安装自动补全功能

使用 `zsh` 我认为第一个吸引我的重要插件就是 [zsh-autosuggestions](https://github.com/zsh-users/zsh-autosuggestions)。它会自动为你的终端命令提供补全建议，让你能更加快速的完成命令输入，有了它，你再也不用一遍遍的按 `tab` 来加快你的命令输入了。

`zsh-autosuggestions` 安装文档你可以点击这个[链接](https://github.com/zsh-users/zsh-autosuggestions/blob/master/INSTALL.md) 进入查看，比如使用 Git 安装步骤如下：

使用 Git 把项目从仓库 Clone 下来：

``` shell
git clone https://github.com/zsh-users/zsh-autosuggestions ~/.zsh/zsh-autosuggestions
```

将以下内容添加到 `.zshrc` 文件内：

``` shell
source ~/.zsh/zsh-autosuggestions/zsh-autosuggestions.zsh
```

重新开启终端会话，你就可以享受 `zsh-autosuggestions` 给你带来的便利了。

![install](/images/hyper/auosuggestions.gif)

### 配置主题

`Oh My Zsh` 附带了大量的插件和主题，你可以利用它们可以进一步增强和美化你的终端功能。要想配置主题和插件，你可以直接修改 `~/.zshrc` 文件。

对于插件启用，你可以编辑 `plugins` 输入你想要加载的插件：

``` shell
plugins =（
  git
  bundler
  dotenv
  osx
  rake
  rbenv
  ruby
）
```

对于主题，你也可以编辑文件 `~/.zshrc`，在 `ZSH_THEME` 修改主题名称即可，链接 [External themes](https://github.com/robbyrussell/oh-my-zsh/wiki/External-themes) 提供了大部分主题的样式，你可以直接修改参数后即可看到主题效果，比如 `agnoster` 主题：

``` shell
ZSH_THEME="agnoster"
```

修改完成后，你的终端会变成这个样子：

![install](/images/hyper/agnoster.png)

请注意，很多主题如果你无法显示预期效果，那是因为响应的主题系统字体没有安装，你需要正确的安装了 [Powerline字体](https://github.com/powerline/fonts) 字体才会显示出来。

## 总结

本文详细阐述了如何安装 Hyper 终端软件，其中包括配置 Hyper 插件和主题设置，并讲述了如何使用 Hyper 完全代替 Windows 的默认终端软件 CMD。
最后，介绍了一种使用 zsh 来代替 bash 的方法，你可以让你的终端功能更加强大易用。
用一句时髦的话来总结：Hyper 长得很漂亮哦。