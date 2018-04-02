---
title: Git 教程 - 分支
date: 2018-04-01 14:26:00
categories: Git
tags: [Git 教程, Git 安装, Git 分支]
---

有人把 Git 的分支模型称为它的"必杀技特性"，也正因为这一特性，使得 Git 从众多版本控制系统中脱颖而出。Git 处理分支的方式可谓是难以置信的轻量，创建新分支这一操作几乎能在瞬间完成，并且在不同分支之间的切换操作也是一样便捷。 与许多其它版本控制系统不同，Git 鼓励在工作流程中频繁地使用分支与合并，哪怕一天之内进行许多次。 理解和精通这一特性，你便会意识到 Git 是如此的强大而又独特，并且从此真正改变你的开发方式。

## 分支简介
前面提过，Git 保存的不是文件的变化或者差异，而是一系列不同时刻的文件快照。在进行提交操作时，Git 会保存一个提交对象（commit object）。

该提交对象还包含了作者的姓名和邮箱、提交时输入的信息以及指向它的父对象的指针。首次提交产生的提交对象没有父对象，普通提交操作产生的提交对象有一个父对象，而由多个分支合并产生的提交对象有多个父对象。

Git 的分支，其实本质上仅仅是指向提交对象的可变指针。Git 的默认分支名字是 master。 在多次提交操作之后，你其实已经有一个指向**最后**那个提交对象的 master 分支。它会在每次的提交操作中自动向前移动。

![](/images/git/branch-and-history.png)

<!--more-->


## 分支创建
Git 使用命令 `git branch` 创建一个新分支，创建新分支后，会在当前所在的提交对象上创建一个新指针：

```
$ git branch testing
```

![](/images/git/head-to-master.png)

如图，分支会有一个名为 HEAD 的特殊指针，用来标明当前所处的分支。

可以使用 `git log` 查看各个分支当前所指的对象：

```
git log --oneline --decorate
2f2ff67 (HEAD -> master, origin/master, testing) modify the theme
```


## 分支切换
`git branch` 命令仅仅是创建一个新分支，并不会自动切换到新分支中去。如果要切换到一个已存在的分支，你需要使用 git checkout 命令：

```
$git checkout testing
Switched to branch 'testing'

$ git log --oneline --decorate
2f2ff67 (HEAD -> testing, origin/master, master) modify the theme
```

通过 `git log` 再次查看后，我们可以看到 HEAD 指针已经指向了刚刚创建的新分支 `testing`。

![](/images/git/head-to-testing.png)

如果我们在当前所处分支进行提交，那么 HEAD 分支随着提交操作自动向前移动，HEAD 仍然指向运行 git checkout 时所指的对象。

```
$ vim README.md
$ git commit -a -m 'made a change'
```

![](/images/git/advance-testing.png)

这是如果我们再切换回 master 分支，并且更改后提交，那么在 master 分支也会产生新的提交对象：

```
$ git checkout master
Switched to branch 'master'
$ vim README.md
$ git commit -a -m 'made other changes'
```

![](/images/git/advance-master.png)

以上各个分支切换工作，你只需要熟悉 `branch`、`checkout` 和 `commit` 命令，就可以在不同分支间来回切换和工作，并在时机成熟后合并它们。

最后，你可以通过 `git log` 来查看分叉历史，他会输出你提交的历史、各个分支指向的分支分叉情况：

```
λ git log --oneline --decorate --graph --all
* 92cd086 (HEAD -> master) made other changes
| * b5c6792 (testing) made a change
|/
* 2f2ff67 (origin/master) modify the theme
```


## 分支的新建与合并
Git 在当前所处分支中使用 `git merge` 命令来合并其他分支。

实际工作中你可能会用到类似的工作流。 你将经历如下步骤：
- 开发某个网站。为实现某个新的需求，创建一个分支。
```
$ git checkout -b iss53
Switched to a new branch "iss53"
```

- 在这个分支上开展工作。
```
$ vim index.html
$ git commit -a -m 'added a new footer [issue 53]'
```

正在此时，你突然接到一个电话说有个很严重的问题需要紧急修补。 你将按照如下方式来处理：
- 切换到你的线上分支（production branch）。
```
$ git checkout master
Switched to branch 'master'
```

- 为这个紧急任务新建一个分支，并在其中修复它。
```
$ git checkout -b hotfix
Switched to a new branch 'hotfix'
$ vim index.html
$ git commit -a -m 'fixed the broken email address'
```

- 在测试通过之后，切换回线上分支，然后合并这个修补分支，并将改动推送到线上分支,最后删除 `hotfix` 分支。
```
$ git checkout master
$ git merge hotfix
Updating f42c576..3a0874c
Fast-forward
 index.html | 2 ++
 1 file changed, 2 insertions(+)
$ git branch -d hotfix
Deleted branch hotfix (3a0874c).
```

- 切换回你最初工作的分支上，继续工作。
```
$ git checkout iss53
Switched to branch "iss53"
```

在上面这个典型的使用 Git 分支的工作流中，我们使用了一个带有 -b 参数的 git checkout 命令来创建一个新分支，它等价于执行了`git branch` 和 `git checkout` 两个命令。


## 分支合并冲突解决
在分支合并的适合，如果涉及到同一个文件的同一处，在合并它们的时候就会产生合并冲突：
```
$ git merge iss53
Auto-merging index.html
CONFLICT (content): Merge conflict in index.html
Automatic merge failed; fix conflicts and then commit the result.
```

此时 Git 做了合并，但是没有自动地创建一个新的合并提交。 Git 会暂停下来，等待你去解决合并产生的冲突。 你可以在合并冲突后的任意时刻使用 git status 命令来查看那些因包含合并冲突而处于未合并（unmerged）状态的文件：
```
$ git status
On branch master
You have unmerged paths.
  (fix conflicts and run "git commit")

Unmerged paths:
  (use "git add <file>..." to mark resolution)

    both modified:      index.html
```

Git 会在有冲突的文件中加入标准的冲突解决标记，这样你可以打开这些包含冲突的文件然后手动解决冲突。
```
<<<<<<< HEAD:index.html
<div id="footer">contact : email.support@github.com</div>
=======
<div id="footer">
 please contact us at support@github.com
</div>
>>>>>>> iss53:index.html
```

冲突的地方使用`=======`来区分，你所处的 HEAD 指向的版本在这个区段的上半部分，合并过来的分支在这个区段的下半部分。

你可以修改冲突的地方，保留你认为何时的内容，并且删除冲突的其余部分，如上面例子，我们需要保留下半部分提交内容：
```
<div id="footer">
 please contact us at support@github.com
</div>
```

在所有冲突解决完成后，你可以再次输入 `git commit` 来完成这次合并提交。


## 分支管理
git branch 命令不只是可以创建与删除分支。 如果不加任何参数运行它，会得到当前所有分支的一个列表，其中 `master` 分支前的 `*` 号代表当前 HEAD 所指向的分支：
```
$ git branch
* master
  testing
```

如果需要查看每一个分支的最后一次提交，可以运行 git branch -v 命令：
```
$ git branch -v
* master  92cd086 made other changes
  testing b5c6792 made a change
```

使用 -merged 与 --no-merged 这两个选项可以过滤这个列表中已经合并或尚未合并到当前分支的分支，。 
```
$ git branch --no-merged
  testing
$ git merge testing
$ git branch --merged
* master
  testing
```

包含未合并的分支时，尝试使用 git branch -d 命令删除它时会失败，如果你确定要删除并丢弃那些工作，你可以使用 `-D` 选项强制删除。


