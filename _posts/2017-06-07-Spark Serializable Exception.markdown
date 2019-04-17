---
layout: post
title:  "Spark Serializable Exception"
date:   2017-06-07 11:40:00
categories: Spark
tags: Spark
---
#### object not serializable (class: scala.util.matching.Regex$Match xxx)
* 解决方案
``` shell
val spark = SparkSession
...
spark.config("spark.serializer", "org.apache.spark.serializer.KryoSerializer")
```
