---
layout: post
title: 【Linux 开发者学习篇】Linux 简介
category: Linux&Docker
tags: [Linux&Docker]
keywords: linux,docker
excerpt: Linux 简介
---

# 【Linux 开发者学习篇】Linux 简介

## Linux 概述

Linux是一套免费使用和自由传播的操作系统内核，是一个基于POSIX和Unix的多用户、多任务、支持多线程和多CPU的操作系统内核。它能运行主要的Unix工具软件、应用程序和网络协议。它支持32位和64位硬件。Linux继承了Unix以网络为核心的设计思想，是一个性能稳定的多用户网络操作系统内核。

Linux 是一个操作系统。它是一位赫尔辛基大学学生 Linus Torvalds（Linux 是 Linus's UNIX 的缩写）在 1991 年 10 月创造的。

## Linux 和 UNIX 关系？

UNIX（此名称是源自以前的“Multics”操作系统）于 1969 年在 AT&T 贝尔实验室被创造出来，它是一种健壮的、灵活的和对开发人员友好的计算环境。尽管 UNIX 最初是为 Digital Equipment Corporation（DEC）的 PDP 微型计算机系列编写的，但它却成为最受欢迎的多用户通用操作系统，并已在所有计算领域 ― 甚至包括曾一度被大型机垄断的领域 ― 占据主导地位。

## Linux 和 windows 比较

目前国内Linux更多的是应用于服务器上，而桌面操作系统更多使用的是Window。主要区别如下：

| 比较     | Windows                                                      | Linux                                                        |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 界面     | 界面统一，外壳程序固定所有Windows程序菜单几乎一致，快捷键也几乎相同 | 图形界面风格依发布版不同而不同，可能互不兼容。GNU/Linux的终端机是从UNIX传承下来，基本命令和操作方法也几乎一致。 |
| 驱动程序 | 驱动程序丰富，版本更新频繁。默认安装程序里面一般包含有该版本发布时流行的硬件驱动程序，之后所出的新硬件驱动依赖于硬件厂商提供。对于一些老硬件，如果没有了原配的驱动有时很难支持。另外，有时硬件厂商未提供所需版本的Windows下的驱动，也会比较头痛。 | 由志愿者开发，由Linux核心开发小组发布，很多硬件厂商基于版权考虑并未提供驱动程序，尽管多数无需手动安装，但是涉及安装则相对复杂，使得新用户面对驱动程序问题（是否存在和安装方法）会一筹莫展。但是在开源开发模式下，许多老硬件尽管在Windows下很难支持的也容易找到驱动。HP、Intel、AMD等硬件厂商逐步不同程度支持开源驱动，问题正在得到缓解。 |
| 使用     | 使用比较简单，容易入门。图形化界面对没有计算机背景知识的用户使用十分有利。 | 图形界面使用简单，容易入门。文字界面，需要学习才能掌握。     |
| 学习     | 系统构造复杂、变化频繁，且知识、技能淘汰快，深入学习困难。   | 系统构造简单、稳定，且知识、技能传承性好，深入学习相对容易。 |
| 软件     | 每一种特定功能可能都需要商业软件的支持，需要购买相应的授权。 | 大部分软件都可以自由获取，同样功能的软件选择较少。           |

## 镜像下载地址
- centos镜像地址: `http://mirror.centos.org/`

- 阿里云镜像地址：`https://mirrors.aliyun.com/centos/`

- 网易开源镜像站: `http://mirrors.163.com/`

- 搜狐镜像地址：`http://mirrors.sohu.com/centos/`

## Linux 目录

**通过命令查看目录：**

```
[root@localhost ~]# ls /
bin   dev  home  lib64  mnt  proc  run   srv  tmp  var
boot  etc  lib   media  opt  root  sbin  sys  usr
```

**目录解释：**

| 目录        | 功能（作用）                                                 |
| ----------- | ------------------------------------------------------------ |
| /bin        | *bin是Binary的缩写, 这个目录存放着最经常使用的命令。         |
| /boot       | *这里存放的是启动Linux时使用的一些核心文件，包括一些连接文件以及镜像文件。 |
| /dev        | dev是Device(设备)的缩写, 该目录下存放的是Linux的外部设备，在Linux中访问设备的方式和访问文件的方式是相同的。 |
| /etc        | 这个目录用来存放所有的系统管理所需要的配置文件和子目录。     |
| /home       | *用户的主目录，在Linux中，每个用户都有一个自己的目录，一般该目录名是以用户的账号命名的。 |
| /lib        | 这个目录里存放着系统最基本的动态连接共享库，其作用类似于Windows里的DLL文件。几乎所有的应用程序都需要用到这些共享库。 |
| /lost+found | 这个目录一般情况下是空的，当系统非法关机后，这里就存放了一些文件。 |
| /media      | *linux系统会自动识别一些设备，例如U盘、光驱等等，当识别后，linux会把识别的设备挂载到这个目录下。 |
| /mnt        | *系统提供该目录是为了让用户临时挂载别的文件系统的，我们可以将光驱挂载在/mnt/上，然后进入该目录就可以查看光驱里的内容了。 |
| /opt        | 这是给主机额外安装软件所摆放的目录。比如你安装一个ORACLE数据库则就可以放到这个目录下。默认是空的。 |
| /proc       | 这个目录是一个虚拟的目录，它是系统内存的映射，我们可以通过直接访问这个目录来获取系统信息。 |
| /root       | 该目录为系统管理员，也称作超级权限者的用户主目录。           |
| /sbin       | *s就是Super User的意思，这里存放的是系统管理员使用的系统管理程序。 |
| /sys        | 这是linux2.6内核的一个很大的变化。该目录下安装了2.6内核中新出现的一个文件系统 sysfs 。sysfs文件系统集成了下面3种文件系统的信息：针对进程信息的proc文件系统、针对设备的devfs文件系统以及针对伪终端的devpts文件系统。 |
| /tmp        | 这个目录是用来存放一些临时文件的。                           |
| /usr        | 这是一个非常重要的目录，用户的很多应用程序和文件都放在这个目录下，类似于windows下的program files目录。 |
| /var        | *这个目录中存放着在不断扩充着的东西，我们习惯将那些经常被修改的目录放在这个目录下。包括各种日志文件。 |
| /srv        | 该目录存放一些服务启动之后需要提取的数据。                   |
| /run        | 是一个临时文件系统，存储系统启动以来的信息。当系统重启时，这个目录下的文件应该被删掉或清除。如果你的系统上有 /var/run 目录，应该让它指向 run。 |


## 文章参考

- *https://baike.baidu.com/item/linux/27050*
- *https://www.ibm.com/developerworks/cn/linux/newto/*
- *https://www.runoob.com/linux/linux-system-contents.html*

## 代码示例

更多系列文章请访问下面查看仓库：

- *github：* [https://github.com/mtcarpenter/spring-data-chapter](https://github.com/mtcarpenter/spring-data-chapter)
- *gitee* :      [https://gitee.com/mtcarpenter/spring-data-chapter](https://gitee.com/mtcarpenter/spring-data-chapter)