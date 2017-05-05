---
layout: post
title:  "Cloudera中Oozie安装ExtJs"
date:   2017-05-05 09:50:00
categories: Cloudera
tags: Cloudera
---
Cloudera安装Oozie过后进入Oozie的管理界面时提示要安装ExtJs才能使用.

1. 下载 ExtJs的包wget http://archive.cloudera.com/gplextras/misc/ext-2.2.zip
2. 解压 unzip ext-2.2.zip 
3. 移动 ext-2.2文件夹到/var/lib/oozie目录下mv ext-2.2 /var/lib/oozie/
4. 授权 将ext-2.2文件夹拥有用户改为oozie chown -R oozie:oozie /var/lib/oozie/ext-2.2

再次进入oozie的web界面便可以正常使用了.
