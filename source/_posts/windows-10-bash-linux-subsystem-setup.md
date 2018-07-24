---
title: Windows 10 子系统安装指南
date: 2018-05-09 22:50:00
categories: System
tags: [Windows 10 Bash, Linux Subsystem Setup]
---

## 安装 Windows 的 Linux 子系统

### 步骤1 开启开发者模式
菜单栏搜索 -> 设置 -> 针对开发人员 -> 开发者模式 -> 等待下载相关软件
![](/images/win_bash/01.png)

<!--more-->

### 步骤2 开启 Windows 子系统功能
菜单栏搜索 -> 控制面板 -> 程序 -> 勾选 “启动或关闭 Windows 功能” -> 确定 -> 重启电脑
![](/images/win_bash/02.png)

### 步骤3 查看 Bash 程序
菜单栏搜索 -> Bash -> 打开后提示可以通过 Windows 应用商店来安装分发版
![](/images/win_bash/03.png)

### 步骤4 安装 ubuntu 子系统
打开 Windows 应用商店 -> 搜索 ubuntu -> 点击进入并选择安装 -> 等待安装完成
![](/images/win_bash/04.png)

### 步骤5 设置用户名、密码
从 Windows 商店或菜单栏搜索 Ubuntu 程序启动 -> 等待几分钟后 -> 输入子系统的用户名和密码 -> 完成安装
![](/images/win_bash/05.png)


## 升级到最新版本子系统
终端输入：
```
sudo apt update
sudo apt upgrade
```

## Hyper 整合 Bash
在终端你可以使用 bash 命令直接进入ubuntu子系统，但是 Windows 终端对于开发人员来说还是很难用的，更加推荐第三方的终端软件，比如以下两个软件：
- [hyper](https://hyper.is/)
- [cmder](http://cmder.net/)

这里推荐的是 `hyper` 终端软件，它是一个非常漂亮且实用的终端软件，并且提供了一大堆漂亮的插件和主题。

Hyper 设置直接启动 bash 终端步骤：
设置按钮 -> Editor -> Preferences -> 设置文本中找到 shell 一栏 -> 填入 `bash.exe` 路径

设置完毕后，重新打开 `hyper`，会自动执行 bash 命令进入 Linux 子系统。

![](/images/win_bash/06.png)

## VS Code 集成 Bash
VS Code 终端默认使用的终端是 `PowwerShell`，你可以设置终端使用 Bash，这样你就可以在 VS Code 编辑代码后，直接在 Bash 终端里面编译、运行查看结果了。

VS Code 设置默认终端步骤：
文件菜单 -> 首选项 -> 设置 -> 将 `"terminal.integrated.shell.windows": "C:\\Windows\\System32\\bash.exe",` 加入到用户设置 -> 保存

设置完毕后，重新打开终端，可以看到默认打开的是 bash 终端了。

![](/images/win_bash/07.png)