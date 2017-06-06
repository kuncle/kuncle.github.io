---
layout: post
title:  "Cloudera集成Spark2.x启动异常"
date:   2017-06-06 16:55:00
categories: Spark
tags: Spark
---
#### 使用Cloudera Manager 5.10.0集成Spark2.1.0 启动异常
* Cloudera Manager 5.10.0集成Spark2.1.0步骤
https://www.cloudera.com/documentation/spark2/latest/topics/spark2_packaging.html
* 安装好Spark2后/etc/spark2/conf/ 下面是空的,可以先拷贝/etc/spark/conf/下的配置文件过来
``` shell
cp -r /etc/spark/conf.cloudera.spark_on_yarn/ /etc/spark2/conf/
``` 
* 修改spark2下面的配置
``` shell
vi spark-env.sh
export SPARK_HOME=/xxx/cloudera/parcels/SPARK2-2.1.0.cloudera1-1.cdh5.7.0.p0.120904/lib/spark2
```
``` shell
vi spark-defaults.conf
spark.yarn.jars=/xxx/cloudera/parcels/SPARK2-2.1.0.cloudera1-1.cdh5.7.0.p0.120904/lib/spark2/jars/* (这里你可以后续改为HDFS路径)
spark.master=yarn
```
