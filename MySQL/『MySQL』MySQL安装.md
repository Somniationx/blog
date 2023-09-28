---
title: 『MySQL』MySQL安装
date: 2023-9-18 17:25:00
updated: 2023-9-18 17:25:00
categories:
  - MySQL
post_copyright:
copyright_author: Somniationx
# copyright_author_href: https://xxxxxx.com
# copyright_url: https://xxxxxx.com
copyright_info: 此文章版权归Somniationx所有，如有转载，请註明来自原作者
---
# 前置说明

> - 此教程面向**`Centos7`**版本操作系统。
> - 卸载安装均在`root`用户下进行，其他用户均能使用。
> - 前期教学在`roo`t用户下，熟悉相关操作等，后续切换至普通用户。

# 卸载内置环境

> **若读者环境配置完善可跳过此教程。**

## 检测`mriadb`与`mysql`进程

```bash
[root@VM-16-4-centos ~]# ps axj | grep mariadb | grep -v grep
[root@VM-16-4-centos ~]# ps axj | grep mysql | grep -v grep
[root@VM-16-4-centos ~]# 
```

> 没有显示任何输出，`grep`命令都没有找到匹配的进程。意味着系统中没有运行中的与`mariadb`或`mysql`相关的进程。

**若读者的系统中有其中服务，可执行下列指令停止相关服务：**

```bash
systemctl stop mariadb.service # 停⽌mariadb 服务
systemctl stop mysqld # 停⽌mysql 服务
```

命令使用`systemctl`工具，该工具用于管理系统的Systemd服务。**请注意，MariaDB和MySQL是两种不同的数据库管理系统，因此你需要根据实际安装在系统上的数据库服务来选择停止的命令。**

- `systemctl stop mariadb.service`：这个命令用于停止MariaDB服务。MariaDB是一个MySQL分支，所以它的服务通常被命名为`mariadb.service`。
- `systemctl stop mysqld`：这个命令用于停止MySQL服务。MySQL的服务通常被命名为`mysqld`或类似的名称。

## 检查系统安装包

```bash
[root@VM-16-4-centos ~]# rpm -qa | grep mariadb
mariadb-libs-5.5.68-1.el7.x86_64
[root@VM-16-4-centos ~]# rpm -qa | grep mysql
[root@VM-16-4-centos ~]# 
```

1. `rpm -qa | grep mariadb`：这个命令使用`rpm`工具查询已安装的软件包，并通过管道将结果传递给`grep`命令，以查找包含字符串`mariadb`的软件包。根据输出，系统已安装了一个名为`mariadb-libs-5.5.68-1.el7.x86_64`的软件包，这是`Mariadb`数据库的库文件。
2. `rpm -qa | grep mysql`：输出中，没有找到任何匹配项，这意味着系统中没有安装任何名为`mysql`的软件包。

## 卸载这些默认安装包

```bash
[root@VM-16-4-centos ~]# rpm -qa | grep mariadb | xargs yum -y remove
[root@VM-16-4-centos ~]# rpm -qa | grep mmysql | xargs yum -y remove
```

这个命令使用一系列Linux命令来卸载（删除）与"Mariadb"相关的软件包。**（mysql相关指令同理）**

1. `rpm -qa | grep mariadb`：这部分命令使用`rpm`工具来查询已安装的软件包列表（`rpm -qa`），然后通过管道(`|`)将结果传递给`grep`命令。`grep`命令用于筛选包含字符串`mariadb`的软件包。这将列出与Mariadb相关的所有已安装软件包。

2. `xargs`：`xargs`命令用于将前一步的输出（即已安装的Mariadb相关软件包的列表）作为参数传递给下一个命令。

3. `yum -y remove`：这部分命令使用`yum`工具来卸载（删除）指定的软件包。`-y`选项表示在卸载时**不需要用户确认**，直接进行卸载操作。`remove`命令后面需要指定要卸载的软件包。

所以，这个命令的整体目的是列出并卸载所有与Mariadb相关的已安装软件包。通过使用`rpm`和`grep`命令，它列出与Mariadb相关的软件包，然后使用`xargs`将它们传递给`yum -y remove`来进行卸载操作。这将自动删除与Mariadb相关的所有软件包而不需要用户手动确认。

# 获取`mysql`官⽅`yum`源

> 获取mysql官⽅yum源: http://repo.mysql.com/
>
> 注意：最好安装与系统⼀致的mysql版本，否则可能会存在软件兼容性问题。

## 查看系统版本

```bash
[root@VM-16-4-centos ~]# cat /etc/redhat-release 
CentOS Linux release 7.6.1810 (Core) 
```

**获取系统版本之后在官方源上下载对应版本上传至Linux服务器。**

> **笔者下载版本：`mysql57-community-release-el7.rpm`**

```bash
[root@VM-16-4-centos MySQL]# ls
mysql57-community-release-el7.rpm
```

`mysql57-community-release-el7.rpm`是MySQL 5.7社区版的安装仓库文件（Repository RPM），用于在CentOS 7及其衍生版本上安装MySQL 5.7社区版。这个Repository RPM包含了MySQL 5.7数据库服务器的安装源配置信息，使你可以使用`yum`或`dnf`等包管理工具轻松安装MySQL 5.7。

## 安装`mysql` `yum` 源

```bash
# 查看本地的yum源
[root@VM-16-4-centos ~]# ls /etc/yum.repos.d/ -al
total 32
drwxr-xr-x.  2 root root  4096 Sep 26 07:51 .
drwxr-xr-x. 93 root root 12288 Sep 27 23:53 ..
-rw-r--r--   1 root root   614 Sep 25 22:12 CentOS-Base.repo
-rw-r--r--   1 root root   230 Sep 25 22:12 CentOS-Epel.repo
-rw-r--r--   1 root root  1358 Sep  5  2021 epel.repo
-rw-r--r--   1 root root  1457 Sep  5  2021 epel-testing.repo
[root@VM-16-4-centos MySQL]cd MySQL
# 安装
[root@VM-16-4-centos MySQL]# rpm -ivh mysql57-community-release-el7.rpm
warning: mysql57-community-release-el7.rpm: Header V3 DSA/SHA1 Signature, key ID 5072e1f5: NOKEY
Preparing...                          ################################# [100%]
Updating / installing...
   1:mysql57-community-release-el7-11 ################################# [100%]
# 安装后再次查看
[root@VM-16-4-centos MySQL]# ls /etc/yum.repos.d/ -al
total 40
drwxr-xr-x.  2 root root  4096 Sep 28 00:17 .
drwxr-xr-x. 93 root root 12288 Sep 27 23:53 ..
-rw-r--r--   1 root root   614 Sep 25 22:12 CentOS-Base.repo
-rw-r--r--   1 root root   230 Sep 25 22:12 CentOS-Epel.repo
-rw-r--r--   1 root root  1358 Sep  5  2021 epel.repo
-rw-r--r--   1 root root  1457 Sep  5  2021 epel-testing.repo
-rw-r--r--   1 root root  1838 Apr 27  2017 mysql-community.repo
-rw-r--r--   1 root root  1885 Apr 27  2017 mysql-community-source.repo
```

1. `ls /etc/yum.repos.d/ -al`：这个命令用于列出`/etc/yum.repos.d/`目录中的所有文件和子目录的详细信息，包括权限、所有者、文件大小等。这个命令的目的是查看当前系统中已有的yum仓库配置文件。

2. `rpm -ivh mysql57-community-release-el7.rpm`：这个命令使用`rpm`工具来安装名为`mysql57-community-release-el7.rpm`的MySQL 5.7的yum仓库配置文件。具体来说：
   
   - `-i`选项表示安装软件包。
   - `-v`选项表示在执行安装时显示详细信息。
   - `-h`选项表示以哈希标记的形式显示安装进度。
   - `mysql57-community-release-el7.rpm`是要安装的Repository RPM文件的名称。
   
   安装Repository RPM文件后，系统将获得与MySQL 5.7相关的软件包信息，可以使用`yum`命令来安装MySQL 5.7。
   
3. 最后，`ls /etc/yum.repos.d/ -al`命令再次执行，以确认MySQL 5.7的yum仓库配置文件已成功添加到`/etc/yum.repos.d/`目录中。从输出中可以看到，两个新的Repository RPM文件`mysql-community.repo`和`mysql-community-source.repo`已经存在，这表示MySQL 5.7的yum仓库配置已经添加到系统中。

> 目的是准备系统以安装MySQL 5.7，包括添加MySQL 5.7的yum仓库配置文件以及确认它们已经成功添加到系统中。接下来，可以使用`yum`命令来安装MySQL 5.7或其他MySQL相关的软件包。

## 检测能否正常工作

```bash
[root@VM-16-4-centos MySQL]# yum list | grep mysql
Repository epel is listed more than once in the configuration
mysql57-community-release.noarch         el7-11                        installed
akonadi-mysql.x86_64                     1.9.2-4.el7                   os       
anope-mysql.x86_64                       2.0.14-1.el7                  epel     
apr-util-mysql.x86_64                    1.5.2-6.el7_9.1               updates  
calligra-kexi-driver-mysql.x86_64        2.9.10-2.el7                  epel     
collectd-mysql.x86_64                    5.8.1-1.el7                   epel     
dmlite-plugins-mysql.x86_64              1.15.2-15.el7                 epel     
dovecot-mysql.x86_64                     1:2.2.36-8.el7                os       
dpm-copy-server-mysql.x86_64             1.13.0-1.el7                  epel     
dpm-name-server-mysql.x86_64             1.13.0-1.el7                  epel     
dpm-server-mysql.x86_64                  1.13.0-1.el7                  epel     
dpm-srm-server-mysql.x86_64              1.13.0-1.el7                  epel     
dspam-mysql.x86_64                       3.10.2-12.el7                 epel     
...
# 能看到上列信息即可
```

# 安装`mysql`服务

```bash
[root@VM-16-4-centos MySQL]# yum install -y mysql-community-server
# 若结尾出现报错
Public key for mysql-community-server-5.7.43-1.el7.x86_64.rpm is not installed
 Failing package is: mysql-community-server-5.7.43-1.el7.x86_64
 GPG Keys are configured as: file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql
 # 解决方案
 rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2022
 # 终端输入并再次执行安装指令即可
 # 安装成功
 Installed:
  mysql-community-server.x86_64 0:5.7.43-1.el7                                                                     
Dependency Installed:
  mysql-community-client.x86_64 0:5.7.43-1.el7   mysql-community-common.x86_64 0:5.7.43-1.el7   mysql-community-libs.x86_64 0:5.7.43-1.el7  

Complete!
```

当尝试使用 `yum` 包管理器安装 MySQL Community Server 时，系统会尝试验证软件包的**完整性和安全性**，以确保它来自受信任的源。这个验证过程包括检查数字签名，以确保软件包没有被篡改或损坏。如果系统不具备安装软件包所需的 **GPG（GNU Privacy Guard）**公钥，就会出现公钥错误。

以下是在安装 MySQL Community Server 时遇到的问题及解决方法的详细解释：

1. **问题：** 在尝试安装 MySQL Community Server 时，出现了错误消息，指示缺少 GPG 公钥。这是因为系统无法验证 MySQL 软件包的真实性，因此不允许安装。

2. **解决方案：** 为了解决这个问题，需要获取并导入 MySQL 的 GPG 公钥，**以便系统能够验证 MySQL 软件包的完整性和来源**。您使用了以下命令：

   ```bash
   rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2022
   ```

   - `rpm --import`：这是一个 RPM 包管理器的命令，用于导入 GPG 密钥。
   - `https://repo.mysql.com/RPM-GPG-KEY-mysql-2022`：这是 MySQL 2022 年的 GPG 密钥的 URL，它包含了用于验证 MySQL 软件包的数字签名。

3. 导入了 GPG 密钥后，您再次运行 `yum install -y mysql-community-server` 命令，这次成功安装了 MySQL Community Server。

通过导入 GPG 密钥，告诉系统要信任来自 MySQL 官方仓库的软件包，并确保它们没有被篡改或损坏。这是一种常见的安全措施，用于确保您下载和安装的软件包是可信的。

如果将来遇到类似的 GPG 错误，可以考虑使用相同的方法导入适当的 GPG 密钥，以解决问题并安全地安装软件包。这有助于保护系统免受恶意软件包的威胁。

## 启动服务

```bash
[root@VM-16-4-centos MySQL]# systemctl start mysqld
[root@VM-16-4-centos MySQL]# ps axj | grep mysqld | grep -v grep
    1 11203 11202 11202 ?           -1 Sl      27   0:00 /usr/sbin/mysqld --daemonize --pid-file=/var/run/mysqld/mysqld.pid
[root@VM-16-4-centos MySQL]# netstat -nltp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      1378/sshd           
tcp6       0      0 :::3306                 :::*                    LISTEN      11203/mysqld        
tcp6       0      0 :::22                   :::*                    LISTEN      1378/sshd  
```

1. 使用 `systemctl start mysqld` 命令成功启动了 **MySQL 服务器**。这是启动 MySQL 服务器的**标准命令**。
2. 使用 `netstat -nltp` 命令，您列出了所有当前**正在侦听网络连接的服务器程序**。结果显示 MySQL 服务器正在侦听端口 3306，这是 MySQL 默认的端口。此端口用于与 MySQL 数据库建立连接。

## 登录

> 登陆方式有多种，此处笔者按照习惯推荐。

```bash
# 打开mysql配置⽂件
[root@VM-16-4-centos MySQL]# vim /etc/my.cnf
# 在最后⼀栏配置加⼊:skip-grant-tables选项,保存退出即可
[root@VM-16-4-centos MySQL]# mysql
ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: NO)
# 报错是因为此处需要重启服务
[root@VM-16-4-centos MySQL]# systemctl restart mysqld
[root@VM-16-4-centos MySQL]# mysql
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.7.43 MySQL Community Server (GPL)

Copyright (c) 2000, 2023, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> 
# 到此为登陆成功！
mysql> quit
Bye
# 输入quit按下回车即可退出
```

> `skip-grant-tables` 选项是 MySQL 的一个配置选项，它的作用是在启动 MySQL 数据库时跳过用户权限验证**（跳过权限表）**，允许用户在不提供有效凭据**（用户名和密码）**的情况下登录到 MySQL 数据库，会使 MySQL 数据库变得非常不安全。只有在必要时才应使用它，而且在使用完毕后应立即将其从 MySQL 配置中删除，并重新启用权限验证，以确保数据库的安全性。在生产环境中，应谨慎使用此选项。
>
> 此处只用来练习数据库，后续会出其他登陆方式教程。

## 设置开机启动（可选）

```bash
[root@VM-16-4-centos ~]# systemctl enable mysqld
[root@VM-16-4-centos ~]# systemctl daemon-reload
[root@VM-16-4-centos ~]# 
```

1. `systemctl enable mysqld`：此命令会使 MySQL 服务在系统启动时自动启动。这意味着无需手动启动 MySQL 服务，它将在系统启动时自动运行。
2. `systemctl daemon-reload`：此命令重新加载 systemd 系统管理守护程序的配置。通常，当您修改了 systemd 服务单元的配置文件时，需要运行此命令，以确保 systemd 重新读取并使用新的配置。

> 确保 MySQL 在系统启动时自动启动，并确保 systemd 使用最新的配置信息。

## 配置`my.cnf`

```bash
[root@VM-16-4-centos ~]# vim /etc/my.cnf
# 加入下面三行配置
+ port=3306
+ datadir=/var/lib/mysql
+ socket=/var/lib/mysql/mysql.sock

datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock

# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0

log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid
skip-grant-tables
# 配置好之后重新启动服务即可
[root@VM-16-4-centos ~]# systemctl restart mysqld
[root@VM-16-4-centos ~]# 
```

1. `port=3306`：此选项指定 **MySQL 服务器监听的端口号**。默认情况下，MySQL 使用端口 3306 进行通信，因此这一行表示 MySQL 将在 3306 端口上侦听连接请求。
2. `datadir=/var/lib/mysql`：这是 MySQL 数据目录的路径。MySQL 将存储数据库文件和相关数据在此目录中。
3. `socket=/var/lib/mysql/mysql.sock`：MySQL 服务器套接字文件的路径。套接字文件用于本地连接到 MySQL 服务器。这里指定了 MySQL 套接字的位置。