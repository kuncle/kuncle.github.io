---
layout: post
title:  "Cloudera分配Parcel卡住"
date:   2017-06-06 10:50:00
categories: Cloudera
tags: Cloudera
---
* 在Cloudera Manager5.10.0中准备取消Spark2.1.0的分配,可能由于我再一台机器上误删了部分文件,导致该机器一直无法取消分配
* cloudera-scm-agent日志提示为:Deleting unmanaged parcel xxxxxxx
* 解决方案:删除该机器 xxx/cloudera/parcel-repo目录下的Spark2.1.0相关文件,删除 xxx/cloudera/parcels/.flood/目录下的Spark2.1.0相关文件
* 重启cloudera-scm-agent服务
