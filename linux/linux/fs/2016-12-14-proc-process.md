---
layout: post
title:  linux程序分析工具介绍(一)—-”/proc”
categories: [Linux]
tags: [Linux]
description: ""
---

&emsp;&emsp;&emsp;&emsp;写在最前面：在开始本文之前，笔者认为先有必要介绍一下linux下的man，
如果读者手头用linux系统，直接在终端输入man man便可以看到详细的说明，我在这里简单的总结一下，
man命令是用来查看linux下各种命令、工具等的用户手册(manual)的。一种比较常用的用法是”man n field”，
这里的n是要查找的手册了类型，field是关键字。在这里介绍一下n：

* 0 /usr/include下的头文件
* 1 可执行程序和shell命令
* 2 系统调用
* 3 系统库函数
* 4 /dev下的特殊文件
* 5 文件格式和约定（比如/etc/passwd)
* 6 游戏
* 7 其它
* 8 仅root可用的系统管理命令
* 9 内核相关的内容

&emsp;&emsp;&emsp;&emsp;通常情况下，如果不加n的话，系统会按一定的顺序，有时候得到的可能不是你想要的，
这时候就需要加上n了，这就是我要介绍n的目的。比如，你man printf，系统返回的肯定是shell命令printf，
你要看库函数printf怎么办呢，那就man 3 printf，that’s ok

&emsp;&emsp;&emsp;&emsp;下面进入今天的正题，/proc是linux系统为我们用户提供的一个可以用来访问
系统相关数据及信息的一个伪文件 系统，通过它我们不仅可以获取指定某个进程的相关信息，
还可以获取系统整体的运行情况及信息。因为本文讲的是分析程序的工具，所以本文将侧重介绍通过
/proc来分析程序本身，关于如何通过/proc来查看系统相关信息，可以通过man 5 proc来看
（这也是我开始就讲man的一个原因 😛 ）。

* /proc/[number]/cmdline 程序命令行参数，以’\0’分隔的字符串文件（在程序中，可以通过直接读此文件，
获取程序的命令行参数，但不推荐这么做，这样做了程序的可移植性不好 😥 ）
* /proc/[number]/cwd     程序的当前工作路径的软链接(readlink就可以得到被链接的目录)
* /proc/[number]/environ 程序的当前环境变量，以’\0’分隔的字符串文件
* /proc/[number]/exe     程序的可执行文件的软链接（通过readlink可以获取程序可执行文件的完整路径）
* /proc/[number]/fd      程序当前正在使用的fd，这些fd都链向实际的文件
* /proc/[number]/maps    程序的地址空间分布和访问权限（通过这些信息，可以查看进程的地址是否在合法

```
1  address           perms offset  dev   inode      pathname
2  08048000-08056000 r-xp 00000000 03:0c 64593      /usr/sbin/gpm
3  08056000-08058000 rw-p 0000d000 03:0c 64593      /usr/sbin/gpm
4  08058000-0805b000 rwxp 00000000 00:00 0
5  40000000-40013000 r-xp 00000000 03:0c 4165       /lib/ld-2.2.4.so
6  40013000-40015000 rw-p 00012000 03:0c 4165       /lib/ld-2.2.4.so
7  4001f000-40135000 r-xp 00000000 03:0c 45494      /lib/libc-2.2.4.so
8  40135000-4013e000 rw-p 00115000 03:0c 45494      /lib/libc-2.2.4.so
9  4013e000-40142000 rw-p 00000000 00:00 0
10 bffff000-c0000000 rwxp 00000000 00:00 0
```

* /proc/[number]/smaps (since Linux 2.6.14) 程序的每块内存映射区域的内存使用情况

```
1  08048000-080bc000 r-xp 00000000 03:02 13130 /bin/bash    #与maps中的相同
2  Size:               464 kB                       #映射区的大小
3  Rss:                424 kB                       #实际在内存中的大小
4  Shared_Clean:       424 kB 
5  Shared_Dirty:         0 kB
6  Shared_Dirty:         0 kB
7  4Private_Dirty:        0 kB
```

* /proc/[number]/stat 程序的状态信息，ps命令得到的程序信息就是从此处获取的，因此详细的因容可以ps命令
* /proc/[number]/statm 程序的内存页(page)状态

```
1 size       total program size
2 resident   resident set size
3 share      shared pages
4 text       text (code)
5 lib        library
6 data       data/stack
7 dt         dirty pages (unused in Linux 2.6)
```
