---
layout: post
title:  "Flume使用Hive Sink问题"
date:   2017-08-24 13:10:00
categories: Flume
tags: Flume
---
* Failed connecting to EndPoint
``` shell
Caused by: org.apache.hive.hcatalog.streaming.StreamingException: Cannot stream to table that has not been bucketed
```
使用Hive做为Sink，Hive表必须cluster by bucket

* ClassCastException
``` shell
ERROR - org.apache.flume.SinkRunner$PollingRunner.run(SinkRunner.java:160)] Unable to deliver event. Exception follows.
org.apache.flume.EventDeliveryException: java.lang.ClassCastException:    
org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat cannot be cast to org.apache.hadoop.hive.ql.io.AcidOutputFormat
```
通过查看源码，接口AcidOutputFormat的实现类只有OrcOutputFormat，所以Hive表需要stored as orc

* Column 'CHANNEL' not found 
``` shell
Caused by: org.apache.hive.hcatalog.streaming.InvalidColumn: Column 'CHANNEL' not found in table for input field 81
```
Flume配置的Hive 列名必须都为小写字母
