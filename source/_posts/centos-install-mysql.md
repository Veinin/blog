---
title: Centos 从官方 Yum Repositories 安装 MySQL
date: 2017-11-08 21:55:49
tags: [CentOS, MySQL]
---

## 介绍
2013年10月，MySQL开发团队正式启动了对 Yum 版本库的支持。这意味着您现在可以确保您拥有从源代码直接安装的最新版本的 MySQL！
本文将在全新的CentOS 6中安装 MySQL 5.7。5.7是当前默认最新版本，当然你也可以选择直接安装其他版本，可参考本文关于 [选择指定版本安装](http://localhost:8080/2017/11/08/centos-install-mysql/#选择指定版本安装)章节。

<!--more-->

## 安装
Yum 版本库文件需要从 MYSQL 开发者网站下载，安装完成后，使用 `yum update` 来确保你运行的是最新的MySQL版本，包括其安全更新，Yum 会帮你解决任何依赖关系，这使你安装过程变成很简单。

刚开始，你需要访问 [MySQL Repositories](https://dev.mysql.com/downloads/repo/) ，选择 [MySQL Yum Repository](https://dev.mysql.com/downloads/repo/yum/)，根据你 CentOS 的版本，选择相应的下载链接，然后点击 Download。
![](/images/02_centos_install_mysql_download.jpg)

右键 "No thanks, just start my download." ， 点击 “复制链接地址”。
![](/images/03_centos_install_mysql_link.jpg)

登录到您的服务器，根据复制的连接地址然后下载此文件。以下只是一个示例网址 - 你可能需要检查这个地址是否正确：
```
wget https://dev.mysql.com/get/mysql57-community-release-el6-11.noarch.rpm
```

从本地文件中安装下载好的 rpm 文件:
```
sudo yum localinstall mysql57-community-release-el6-11.noarch.rpm
```

您现在已经在服务器上从官方存储库安装了，但尚未安装任何软件。存储库包括MySQL服务器，MySQL Workbench管理工具和ODBC驱动程序。让我们安装MySQL服务器：
```
sudo yum install mysql-community-server
```

启动 MySQL 服务：
```
sudo service mysqld start
```

配置MySQL在重新启动时自动启动：
```
sudo chkconfig mysqld on
chkconfig --list mysqld
```

到此，你的MySQL服务已经安装完成。

----

## 重置 Root 密码
如果你安装的是 MySQL 5.7 版本，root 密码会自动生成，并在启动日志 /var/log/mysqld.log 里面输出。

获取初始密码：
```
grep "password" /var/log/mysqld.log
2017-11-08T05:09:31.492454Z 1 [Note] A temporary password is generated for root@localhost: 1!Ta0EEis-yR
```

```
登录 root 账号，如果默认密码为空，请去除 -p 参数：
mysql -uroot -p '1!Ta0EEis-yR'
```

用你的新密码替换字符串 " your_new_password "，如果你的版本是 5.6 或之前版本，请使用一下命令:
```
UPDATE mysql.user SET Password=PASSWORD('your_new_password') WHERE User='root';
flush privileges;
exit
```

如果你的版本是 5.7 版本，请使用以下命令：
```
ALTER USER USER() IDENTIFIED BY ' your_new_password ';
```

如果提示：ERROR 1819 (HY000): Your password does not satisfy the current policy requirements，那是因为默认开启了简单密码检查政策，你可以通过以下命令关闭其功能：
```
set global validate_password_policy=0;
```

----

## 选择指定版本安装
使用 MySQL Yum 存储库时，默认情况下选择最新的 GA 版本（当前为 MySQL 5.7）进行安装。

在 MySQL Yum 存储库中，MySQL社区服务器的不同版本托管在不同的子库中。最新的 GA 系列（目前是 MySQL 5.7）的子库是默认启用的，所有其他系列的子库（例如MySQL 5.6版本）默认是禁用的。可以使用以下命令查看 MySQL Yum 存储库中的所有子存储库，并查看其中哪些被启用或禁用：
```
yum repolist all | grep mysql
```

如果要安装最新的版本，不需要配置。如果要选择特定版本，请在运行安装命令之前，禁用最新 GA 版本的子库，并启用特定版本的子库。如果你的平台支持 yum-config-manager，你可以通过以下这些命令来实现，这些命令将禁用 5.7 版本的 subrepository，并启用5.6 版本的 subrepository：
```
sudo yum-config-manager --disable mysql57-community
sudo yum-config-manager --enable mysql56-community
```

除了使用 yum-config-manager 命令之外，还可以通过手动编辑 /etc/yum.repos.d/mysql-community.repo 文件来选择发行版，项目是一个典型的文件系列子库的条目信息：
```
[mysql57-community]
name=MySQL 5.7 Community Server
baseurl=http://repo.mysql.com/yum/mysql-5.7-community/el/6/$basearch/
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql
```

找到要配置的子库的条目，然后编辑该 enabled 选项。指定 enabled=0 禁用子库，或 enabled=1 启用子库。例如，要安装MySQL 5.6，请确保您拥有 enabled=0 MySQL 5.7版本的上述子目录条目，并且具有 enabled=1 MySQL 5.6版本的条目：
```
# Enable to use MySQL 5.6
[mysql56-community]
name=MySQL 5.6 Community Server
baseurl=http://repo.mysql.com/yum/mysql-5.6-community/el/6/$basearch/
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql
```

最后再运行安装命令（mysql-community-server），那么你安装到的是你想要的指定版本的 MySQL。