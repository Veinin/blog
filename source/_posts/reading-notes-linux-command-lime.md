---
title: 《Linux 命令行与shell脚本编程大全》读书笔记（命令行部分）
date: 2017-11-24 23:30:00
categories: 读书笔记
tags: [Linux, Linux 命令行]
---

## 什么是 Linux

Linux可划分为以下四部分：
### Linux内核
- 系统内存管理
- 软件程序管理
- 硬件设备管理
- 文件系统管理

### GNU工具
GNU组织（GNU是GNU’s Not Unix的缩写），模仿Unix操作系统开发一系列标准的计算机系统工具，但没有可以运行它们的内核系统。

<!--more-->

- **核心GNU工具**
该项目移植了很多常见的Unix系统命令行工具。供Linux系统使用的这组核心工具被称为coreutils（coreutilities）软件包。由三部分构成：用以处理文件的工具、用以操作文本的工具、用以管理进程的工具

- **shell**
GNU/Linux shell是一种特殊的交互式工具。它为用户提供了启动程序、管理文件系统中的文件以及运行在Linux系统上的进程的途径。 shell的核心是命令行提示符。命令行提示符是shell负责交互的部分。它允许你输入文本命令，然后解释命令，并在内核中执行。所有Linux发行版默认的shell都是bash shell。

### 图形化桌面环境
- X Window系统
- KDE桌面（K Desktop Environment， K桌面环境）
- GNOME桌面（the GNU Network Object Model Environment， GNU网络对象模型环境）
- Unity桌面......

### 应用软件



## Linux 发行版
不同的Linux发行版通常归类为3种：
### 完整的核心Linux发行版
核心Linux发行版含有内核、一个或多个图形化桌面环境以及预编译好的几乎所有能见到的Linux应用。它提供了一站式的完整Linux安装。

### 特定用途的发行版
它们通常基于某个主流发行版，但仅包含主流发行版中一小部分用于某种特定用途的应用程序。如许多特定用途的Linux发行版都是基于Debian Linux，但仅打包了完整Debian系统中的一小部分。

### LiveCD测试发行版
它无需安装就可以看到Linux系统是什么样的。多数现代PC都能从CD启动，而不是必须从标准硬盘启动。基于这点，一些Linux发行版创建了含有Linux样本系统（称为Linux LiveCD）的可引导CD。



## bash 手册
man命令用来访问存储在Linux系统上的手册页面。在想要查找的工具的名称前面输入man命令，就可以找到那个工具相应的手册条目。
手册每个内容区域都分配了一个数字，从1开始，一直到9：
1：可执行程序或shell命令
2：系统调用
3：库调用
4：特殊文件
5：文件格式与约定
6：游戏
7：概览、约定及杂项
8：超级用户和系统管理员命令
9：内核例程



## 过滤输出列表
ls命令能够识别标准通配符，并在过滤器中用它们进行模式匹配：
- 问号（?）代表一个字符；
- 星号（*）代表零个或多个字符。
- 在特定位置上可能出现的两种字符，a或i:
```
ls -l my_scr[ai]pt
```
- 指定字符范围，例如字母范围[a – i]:
```
ls -l f[a-i]ll
```
- 使用感叹号（!）将不需要的内容排除在外:
```
ls -l f[!a]ll
```



## 管理文件目录

### 处理文件
- 创建文件：touch
- 复制文件：cp
- 重命名文件：mv
- 删除文件：rm
- 链接文件：ln
在系统上维护同一文件的两份或多份副本，除了保存多份单独的物理文件副本之外，还可以采用保存一份物理文件副本和多个虚拟副本的方法。这种虚拟的副本就称为链接。

### 文件链接
在Linux中有两种不同类型的文件链接：
#### 符号链接
符号链接就是一个实实在在的文件，它指向存放在虚拟目录结构中某个地方的另一个文件。这两个通过符号链接在一起的文件，彼此的内容并不相同。
使用ln命令以及-s选项来创建符号链接：
```
ln -s data_file sl_data_file
```

#### 硬链接
硬链接会创建独立的虚拟文件，其中包含了原始文件的信息及位置。但是它们从根本上而言是同一个文件。引用硬链接文件等同于引用了源文件。
使用ln命令时不再需要加入额外的参数创建硬链接：
```
ln code_file hl_code_file
```

### 查看文件
- 显示文本文件中所有数据；cat
- 显示文本文件的内容，但会在显示每页数据之后停下来：more
- more命令的升级版：less
- 显示文件最后几行的内容：tail
- 显示文件开头那些行的内容：head

### 处理目录
- 创建目录：mkdir New_Dir
- 创建多个目录和子目录，需要加入-p参数：mkdir -p New_Dir/Sub_Dir/Under_Dir
- 删除目录，只删除空目录：rmdir New_Dir
- 删除目录及其所有内容：rm -rf Small_Dir
- 查看文件类型：file my_file


## 处理数据文件

### 排序、搜索
- 排序数据：sort [options] [file]
- 搜索数据：grep [options] pattern [file]

### 压缩数据
Linux上的文件压缩工具：
- bzip2，采用Burrows-Wheeler块排序文本压缩算法和霍夫曼编码，文件扩展名：.bz2        
- compress，最初的Unix文件压缩工具，已经快没人用了，文件扩展名：.Z  
- gzip，GNU压缩工具，用Lempel-Ziv编码，文件扩展名：.gz 
- zip，Windows上PKZIP工具的Unix实现，文件扩展名：.zip

gzip 软件包是GNU项目的产物，意在编写一个能够替代原先Unix中compress工具的免费版本。这个软件包含有下面的工具：
- gzip：用来压缩文件。
- gzcat：用来查看压缩过的文本文件的内容。
- gunzip：用来解压文件。

### 归档数据
虽然zip命令能够很好地将数据压缩和归档进单个文件，但它不是Unix和Linux中的标准归档工具。目前，Unix和Linux上最广泛使用的归档工具是tar命令。

tar命令的格式：
```
tar function [options] object1 object2 ...
```

function参数定义了tar命令应该做什么：
- -A --concatenate 将一个已有tar归档文件追加到另一个已有tar归档文件
- -c --create 创建一个新的tar归档文件
- -d --diff 检查归档文件和文件系统的不同之处
-   --delete 从已有tar归档文件中删除
- -r --append 追加文件到已有tar归档文件末尾
- -t --list 列出已有tar归档文件的内容
- -u --update 将比tar归档文件中已有的同名文件新的文件追加到该tar归档文件中
- -x --extract 从已有tar归档文件中提取文件

每个功能可用选项来针对tar归档文件定义一个特定行为：
- -C dir 切换到指定目录
- -f file 输出结果到文件或设备file
- -j 将输出重定向给bzip2命令来压缩内容
- -p 保留所有文件权限
- -v 在处理文件时显示文件
- -z 将输出重定向给gzip命令来压缩内容

列出归档内容：tar -tf test.tar
提取归档内容：tar -xvf test.tar
创建归档文件：tar -cvf test.tar test/ test2


## 管理进程

### 探查进程
当程序运行在系统上时，我们称之为进程（process）。想监测这些进程，需要熟悉ps命令的用法。
使用ps命令的关键不在于记住所有可用的参数，而在于记住最有用的那些参数。

```
$ ps -ef
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 Aug21 ?        00:07:16 /usr/lib/systemd/systemd --system --deserialize 26
root         2     0  0 Aug21 ?        00:00:04 [kthreadd]
root         3     2  0 Aug21 ?        00:00:28 [ksoftirqd/0]
root         7     2  0 Aug21 ?        00:00:00 [migration/0]
```

常用参数：
- -e参数指定显示所有运行在系统上的进程；
- -f参数则扩展了输出，这些扩展的列包含了有用的信息。
- -l参数，它会产生一个长格式输出。

这些扩展的列包含了有用的信息：
- UID：启动这些进程的用户。
- PID：进程的进程ID。
- PPID：父进程的进程号（如果该进程是由另一个进程启动的）。
- C：进程生命周期中的CPU利用率。
- STIME：进程启动时的系统时间。
- TTY：进程启动时的终端设备。
- TIME：运行进程需要的累计CPU时间。
- CMD：启动的程序名称。

### 实时监测进程
想观察那些频繁换进换出的内存的进程趋势，top命令刚好适用这种情况。

```
top - 15:54:00 up 95 days, 10:46,  1 user,  load average: 0.14, 0.07, 0.06
Tasks: 176 total,   1 running, 175 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.3 us,  0.3 sy,  0.0 ni, 99.3 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem :   602816 total,    65452 free,   248696 used,   288668 buff/cache
KiB Swap:        0 total,        0 free,        0 used.   177264 avail Mem

  PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND
  170 root      20   0       0      0      0 S  0.3  0.0  20:03.70 xfsaild/sda1
14270 root      20   0  828800  24620   2880 S  0.3  4.1  19:15.48 dockerd
16454 systemd+  20   0  888224  33020      0 S  0.3  5.5   7:18.88 mysqld
31990 veining+  20   0  155620   2216   1500 R  0.3  0.4   0:00.04 top
    1 root      20   0  193496   4944   2392 S  0.0  0.8   7:25.59 systemd
    2 root      20   0       0      0      0 S  0.0  0.0   0:04.07 kthreadd
```

输出的第一部分显示的是系统的概况：
- 第一行显示了当前时间、系统的运行时间、登录的用户数以及系统的平均负载。
- 第二行显示了进程概要信息——top命令的输出中将进程叫作任务（task）：有多少进程处在运行、休眠、停止或是僵化状态（僵化状态是指进程完成了，但父进程没有响应）。
- 第三行显示了CPU的概要信息。 top根据进程的属主（用户还是系统）和进程的状态（运行、
空闲还是等待）将CPU利用率分成几类输出。
- 最后两行说明了系统内存的状态。第一行说的是系统的物理内存：总共有多少内存，当前用了多少，还有多少空闲。
后一行说的是同样的信息，不过是针对系统交换空间（如果分配了的话）的状态而言的。

最后一部分显示了当前运行中的进程的详细列表，有些列跟ps命令的输出类似：
- PID：进程的ID。
- USER：进程属主的名字。
- PR：进程的优先级。
- NI：进程的谦让度值。
- VIRT：进程占用的虚拟内存总量。
- RES：进程占用的物理内存总量。
- SHR：进程和其他进程共享的内存总量。
- S：进程的状态（D代表可中断的休眠状态， R代表在运行状态， S代表休眠状态， T代表跟踪状态或停止状态， Z代表僵化状态）。
- %CPU：进程使用的CPU时间比例。
- %MEM：进程使用的内存占可用内存的比例。
- TIME+：自进程启动到目前为止的CPU时间总量。
- COMMAND：进程所对应的命令行名称，也就是启动的程序名。

### 结束进程
在Linux中，进程之间通过信号来通信。进程的信号就是预定义好的一个消息，进程能识别它并决定忽略还是作出反应。进程如何处理信号是由开发人员通过编程来决定的。

在Linux上有两个命令可以向运行中的进程发出进程信号。
1. kill命令：kill 3940
kill命令可通过进程ID（PID）给进程发信号。默认情况下，kill命令会向命令行中列出的全部PID发送一个TERM信号。遗憾的是，你只能用进程的PID而不能用命令名，所以kill命令有时并不好用。
2. killall命令：killall http*
killall命令非常强大，它支持通过进程名而不是PID来结束进程。 killall命令也支持通配符，这在系统因负载过大而变得很慢时很有用。



## 管理磁盘
Linux文件系统将所有的磁盘都并入一个虚拟目录下。在使用新的存储媒体之前，需要把它放到虚拟目录下。这项工作称为挂载（mounting）。如果用的Linux发行版不支持自动挂载和卸载可移动存储媒体，就必须手动完成。

### mount命令
Linux上用来挂载媒体的命令叫作mount。默认情况下， mount命令会输出当前系统上挂载的设备列表。
```
$ mount
sysfs on /sys type sysfs (rw,nosuid,nodev,noexec,relatime,seclabel)
proc on /proc type proc (rw,nosuid,nodev,noexec,relatime)
devtmpfs on /dev type devtmpfs (rw,nosuid,seclabel,size=294632k,nr_inodes=73658,mode=755)
securityfs on /sys/kernel/security type securityfs (rw,nosuid,nodev,noexec,relatime)
```

mount命令提供如下四部分信息：
- 媒体的设备文件名
- 媒体挂载到虚拟目录的挂载点
- 文件系统类型
- 已挂载媒体的访问状态

手动挂载媒体设备的基本命令：
```
mount -t type device directory
```

### umount命令
从Linux系统上移除一个可移动设备时，不能直接从系统上移除，而应该先卸载。umount命令的格式非常简单：
```
umount [directory | device ]
```

### df 命令
有时你需要知道在某个设备上还有多少磁盘空间。df命令可以让你很方便地查看所有已挂载磁盘的使用情况
常用参数是-h，它会把输出中的磁盘空间按照用户易读的形式显示，通常用M来替代兆字节，用G替代吉字节。
```
$ df
Filesystem     1K-blocks    Used Available Use% Mounted on
/dev/sda1       10474496 8675760   1798736  83% /
devtmpfs          294632       0    294632   0% /dev
tmpfs             301408       0    301408   0% /dev/shm
```

df命令会显示每个有数据的已挂载文件系统。命令输出如下：
- 设备的设备文件位置；
- 能容纳多少个1024字节大小的块；
- 已用了多少个1024字节大小的块；
- 还有多少个1024字节大小的块可用；
- 已用空间所占的比例；
- 设备挂载到了哪个挂载点上。

### du 命令
du命令可以显示某个特定目录（默认情况下是当前目录）的磁盘使用情况。这一方法可用来快速判断系统上某个目录下是不是有超大文件。

能让du命令用起来更方便的几个命令行参数：
- -c：显示所有已列出文件总的大小。
- -h：按用户易读的格式输出大小，即用K替代千字节，用M替代兆字节，用G替代吉字节。
- -s：显示每个输出参数的总计。



## 理解Shell

### shell 的类型
系统启动什么样的shell程序取决于你个人的用户ID配置，在/etc/passwd文件中。
默认的交互shell会在用户登录某个虚拟控制台终端或在GUI中运行终端仿真器时启动。不过还有另外一个默认shell是/bin/sh，它作为默认的系统shell，用于那些需要在启动时使用的系统shell脚本。

### shell 的父子关系
用于登录某个虚拟控制器终端或在GUI中运行终端仿真器时所启动的默认的交互shell，是一个父shell。
在CLI提示符后输入/bin/bash命令或其他等效的bash命令时， 会创建一个新的shell程序。这个shell程序被称为子shell（child shell） 。

bash shell程序可使用命令行参数修改shell启动方式：
- -c string 从string中读取命令并进行处理
- -i 启动一个能够接收用户输入的交互shell
- -l 以登录shell的形式启动
- -r 启动一个受限shell，用户会被限制在默认目录中
- -s 从标准输入中读取命令

- 依次运行的一系列命令。这可以通过命令列表来实现，只需要在命令之间加入分号（;）即可：pwd ; ls ; cd /etc ; pwd ;
- 要想将命令置入后台模式，可以在命令末尾加上字符&：sleep 3000&
- 协程可以同时做两件事。它在后台生成一个子shell，并在这个子shell中执行命令。使用coproc命令进行协程处理：coproc sleep 10

### 理解 shell 的内建命令
#### 外部命令
外部命令，有时候也被称为文件系统命令，是存在于bash shell之外的程序。它们并不是shell程序的一部分。外部命令程序通常位于/bin、 /usr/bin、 /sbin或/usr/sbin中。
ps就是一个外部命令。你可以使用which和type命令找到它。
当外部命令执行时，会创建出一个子进程。这种操作被称为衍生（forking）。外部命令ps很方便显示出它的父进程以及自己所对应的衍生子进程。

#### 内建命令
内建命令和外部命令的区别在于前者不需要使用子进程来执行。它们已经和shell编译成了一体，作为shell工具的组成部分存在。不需要借助外部程序文件来运行。cd和exit命令都内建于bash shell。可以利用type命令来了解某个命令是否是内建的：
```
$ type ps
ps is hashed (/usr/bin/ps)
$
$ type -a echo
echo is a shell builtin
echo is /bin/echo
```

- 查看最近用过的命令列表：history
- 为常用的命令（及其参数）创建另一个名称：alias
- 要查看当前可用的别名，使用alias命令以及选项-p：alias -p



## 环境变量
Linux环境变量能帮你提升Linux shell体验。很多程序和脚本都通过环境变量来获取系统信息、存储临时数据和配置信息。在Linux系统上有很多地方可以设置环境变量，了解去哪里设置相应的环境变量很重要。
在涉及环境变量名时，如果要用到变量，使用$；如果要操作变量，不使用$。

在bash shell中，环境变量分为两类：
### 全局变量
全局环境变量对于shell会话和所有生成的子shell都是可见的。局部变量则只对创建它们的
shell可见。这让全局环境变量对那些所创建的子shell需要获取父shell信息的程序来说非常有用。
要查看全局变量，可以使用 `env` 或 `printenv` 命令。

要显示个别环境变量的值，可以使用 `printenv` 命令：
```
$ printenv HOME
/home/veininguo
```

也可以使用 `echo` 显示变量的值。在变量前面加上一个美元符（$）:
```
echo $HOME
/home/veininguo
```

### 局部变量
局部环境变量只能在定义它们的进程中可见。
在Linux系统并没有一个只显示局部环境变量的命令。 set命令会显示为某个特定进程设置的所有环境变量，包括局部变量、全局变量以及用户定义变量。

### 设置用户定义变量
可以在bash shell中直接设置自己的变量。

#### 设置局部用户定义变量
创建在shell进程内可见的局部变量，但在子shell中无法使用用户定义变量。可以通过等号给环境变量赋值，值可以是数值或字符串。
所有的环境变量名均使用大写字母，如果是你自己创建的局部变量或是shell脚本，请使用小写字母。
变量名、等号和值之间没有空格，这一点非常重要。
```
$ my_variable="Veinin Guo"
$ echo $my_variable
Veinin Guo
```

#### 设置全局环境变量
在设定全局环境变量的进程所创建的子进程中，该变量都是可见的。
创建全局环境变量的方法是先创建一个局部环境变量，然后再把它导出到全局环境中。通过export命令来完成，变量名前面不需要加$。
子shell无法使用export命令改变父shell中全局环境变量的值。
```
$ my_variable="I am Global now"
$ export my_variable
$
$ echo $my_variable
I am Global now
$
$ bash
$ echo $my_variable
I am Global now
$
$ exit
exit
$ echo $my_variable
I am Global now
```

#### 删除环境变量
可以用 `unset` 命令完成这个操作。在命令中引用环境变量时不要使用$。
```
$ echo $my_variable
I am Global now
$ unset my_variable
$ echo $my_variable

```

### 设置 PATH 环境变量
在 shell 命令行界面中输入一个外部命令时， shell 必须搜索系统来找到对应的程序。PATH 环境变量定义了用于进行命令和程序查找的目录。PATH中的目录使用冒号分隔。

```
$ echo $PATH
/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/home/veininguo/.local/bin:/home/veininguo/bin
```

把新的搜索目录添加到现有的PATH环境变量中，无需从头定义。PATH中各个目录之间是用冒号分隔的。你只需引用原来的PATH值，然后再给这个字符串添加新目录就行了。也将单点符也加入PATH环境变量，单点符代表当前目录。

```
$ echo $PATH
/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/home/veininguo/.local/bin:/home/veininguo/bin
$
$ PATH=$PATH:/usr/games:/usr/local/games
$
$ echo $PATH
/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/home/veininguo/.local/bin:/home/veininguo/bin:/usr/games:/usr/local/games
$
$ PATH=$PATH:.
```

### 定位系统环境变量
在你登入Linux系统启动一个bash shell时，默认情况下bash会在几个文件中查找命令。这些文件叫作启动文件或环境文件。
在大多数发行版中，存储个人用户永久性bash shell变量的地方是$HOME/.bashrc文件。 

启动bash shell有3种方式：
- 登录时作为默认登录shell，登录shell会从5个不同的启动文件里读取命令：
1. /etc/profile，是bash shell默认的的主启动文件。只要你登录了Linux系统， bash就会执行/etc/profile启动文件中的命令。
2. $HOME目录下的启动文件，剩下的启动文件都起着同一个作用：提供一个用户专属的启动文件来定义该用户所用到的环
境变量。大多数Linux发行版只用这四个启动文件中的一到两个：
$HOME/.bash_profile
$HOME/.bashrc
$HOME/.bash_login
$HOME/.profile

- 作为非登录shell的交互式shell
如果你的bash shell不是登录系统时启动的（比如是在命令行提示符下敲入bash时启动），那么你启动的shell叫作交互式shell。
交互式shell启动时，只会检查用户HOME目录中的.bashrc文件。

- 作为运行脚本的非交互shell
系统执行shell脚本时用的就是这种shell。不同的地方在于它没有命令行提示符。
但是当你在系统上运行脚本时，也许希望能够运行一些特定启动的命令，为了处理这种情况， bash shell提供了BASH_ENV环境变量。

### 数组变量
要给某个环境变量设置多个值，可以把值放在括号里，值与值之间用空格分隔。
```
$ mytest=(one two three four five)
```

引用一个单独的数组元素，就必须用代表它在数组中位置的数值索引值。索引值要用方括号括起来。
```
$ echo ${mytest[2]}
three
```



## 文件权限
缺乏安全性的系统不是完整的系统。系统中必须有一套能够保护文件免遭非授权用户浏览或修改的机制。 Linux沿用了Unix文件权限的办法，即允许用户和组根据每个文件和目录的安全性设置来访问文件。

### Linux 的安全性
Linux安全系统的核心是用户账户。每个能进入Linux系统的用户都会被分配唯一的用户账户。用户对系统中各种对象的访问权限取决于他们登录系统时用的账户。用户权限是通过创建用户时分配的用户ID（User ID，通常缩写为UID）来跟踪的。UID是数值，每个用户都有唯一的UID，但在登录系统时用的不是UID，而是登录名。

#### /etc/passwd 文件
这个文件将用户的登录名匹配到对应的UID值，它包含了一些与用户有关的信息。

```
$ cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
...
veininguo:x:1000:1001::/home/veininguo:/bin/bash
dockerroot:x:996:993:Docker User:/var/lib/docker:/sbin/nologin
```

/etc/passwd文件的字段包含了如下信息：
- 登录用户名
- 用户密码
- 用户账户的UID（数字形式）
- 用户账户的组ID（GID）（数字形式）
- 用户账户的文本描述（称为备注字段）
- 用户HOME目录的位置
- 用户的默认shell

#### /etc/shadow 文件
/etc/shadow文件对Linux系统密码管理提供了更多的控制。只有root用户才能访问/etc/shadow文件，这让它比起/etc/passwd安全许多。
etc/shadow文件为系统上的每个用户账户都保存了一条记录。记录就像下面这样：
```
veininguo:*:17399:0:99999:7:::
```

每条记录中都有9个字段：
- 与/etc/passwd文件中的登录名字段对应的登录名
- 加密后的密码
- 自上次修改密码后过去的天数密码（自1970年1月1日开始计算）
- 多少天后才能更改密码
- 多少天后必须更改密码
- 密码过期前提前多少天提醒用户更改密码
- 密码过期后多少天禁用用户账户
- 用户账户被禁用的日期（用自1970年1月1日到当天的天数表示）
- 预留字段给将来使用

#### 添加新用户
用来向Linux系统添加新用户的主要工具是useradd。系统默认值被设置在/etc/default/useradd文件中。

在创建新用户时，如果你不在命令行中指定具体的值， useradd命令就会使用-D选项所显示的那些默认值。这个例子列出的默认值如下：
- 新用户会被添加到GID为100的公共组；
- 新用户的HOME目录将会位于/home/loginname；
- 新用户账户密码在过期后不会被禁用；
- 新用户账户未被设置过期日期；
- 新用户账户将bash shell作为默认shell；
- 系统会将/etc/skel目录下的内容复制到用户的HOME目录下；
- 系统为该用户账户在mail目录下创建一个用于接收邮件的文件。

默认情况下，useradd命令不会创建HOME目录，但是-m命令行选项会使其创建HOME目录。

```
$ sudo useradd -m test
$ sudo ls -al /home/test/
total 12
drwx------. 2 test test  62 Nov 26 13:18 .
drwxr-xr-x. 4 root root  35 Nov 26 13:18 ..
-rw-r--r--. 1 test test  18 Sep  6 16:25 .bash_logout
-rw-r--r--. 1 test test 193 Sep  6 16:25 .bash_profile
-rw-r--r--. 1 test test 231 Sep  6 16:25 .bashrc
```

#### 删除用户
userdel可以从系统中删除用户， 加上-r参数， userdel会删除用户的HOME目录以及邮件目录。

```
$sudo userdel -r test
$
$ sudo ls -al /home/test/
ls: cannot access /home/test/: No such file or directory
```

#### 修改用户
Linux提供了一些不同的工具来修改已有用户账户的信息：
1. usermod，修改用户账户的字段，还可以指定主要组以及附加组的所属关系。常用参数如下：
-c修改备注字段。
-e修改过期日期。
-g修改默认的登录组。
-l修改用户账户的登录名。
-L锁定账户，使用户无法登录。
-p修改账户的密码。
-U解除锁定，使用户能够登录。

2. passwd和chpasswd
改变用户密码的一个简便方法就是用passwd命令。
如果需要为系统中的大量用户修改密码， chpasswd命令可以事半功倍。 

3. chsh、 chfn和chage
chsh、 chfn和chage工具专门用来修改特定的账户信息。
chsh命令用来快速修改默认的用户登录shell。
chfn命令提供了在/etc/passwd文件的备注字段中存储信息的标准方法。 
chage命令用来帮助管理用户账户的有效期。


### Linux 组
组权限允许多个用户对系统中的对象（比如文件、目录或设备等）共享一组共用的权限。
每个组都有唯一的GID——跟UID类似，在系统上这是个唯一的数值。

#### /etc/group 文件
/etc/group文件包含系统上用到的每个组的信息。

```
$ cat /etc/group
root:x:0:
bin:x:1:
daemon:x:2:
...
```

/etc/group文件有4个字段：
- 组名
- 组密码
- ID
- 属于该组的用户列表

#### 创建新组
groupadd命令可在系统上创建新组。
```
$ sudo groupadd shared
$ tail /etc/group
docker:x:992:
shared:x:1002:
```

usermod命令将用户被分配到该组。
```
$ sudo usermod -G shared test
$
$ tail /etc/group
shared:x:1002:test
test:x:1003:
```

#### 修改组
groupmod命令可以修改已有组的GID（加-g选项）或组名（加-n选项）。
```
$ sudo groupmod -n sharering shared
$ tail /etc/group
google-sudoers:x:1000:veininguo
test:x:1003:
sharering:x:1002:test
```


### 文件权限

#### 文件权限符
ls命令可以用来查看Linux系统上的文件、目录和设备的权限。
```
$ ls -l
total 1693436
drwxrwxr-x. 2 veininguo veininguo         6 Nov 21 16:08 data_file
```

输出结果的第一个字段就是描述文件和目录权限的编码。
第一个字符代表了对象的类型：
- -代表文件
- d代表目录
- l代表链接
- c代表字符型设备
- b代表块设备
- n代表网络设备

之后有3组三字符的编码。每一组定义了3种访问权限：
- r代表对象是可读的
- w代表对象是可写的
- x代表对象是可执行的

若没有某种权限，在该权限位会出现单破折线。这3组权限分别对应对象的3个安全级别：
- 对象的属主
- 对象的属组
- 系统其他用户

#### 改变权限
chmod命令用来改变文件和目录的安全性设置：`chmod options mode file`

chmod命令采用了另一种方法。下面是在符号模式下指定权限的格式：`[ugoa…][[+-=][rwxXstugo…]`
第一组字符定义了权限作用的对象：
- u代表用户
- g代表组
- o代表其他
- a代表上述所有

现有权限基础上增加权限（+），还是在现有权限基础上移除权限（-），或是将权限设置成后面的值（=）
```
$ chmod o+r newfile
$ ls -lF newfile
-rwxrw-r-- 1 rich rich 0 Sep 20 19:16 newfile
$
$ chmod u-x newfile
$ ls -lF newfile
-rw-rw-r-- 1 rich rich 0 Sep 20 19:16 newfile
```

#### 改变所属关系
chown命令用来改变文件的属主。
```
$ chown dan newfile
$ ls -l newfile
-rw-rw-r-- 1 dan rich 0 Sep 20 19:16 newfile
$
$ chown dan.shared newfile
$ ls -l newfile
-rw-rw-r-- 1 dan shared 0 Sep 20 19:16 newfile
```

chgrp命令可以更改文件或目录的默认属组。
```
$ chgrp shared newfile
$ ls -l newfile
-rw-rw-r-- 1 rich shared 0 Sep 20 19:16 newfile
```



## 文件系统
使用Linux系统时，需要作出的决策之一就是为存储设备选用什么文件系统。大多数Linux发行版在安装时会非常贴心地提供默认的文件系统，了解一下可用的选择有时也会有所帮助。

### Linux 文件系统
1. ext文件系统，最早的文件系统，叫作扩展文件系统 （extended filesystem，简记为ext）。
2. ext2文件系统，ext文件系统有不少限制，比如文件大小不得超过2 GB。ext文件系统就升级到了第二代扩展文件系统，叫作ext2。

### 日志文件系统
日志文件系统为Linux系统增加了一层安全性。它不再使用之前先将数据直接写入存储设备再更新索引节点表的做法，而是先将文件的更改写入到临时文件（称作日志， journal）中。在数据成功写到存储设备和索引节点表之后，再删除对应的日志条目。

1. ext3文件系统，它采用和ext2文件系统相同的索引节点表结构，但给每个存储设备增加了一个日志文件，以将准备写入存储设备的数据先记入日志。
2. ext4文件系统，扩展ext3文件系统功能的结果是ext4文件系统。
3. Reiser文件系统。
4. JFS文件系统，最老的日志文件系统之一， 是IBM在1990年为其Unix衍生版AIX开发的。
5. XFS文件系统，美国硅图公司（SGI）最初在1994年为其商业化的IRIX Unix系统开发的。

### 写时复制文件系统
写时复制（copy-on-write， COW）的技术。COW利用快照兼顾了安全性和性能。如果要修改数据，会使用克隆或可写快照。修改过的数据
并不会直接覆盖当前数据，而是被放入文件系统中的另一个位置上。即便是数据修改已经完成，之前的旧数据也不会被重写。

1. ZFS文件系统，没有使用GPL许可。OpenZFS项目有可能改变这种局面。
2. Btrf文件系统，被称为B树文件系统。它是由Oracle公司于2007年开始研发的。 

### 操作文件系统
创建分区：fdisk工具用来帮助管理安装在系统上的任何存储设备上的分区。
创建文件系统：每个文件系统命令都有很多命令行选项，允许你定制如何在分区上创建文件系统。

### 逻辑卷管理
如果硬盘上没有地方了，你就必须弄一个更大的硬盘，然后手动将已有的文件系统移动到新的硬盘上。Linux逻辑卷管理器（logical volume manager， LVM）软件包可以用来做动态地添加存储空间。




## 安装软件程序
Linux开发人员通过把软件打包成更易于安装的预编译包，在Linux上能见到的各种包管理系统（package management system， PMS），以及用来进行软件安装、管理和删除的命令行工具。

### 基于 Debian 的系统
dpkg命令是基于Debian系PMS工具的核心。包含在这个PMS中的其他工具有：
- apt-get
- apt-cache
- aptitude

### 基于 Red Hat 的系统
和基于Debian的发行版类似，基于Red Hat的系统也有几种不同的可用前端工具。常见的有以下3种。
- yum：在Red Hat和Fedora中使用。
- urpm：在Mandriva中使用。
- zypper：在openSUSE中使用。