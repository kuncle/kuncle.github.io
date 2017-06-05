---
layout: post
title:  "Windows跑MR异常"
date:   2017-06-05 14:05:00
categories: Hadoop
tags: Hadoop
---
#### java.lang.UnsatisfiedLinkError: org.apache.hadoop.io.nativeio.NativeIO$Windows.access0(Ljava/lang/String;I)Z
* 解决方案
``` java
Windows本地已经配好了HADOOP_HOME,也安装了hadoop windows环境运行的所需插件
通过异常信息可以看出是NativeIO这个类报错了,又不想改源码后再次编译，可以直接在
项目中创建org.apache.hadoop.io.nativeio包,并创建NativeIO类,修改
return access0(path, desiredAccess.accessRight())为retuen true.
```
