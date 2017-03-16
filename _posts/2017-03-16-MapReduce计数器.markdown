---
layout: post
title:  "MapReduce计数器"
date:   2017-03-16 10:10:00
categories: Hadoop
tags: Hadoop
---
####MapReduce计数器
运行MapReduce的example包下的WordCount,计数器日志信息如下：
``` shell
File System Counters 文件系统计数器
                FILE: Number of bytes read=78 
                FILE: Number of bytes written=237975
                FILE: Number of read operations=0
                FILE: Number of large read operations=0
                FILE: Number of write operations=0
                HDFS: Number of bytes read=127
                HDFS: Number of bytes written=36
                HDFS: Number of read operations=6
                HDFS: Number of large read operations=0
                HDFS: Number of write operations=2
        Job Counters 作业计数器
                Launched map tasks=1
                Launched reduce tasks=1
                Data-local map tasks=1
                Total time spent by all maps in occupied slots (ms)=40326
                Total time spent by all reduces in occupied slots (ms)=89505
                Total time spent by all map tasks (ms)=40326
                Total time spent by all reduce tasks (ms)=89505
                Total vcore-milliseconds taken by all map tasks=40326
                Total vcore-milliseconds taken by all reduce tasks=89505
                Total megabyte-milliseconds taken by all map tasks=41293824
                Total megabyte-milliseconds taken by all reduce tasks=91653120
        Map-Reduce Framework MapReduce框架计数器
                Map input records=6
                Map output records=11
                Map output bytes=66
                Map output materialized bytes=78
                Input split bytes=105
                Combine input records=11
                Combine output records=9
                Reduce input groups=9
                Reduce shuffle bytes=78
                Reduce input records=9
                Reduce output records=9
                Spilled Records=18
                Shuffled Maps =1
                Failed Shuffles=0
                Merged Map outputs=1
                GC time elapsed (ms)=172
                CPU time spent (ms)=1600
                Physical memory (bytes) snapshot=331804672
                Virtual memory (bytes) snapshot=1755283456
                Total committed heap usage (bytes)=168562688
        Shuffle Errors Shuffle错误计数器
                BAD_ID=0
                CONNECTION=0
                IO_ERROR=0
                WRONG_LENGTH=0
                WRONG_MAP=0
                WRONG_REDUCE=0
        File Input Format Counters 文件输入格式计数器
                Bytes Read=22
        File Output Format Counters 文件输出格式计数器
                Bytes Written=36
```
