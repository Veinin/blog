---
title: Git 教程 - 基础操作
date: 2018-04-01 10:25:00
categories: Git
tags: [Git 教程, Git 安装, Git 基础]
---

在获得 Git 后你需要学会使用 Git 完成各种**工作中**将要使用的各种基本命令。
例如配置并初始化一个仓库（repository）、开始或停止跟踪（track）文件、暂存（stage）或提交（commit)更改。 
更进一步，你应该学会如何配置 Git 来忽略指定的文件和文件模式、如何迅速而简单地撤销错误操作、如何浏览你的项目的历史版本以及不同提交（commits）间的差异、如何向你的远程仓库推送（push）以及如何从你的远程仓库拉取（pull）文件。

## 获取 Git 仓库
取得 Git 项目仓库有两种方法。
- 在现有项目或目录下导入所有文件到 Git 中。
- 从一个服务器克隆一个现有的 Git 仓库。

### 在现有目录中初始化仓库
如果你打算使用 Git 来对现有的项目进行管理，你只需要进入该项目目录并输入：

```
git init
```

<!--more-->

该命令将创建一个名为 .git 的子目录，这个子目录含有你初始化的 Git 仓库中所有的必须文件，这些文件是 Git 仓库的骨干。但你的项目里的文件还没有被跟踪。

如果你已经有一些文件需要让 Git 仓库来进行版本控制的话，你可使用 `git add` 命令来追踪指定文件，然后用 `git commit` 提交：

```
$ git add *.c
$ git add LICENSE
$ git commit -m 'initial project version'
```

如果你需要追踪指定某一类型的文件，`git add` 是支持通配符的，例如所有 `.c` 的文件：`git add *.c`；当然你也可以指定添加某些文件。


## 克隆现有的仓库
如果你想获得一份已经存在了的 Git 仓库的拷贝，你可以使用到 `git clone` 命令。
与其他 VCS 系统（如 Subversion）的 `checkout` 不同，Git 克隆的是该 Git 仓库服务器上的几乎所有数据，而不是仅仅复制完成你的工作所需要文件。
当 `git clone` 执行完后，默认配置下远程 Git 仓库中的每一个文件的每一个版本都将被拉取下来。

克隆仓库的命令格式是 git clone [url]，比如克隆一个 blog 项目：

```
$ git clone git@github.com:Veinin/blog.git
```

如果你想在克隆远程仓库的时候，自定义本地仓库的名字，你可以使用如下命令：

```
$ git clone git@github.com:Veinin/blog.git myblog
```

克隆命令除了支持 `https://` 协议外，你还可以使用 `git://` 协议或者使用 SSH 传输协议 `user@host:path/to/repo.git`


## 记录每次更新到仓库
你工作目录下的每一个文件都不外乎这两种状态：
- 已跟踪，是指那些被纳入了版本控制的文件，在上一次快照中有它们的记录，在工作一段时间后，它们的状态可能处于未修改，已修改或已放入暂存区。
- 未跟踪，已跟踪文件以外的所有其它文件都属于未跟踪文件。

编辑过某些文件之后，由于自上次提交后你对它们做了修改，Git 将它们标记为已修改文件。 我们逐步将这些修改过的文件放入暂存区，然后提交所有暂存了的修改，如此反复。所以使用 Git 时文件的生命周期如下：

### 检查当前文件状态
可以用 `git status` 命令来查看当前文件状态。刚克隆下来的项目，你的工作目录是相当干净的。如果你使用该命令，会得到如下结果：

```
$ git status
On branch master
nothing to commit, working directory clean
```

此时，你的所有追踪文件都是未修改的，该明显显示了当前所在分支，分支名为 `master`。

随后，你可能想创建新文件，比如 README 文件，创建完后，如果你使用 `git status`，你讲看到一个未跟踪的文件：

```
$ echo 'My Project' > README
$ git status
On branch master
Untracked files:
  (use "git add <file>..." to include in what will be committed)

    README

nothing added to commit but untracked files present (use "git add" to track)
```

上面的 `Untracked files` 为未跟踪文件列表，未跟踪的文件意味着 Git 在之前的快照（提交）中没有这些文件。

### 跟踪新文件
使用命令 `git add` 开始跟踪一个文件。 所以，要跟踪 README 文件，运行：

```
$ git add README
```

然后使用 `git status` 命令，你会看到 README 文件已被追踪，且处于暂存状态。

```
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

    new file:   README
```

只要在 `Changes to be committed` 这行下面的文件，就说明是已暂存状态。

### 暂存已修改文件
如果你修改了一个名为 CONTRIBUTING.md 的已被跟踪的文件，然后运行 git status 命令，会看到下面内容：

```
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

    new file:   README

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

    modified:   CONTRIBUTING.md
```

文件 CONTRIBUTING.md 出现在 `Changes not staged for commit` 这行下面，说明已跟踪文件的内容发生了变化，但还没有放到暂存区。
要暂存这次更新，需要运行 `git add` 命令。这是个多功能命令：
- 可以用它开始跟踪新文件。
- 把已跟踪的文件放到暂存区。
- 用于合并时把有冲突的文件标记为已解决状态等。

我们运行 git add 将"CONTRIBUTING.md"放到暂存区，文件又会变成已暂存状态，然后再看看 git status 的输出：

```
$ git add CONTRIBUTING.md
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

    new file:   README
    modified:   CONTRIBUTING.md
```

假设此时，你想要在 CONTRIBUTING.md 里再加条注释， 重新编辑存盘后，准备好提交。 不过且慢，再运行 git status 看看：

```
$ vim CONTRIBUTING.md
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

    new file:   README
    modified:   CONTRIBUTING.md

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

    modified:   CONTRIBUTING.md
```

现在 CONTRIBUTING.md 文件同时出现在暂存区和非暂存区。实际上 Git 只不过暂存了你运行 `git add` 命令时的版本。
如果你现在提交，CONTRIBUTING.md 的版本是你最后一次运行 git add 命令时的那个版本，而不是你运行 git commit 时，在工作目录中的当前版本。
所以，运行了 git add 之后又作了修订的文件，需要重新运行 git add 把最新版本重新暂存起来：

```
$ git add CONTRIBUTING.md
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

    new file:   README
    modified:   CONTRIBUTING.md
```

### 状态简览
git status 命令的输出十分详细，但其用语有些繁琐。你可以使用 git status -s 命令或 git status --short 命令，你将得到一种更为紧凑的格式输出。

```
$ git status -s
 M README
M  lib/simplegit.rb
MM Rakefile
A  lib/git.rb
?? LICENSE.txt  
```

简介标识分为 5 种：
- ` M`，出现在右边表示该文件被修改了但是还没放入暂存区。
- `M `，出现在靠左边表示该文件被修改了并放入了暂存区。 
- `MM`，表示被修改并提交到暂存区后又在工作区中被修改了，所以在暂存区和工作区都有该文件被修改了的记录。
- `??`，新增未追踪文件。
- `A` ，新添加到暂存区的文件。

### 查看已暂存和未暂存的修改
`git status` 命令显示可能还过于模块，但你可以用 `git diff` 命令来查看你具体修改了什么地方。

查看尚未暂存的文件更新了哪些部分，不加参数直接输入 `git diff`：

```
$ git diff
diff --git a/README.md b/README.md
index 4464ea4..83bfa15 100644
--- a/README.md
+++ b/README.md
```

若要查看已暂存的将要添加到下次提交里的内容，可以用 `git diff --cached` 命令：

```
$ git add README.md
$ git diff --cached
diff --git a/README.md b/README.md
index 4464ea4..83bfa15 100644
--- a/README.md
+++ b/README.md
```

如果对上面暂存 `README.md` 文件再编辑，运行 `git status` 会看到暂存前后的两个版本。运行 `git diff` 将看到暂存前后的变化，而用 `git diff --cached` 将查看已经暂存起来的变化。

### 忽略文件
一般我们总会有些文件无需纳入 Git 的管理，也不希望它们总出现在未跟踪文件列表。 通常都是些自动生成的文件，比如日志文件，或者编译过程中创建的临时文件等。
可以创建一个名为 `.gitignore` 的文件，列出要忽略的文件模式。 

文件 .gitignore 的格式规范如下：
- 所有空行或者以 # 开头的行都会被 Git 忽略。
- 可以使用标准的 glob（shell 正则表达式） 模式匹配。例如星号（*）匹配零个或多个任意字符；[abc] 匹配任何一个列在方括号中的字符等等。
- 匹配模式可以以（/）开头防止递归。
- 匹配模式可以以（/）结尾指定目录。
- 要忽略指定模式以外的文件或目录，可以在模式前加上惊叹号（!）取反。

```
$ cat .gitignore
*.a
!lib.a
/TODO
build/
doc/*.txt
doc/**/*.pdf
```

### 提交更新
如果暂存区域已经准备妥当可以提交了。准备提交前，先用 `git status` 看下，是不是都已暂存起来了， 然后再运行提交命令 `git commit`：

```
git commit
```

这种方式会启动文本编辑器以便输入本次提交的说明。

```
# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
# On branch master
# Changes to be committed:
#	new file:   README
#	modified:   CONTRIBUTING.md
#
~
".git/COMMIT_EDITMSG" 9L, 283C
```

默认的提交消息包含最后一次运行 `git status` 的输出，放在注释行里，另外开头还有一空行，供你输入提交说明。

另外，你也可以在 commit 命令后添加 -m 选项，将提交信息与命令放在同一行，如下所示：

```
$ git commit -m "modify the README file"
[master 463dc4f] modify the README file
 2 files changed, 2 insertions(+)
 create mode 100644 README
```

### 跳过使用暂存区域
尽管使用暂存区域的方式可以精心准备要提交的细节，但有时候这么做略显繁琐。 Git 提供了一个跳过使用暂存区域的方式， 只要在提交的时候，给 `git commit` 加上 `-a` 选项，Git 就会自动把所有已经跟踪过的文件暂存起来一并提交，从而跳过 `git add` 步骤：

### 移除文件
要从 Git 中移除某个文件，就必须要从已跟踪文件清单中移除，然后再提交。 可以用 `git rm` 命令完成此项工作，并连带从工作目录中删除指定的文件，这样以后就不会出现在未跟踪文件清单中了。

如果只是简单地从工作目录中手工删除文件，运行 git status 时就会在 “Changes not staged for commit” 部分：

```
$ rm PROJECTS.md
$ git status
On branch master
Your branch is up-to-date with 'origin/master'.
Changes not staged for commit:
  (use "git add/rm <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        deleted:    PROJECTS.md
```

然后再运行 git rm 记录此次移除文件的操作。注意，如果删除之前修改过并且已经放到暂存区域的话，则必须要用强制删除选项 `-f`：

```
$ git rm PROJECTS.md
rm 'PROJECTS.md'
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

    deleted:    PROJECTS.md
```

另外一种情况是，我们想把文件从 Git 仓库中删除（亦即从暂存区域移除），但仍然希望保留在当前工作目录中。为达到这一目的，使用 --cached 选项：

```
$ git rm --cached README
```

git rm` 命令后面可以列出文件或者目录的名字，也可以使用 glob 模式。 比方说：

```
$ git rm log/\*.log
```

### 移动文件
Git 并不显式跟踪文件移动操作。 如果在 Git 中重命名了某个文件，仓库中存储的元数据并不会体现出这是一次改名操作。Git 并不显式跟踪文件移动操作。 如果在 Git 中重命名了某个文件，仓库中存储的元数据并不会体现出这是一次改名操作。不过 Git 非常聪明，它会推断出究竟发生了什么。

要在 Git 中对文件改名，可以使用 `git mv` 命令：

```
$ git mv README.md README
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

    renamed:    README.md -> README
```

它会恰如预期般正常工作。但实际上运行 `git mv` 相当于运行了以下 3 条命令：

```
$ mv README.md README
$ git rm README.md
$ git add README
```


## Git 查看提交历史
你也许想回顾下提交历史，可以使用 `git log` 命令来查看。默认不用任何参数的话，git log 会按提交时间列出所有的更新，最近的更新排在最上面。

```
$ git log
commit 2f2ff67cb4e8731dad9bb1b8ec4f0963c0368870
Author: Veinin Guo <Veinin@users.noreply.github.com>
Date:   Fri Mar 30 00:46:23 2018 +0800

    modify the theme

commit 633e5f7e0cbf87fba2e922956667c9cb627bd4a0
Author: Veinin Guo <Veinin@users.noreply.github.com>
Date:   Fri Mar 30 00:39:54 2018 +0800

    initialize commit

...
```

git log 有许多选项可以帮助你搜寻你所要找的提交：
- 使用选项 `-p`，用来显示每次提交的内容差异。 
- 可以加上 `-2` 来仅显示最近两次提交。
- 使用 `--stat` 选项查看每次提交的简略的统计信息。
- 使用 `--pretty` 这个选项可以指定使用不同于默认格式的方式展示提交历史。 比如用 `--pretty=oneline` 将每个提交放在一行显示。另外还有 short，full 和 fuller 可以用
- 使用 `--pretty=format` 来定制要显示的记录格式，具体格式可以参考 Git 帮助文档。
- 按照时间作限制的选项 `--since` 和 `--until`，比如列出最近两周内的提交记录：`$ git log --since=2.weeks`；或具体到某一天：`git log --since=2008-03-30`。


## 撤销操作
在任何一个阶段，你都有可能想要撤消某些操作。

有时候我们提交完了才发现漏掉了几个文件没有添加，或者提交信息写错了。 此时，可以运行带有 --amend 选项的提交命令尝试重新提交：
在上次提交后马上执行了此命令，那么快照会保持不变，而你所修改的只是提交信息。例如，你提交后发现忘记了暂存某些需要的修改，可以像下面这样操作：

```
$ git commit -m 'initial commit'
$ git add forgotten_file
$ git commit --amend
```

### 取消暂存的文件
你已经修改了两个文件并且想要将它们作为两次独立的修改提交，但是却意外地输入了 git add * 暂存了它们两个。 如果你想取消暂存的指定文件，可以使用 `git reset HEAD <file>...` 来操作。

```
$ git status
On branch master
Your branch is up-to-date with 'origin/master'.
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        modified:   README.md

$ git reset HEAD README.md
```

### 撤消对文件的修改
如果你并不想保留对某个文件的修改，可以使用 `git checkout -- <file>...` 将文件还原成上次提交时的样子。**请注意**，这个命令是非常危险，除非你确实不想修改那个文件，否则不要使用这个命令。

```
$ git status
On branch master
Your branch is up-to-date with 'origin/master'.
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        modified:   README.md

$ git checkout -- README.md
```


## 远程仓库的使用
远程仓库是指托管在因特网或其他网络中的你的项目的版本库。 你可以有好几个远程仓库，通常有些仓库对你只读，有些则可以读写。 与他人协作涉及管理远程仓库以及根据需要推送或拉取数据。
管理远程仓库包括了解如何添加远程仓库、移除无效的远程仓库、管理不同的远程分支并定义它们是否被跟踪等等。

### 查看远程仓库
如果想查看你已经配置的远程仓库服务器，可以运行 `git remote` 命令。
如果你已经克隆了自己的仓库，那么至少应该能看到 `origin` - 这是 Git 给你克隆的仓库服务器的默认名字：

```
$ git remote
origin
```

指定选项 -v，会显示需要读写远程仓库使用的 Git 保存的简写与其对应的 URL。如果你的远程仓库不止一个，该命令会将它们全部列出。
```
$ git remote -v
origin  https://github.com/Veinin/blog.git (fetch)
origin  https://github.com/Veinin/blog.git (push)
```

### 添加远程仓库
运行 `git remote add <shortname> <url>` 添加一个新的远程 Git 仓库，同时指定一个你可以轻松引用的简写：

```
$ git remote
origin
$ git remote add pb https://github.com/paulboone/ticgit
$ git remote -v
origin	https://github.com/schacon/ticgit (fetch)
origin	https://github.com/schacon/ticgit (push)
pb	https://github.com/paulboone/ticgit (fetch)
pb	https://github.com/paulboone/ticgit (push)
 ```

现在你可以在命令行中使用字符串 pb 来代替整个 URL。例如，如果你想拉取 Paul 的仓库中有但你没有的信息，可以运行 `git fetch pb`。

### 从远程仓库中抓取与拉取
从远程仓库中获得数据，可以执行：

```
$ git fetch [remote-name]
```

这个命令会访问远程仓库，从中拉取所有你还没有的数据。 必须注意 git fetch 命令会将数据拉取到你的本地仓库 - 它并不会自动合并或修改你当前的工作。 当准备好时你必须手动将其合并入你的工作。

如果你有一个分支设置为跟踪一个远程分支，可以使用 `git pull` 命令来自动的抓取然后合并远程分支到当前分支。

### 推送到远程仓库
当你想分享你的项目时，必须将其推送到上游。 这个命令很简单：`git push [remote-name] [branch-name]`。 当你想要将 master 分支推送到 origin 服务器时，那么运行这个命令就可以将你所做的备份到服务器：

```
$ git push origin master
```

你必须先将他们的工作拉取下来并将其合并进你的工作后才能推送。否则，你的推送就会毫无疑问地被拒绝。

### 查看远程仓库
如果想要查看某一个远程仓库的更多信息，可以使用 `git remote show [remote-name]` 命令。 

```
λ git remote show origin
* remote origin
  Fetch URL: https://github.com/chenshuo/muduo.git
  Push  URL: https://github.com/chenshuo/muduo.git
  HEAD branch: master
  Remote branches:
    backport   new (next fetch will store in remotes/origin)
    cpp11      new (next fetch will store in remotes/origin)
    cpp17      tracked
    cpp98      tracked
    experiment stale (use 'git remote prune' to remove)
    gh-pages   tracked
    mac        tracked
    master     tracked
  Local branch configured for 'git pull':
    master merges with remote master
  Local ref configured for 'git push':
    master pushes to master (up to date)
```

这条命令会列出你执行 `git push`、`git pull` 会推送到哪个分支。
也同样地列出了哪些远程分支不在你的本地，哪些远程分支已经从服务器上移除了，还有当你执行 `git pull` 时哪些分支会自动合并。

### 远程仓库的移除与重命名
如果想要重命名引用的名字可以运行 `git remote rename` 去修改一个远程仓库的简写名。

```
$ git remote rename pb paul
$ git remote
origin
paul
```

如果因为一些原因想要移除一个远程仓库， 可以使用 `git remote rm` ：

```
$ git remote rm paul
$ git remote
origin
```


## 打标签
像其他版本控制系统（VCS）一样，Git 可以给历史中的某一个提交打上标签，以示重要。 比较有代表性的是人们会使用这个功能来标记发布结点（v1.0 等等）。

### 列出标签
在 Git 中列出已有的标签是非常简单直观的。 只需要输入 `git tag`：

```
λ git tag   
v0.1.0      
v0.1.1      
v0.1.2      
...
v0.9.7      
v0.9.8      
v1.0.0      
v1.0.1      
...
v2.0.0-beta 
```

你也可以使用特定的模式查找标签。

```
λ git tag -l v1.0.*
v1.0.0
v1.0.0-rc1
v1.0.0-rc2
v1.0.1
v1.0.2
```

### 创建标签
Git 使用两种主要类型的标签：轻量标签（lightweight）与附注标签（annotated）。

#### 附注标签
附注标签是存储在 Git 数据库中的一个完整对象。其中包含打标签者的名字、电子邮件地址、日期时间；还有一个标签信息；并且可以使用 GNU Privacy Guard （GPG）签名与验证。通常建议创建附注标签，这样你可以拥有以上所有信息。
创建附注标签，只需在在运行 `tag` 命令时指定 `-a` 选项：

```
$ git tag -a v0.1.0 -m "new version" 
$ git tag
v0.1.0
```

使用 `git show` 命令可以看到标签信息与对应的提交信息：

```
λ git show v0.1.0
tag v0.1.0
Tagger: veinin <veininguo@gmail.com>
Date:   Fri Mar 30 18:14:44 2018 +0800

new version

commit 2f2ff67cb4e8731dad9bb1b8ec4f0963c0368870
Author: Veinin Guo <Veinin@users.noreply.github.com>
Date:   Fri Mar 30 00:46:23 2018 +0800

    modify the theme
```

#### 轻量标签
轻量标签本质上是将提交校验和存储到一个文件中，没有保存任何其他信息。 创建轻量标签，不需要使用任何选项，只需要提供标签名字：
如果你只是想用一个临时的标签，或者因为某些原因不想要保存那些信息，可以使用轻量标签。

```
$ git tag v0.0.1
$ git tag
v0.0.1
v0.1.0
```

如果在标签上运行 git show，你不会看到额外的标签信息。 命令只会显示出提交信息：

```
$ git show v0.0.1
commit 2f2ff67cb4e8731dad9bb1b8ec4f0963c0368870
Author: Veinin Guo <Veinin@users.noreply.github.com>
Date:   Fri Mar 30 00:46:23 2018 +0800

    modify the theme
```

### 后期标签
可以对过去的某个提交记录打标签，当你忘记给某个时间点的提交打标签时，你可以使用提交记录的校验和（或部分）来重新打上标签。

```
$ git log --pretty=oneline
2f2ff67cb4e8731dad9bb1b8ec4f0963c0368870 modify the theme
6d434e8c571342290ffa15d8852a9487db63b7e1 modify the theme
633e5f7e0cbf87fba2e922956667c9cb627bd4a0 initialize commit
ba67864882e13b588bb5061f334d8a23b7dc29b3 initialize commit
d5a38f1858b5af96f3338060b8f13071a878b17b first commit

$ git tag -a v0.0.2 633e5f7 -m "initial version"

$ git tag
v0.0.1
v0.0.2
v0.1.0
```

### 共享标签
默认情况下，git push 命令并不会传送标签到远程仓库服务器上。在创建完标签后你必须显式地推送标签到共享服务器上。 这个过程就像共享远程分支一样，你可以运行 `git push origin [tagname]` 命令推送一个标签。

```
$ git push origin v0.1.0
Counting objects: 1, done.
Writing objects: 100% (1/1), 166 bytes | 0 bytes/s, done.
Total 1 (delta 0), reused 0 (delta 0)
To https://github.com/Veinin/blog.git
 * [new tag]         v0.1.0 -> v0.1.0
```

如果想要一次性推送很多标签，也可以使用带有 `--tags` 选项，这将会把所有不在远程仓库服务器上的标签全部传送到那里。

```
λ git push origin --tags
Counting objects: 1, done.
Writing objects: 100% (1/1), 161 bytes | 0 bytes/s, done.
Total 1 (delta 0), reused 0 (delta 0)
To https://github.com/Veinin/blog.git
 * [new tag]         v0.0.1 -> v0.0.1
 * [new tag]         v0.0.2 -> v0.0.2
```
 
### 检出标签
在 Git 中你并不能真的检出一个标签，但你可以指定某个新标签来创建新的分支。

```
$ git checkout -b version2 v0.0.2
Switched to a new branch 'version2'
```

### 删除标签
删除一个标签分两种：
- 删除本地标签：`git tag -d 标签名`
- 删除远程仓库标签：`git push origin :ref/tag/标签名`


```
$ git tag -d v0.1.0
Deleted tag 'v0.1.0' (was 729a196)

$ git push origin :refs/tags/v0.0.1
To https://github.com/Veinin/blog.git
 - [deleted]         v0.0.1
```


## 别名
使用别名小技巧可以使你的 Git 体验更简单、容易、熟悉。
如果不想每次都输入完整的 Git 命令，可以通过 git config 文件来轻松地为每一个命令设置一个别名。

```
$ git config --global alias.co checkout
$ git config --global alias.br branch
$ git config --global alias.ci commit
$ git config --global alias.st status
$ git config --global alias.unstage 'reset HEAD --'
$ git config --global alias.last 'log -1 HEAD'
```

有了这些别名后，当你输入 `git co` 时，它就对应了 `git checkout` 命令，所以，如果你把你常用的一些命令设置为别名，那将对你很有帮助。

## 总结
现在，你可以完成所有基本的 Git 本地操作：创建或者克隆一个仓库、做更改、暂存并提交这些更改、浏览你的仓库从创建到现在的所有更改的历史。
下一步，本书将介绍 Git 的杀手级特性：分支模型。