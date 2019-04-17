---
layout: post
title:  "Linux Daemontools Supervise"
date:   2017-11-22 11:00:00
categories: Linux
tags: Linux
---
* supervise是daemontools的一个工具，可以用来监控管理unix下的应用程序运行情况，在应用程序出现异常时，supervise可以重新启动指定程序。
* 安装
``` shell
(http://cr.yp.to/daemontools/install.html)
How to install daemontools

Like any other piece of software (and information generally), daemontools comes with NO WARRANTY.
System requirements

daemontools works only under UNIX.
Installation

Create a /package directory:
     mkdir -p /package
     chmod 1755 /package
     cd /package
Download daemontools-0.76.tar.gz into /package. Unpack the daemontools package:
     gunzip daemontools-0.76.tar
     tar -xpf daemontools-0.76.tar
     rm -f daemontools-0.76.tar
     cd admin/daemontools-0.76
Compile and set up the daemontools programs:
     package/install
On BSD systems, reboot to start svscan.
To report success:

     mail djb-sysdeps@cr.yp.to < /package/admin/daemontools/compile/sysdeps
```
