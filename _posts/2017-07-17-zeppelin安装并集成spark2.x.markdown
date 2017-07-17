---
layout: post
title:  "zeppelin安装并集成spark2.x"
date:   2017-07-17 11:00:00
categories: Zeppelin
tags: Zeppelin
---
* 下载zeppelin:zeppelin.apache.org下载 zeppelin-0.7.2-bin-all.tgz（本次使用的是0.7.2）
* 解压 tar zxvf zeppelin-0.7.2-bin-all.tgz
* 修改配置文件 
``` shell
cp zeppelin-env.sh.template zeppelin-env.sh
cp zeppelin-site.xml.template zeppelin-site.xml

vi zeppelin-env.sh
export SPARK_HOME=/opt/cloudera/parcels/SPARK2/lib/spark2
export HADOOP_CONF_DIR=/opt/cloudera/parcels/CDH/lib/hadoop
export ZEPPELIN_INTP_CLASSPATH_OVERRIDES=/etc/hive/conf
export MASTER="yarn-client"
export ZEPPELIN_PID_DIR=/var/run/zeppelin
export ZEPPELIN_LOG_DIR=/var/log/zeppelin
export ZEPPELIN_CLASSPATH="${SPARK_CLASSPATH}"

vi zeppelin-site.xml
<property>
  <name>zeppelin.server.port</name>
  <value>7080</value>
  <description>Server port.</description>
</property>
```
* 启动 ./bin/zeppelin-daemon.sh start

* 异常
``` shell
1, java.lang.NoSuchMethodError: org.apache.hadoop.io.retry.RetryPolicies.retryOtherThanRemoteException
(Lorg/apache/hadoop/io/retry/RetryPolicy;Ljava/util/Map;)Lorg/apache/hadoop/io/retry/RetryPolicy;
解决方案：替换hadoop jar
mv zeppelin-0.7.2-bin-all/lib/hadoop-annotations-*.jar /opt/zeppelin-0.7.2-bin-all/lib/hadoop-annotations-*.jar.bak
mv zeppelin-0.7.2-bin-all/lib/hadoop-auth-*.jar /opt/zeppelin-0.7.2-bin-all/lib/hadoop-auth-*.jar.bak
mv zeppelin-0.7.2-bin-all/lib/hadoop-common-*.jar /opt/zeppelin-0.7.2-bin-all/lib/hadoop-common-*.jar.bak
cp /opt/cloudera/parcels/CDH/lib/hadoop/lib/hadoop-annotations-*.jar /opt/zeppelin-0.7.2-bin-all/lib
cp /opt/cloudera/parcels/CDH/lib/hadoop/lib/hadoop-auth-*.jar /opt/zeppelin-0.7.2-bin-all/lib
cp /opt/cloudera/parcels/CDH/lib/hadoop/lib/hadoop-common-*.jar /opt/zeppelin-0.7.2-bin-all/lib

2, com.fasterxml.jackson.databind.JsonMappingException: Jackson version is too old 2.5.3
解决方案：替换Jackson jar
mv zeppelin-0.7.2-bin-all/lib/jackson-annotations-2.5.3.jar /opt/zeppelin-0.7.2-bin-all/lib/jackson-annotations-2.5.3.jar.bak
mv zeppelin-0.7.2-bin-all/lib/jackson-core-2.5.3.jar /opt/zeppelin-0.7.2-bin-all/lib/jackson-core-2.5.3.jar
mv zeppelin-0.7.2-bin-all/lib/jackson-databind-2.5.3.jar /opt/zeppelin-0.7.2-bin-all/lib/jackson-databind-2.5.3.jar.bak
cp /opt/cloudera/parcels/SPARK2/lib/spark2/jars/jackson-annotations-2.6.5.jar /opt/zeppelin-0.7.2-bin-all/lib
cp /opt/cloudera/parcels/SPARK2/lib/spark2/jars/jackson-core-2.6.5.jar /opt/zeppelin-0.7.2-bin-all/lib
cp /opt/cloudera/parcels/SPARK2/lib/spark2/jars/jackson-databind-2.6.5.jar /opt/zeppelin-0.7.2-bin-all/lib
```
