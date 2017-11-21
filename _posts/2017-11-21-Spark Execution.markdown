---
layout: post
title:  "Spark Execution"
date:   2017-11-21 10:50:00
categories: Spark
tags: Spark
---
* Local Execution   
在该模式下，在一个含有多个线程的进程中执行task。
``` shell
$SPARK_HOME/bin/spark-submit --class org.apress.prospark.TranslateApp --master local[n]
./target/scala-2.10/FirstApp-assembly-1.0.jar <app_name> <book_path> <output_path> <language>
```

* Standalone Cluster
在该模式下，Driver可以在提交job的机器上执行： 
``` shell
$SPARK_HOME/bin/spark-submit --class org.apress.prospark.TranslateApp --master <master_
url> ./target/scala-2.10/FirstApp-assembly-1.0.jar <app_name> <book_path> <output_path>
<language>
```
![standalone_cluster1](/assets/img/standalone cluster1.png)
也可以在集群中运行Driver：   
``` shell
$SPARK_HOME/bin/spark-submit ¨Cclass org.apress.prospark.TranslateApp --master <master_url>
--deploy-mode cluster ./target/scala-2.10/FirstApp-assembly-1.0.jar <app_name> <book_path>
<output_path> <language>
```
![standalone_cluster2](/assets/img/standalone cluster2.png)
这两种方式的job都在work上执行，只是Driver的执行位置不同而已。

* Yarn
和standalone模式一样，Driver可以在Client执行：   
``` shell
$SPARK_HOME/bin/spark-submit --class org.apress.prospark.TranslateApp --master yarn --deploy-mode client
./target/scala-2.10/FirstApp-assembly-1.0.jar <app_name> <book_path> <output_path> <language>
```
也可以在集群中执行：   
``` shell
$SPARK_HOME/bin/spark-submit --class org.apress.prospark.TranslateApp --master yarn --deploy-mode cluster
./target/scala-2.10/FirstApp-assembly-1.0.jar <app_name> <book_path> <output_path>
<language>
```
