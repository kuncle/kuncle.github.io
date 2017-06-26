---
layout: post
title:  "Yarn权限配置"
date:   2017-03-13 21:25:00
categories: Scala
tags: Scala
---
Exception
``` shell
Error:(32, 48) value filter is not a member of java.util.List[java.util.Map[String,AnyRef]]
    for(res: java.util.Map[String, AnyRef]  <- result){
```
* 这是在scala中引用Java的集合进行迭代抛出的异常
* 解决方案为import scala.collection.JavaConversions._  (Scala支持与Java的隐式转换)
