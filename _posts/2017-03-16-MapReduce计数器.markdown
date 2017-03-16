---
layout: post
title:  "MapReduce计数器"
date:   2017-03-16 10:10:00
categories: Hadoop
tags: Hadoop
---
#### MapReduce计数器
运行MapReduce的example包下的WordCount,计数器日志信息如下：

###### File System Counters 文件系统计数器
* FILE: Number of bytes read=78 
> job读取本地文件系统的文件字节数.假定我们当前map的输入数据都来自于HDFS,那么在map阶段,这个数据应该是0.但reduce在执行前,它的输入数据是经过shuffle的merge后存储在reduce端本地磁盘中,所以这个数据就是所有reduce的总输入字节数
* FILE: Number of bytes written=237975 
> map的中间结果都会spill到本地磁盘中,在map执行完后,形成最终的spill文件.所以map端这里的数据就表示map task往本地磁盘中总共写了多少字节.与map端相对应的是,reduce端在shuffle时,会不断地拉取map端的中间结果,然后做merge并 不断spill到自己的本地磁盘中.最终形成一个单独文件,这个文件就是reduce的输入文件
* FILE: Number of read operations=0
* FILE: Number of large read operations=0
* FILE: Number of write operations=0
* HDFS: Number of bytes read=127 
> 整个job执行过程中,只有map端运行时,才从HDFS读取数据,这些数据不限于源文件内容,还包括所有map的split元数据.所以这个值应该比FileInputFormatCounters.BYTES_READ 要略大些
* HDFS: Number of bytes written=36 
> Reduce的最终结果都会写入HDFS,就是一个job执行结果的总量
* HDFS: Number of read operations=6
* HDFS: Number of large read operations=0
* HDFS: Number of write operations=2

###### Job Counters 作业计数器
* Launched map tasks=1
> job启动的map task个数 
* Launched reduce tasks=1 
> job启动的reduce task个数
* Data-local map tasks=1
> 数据本地化map task个数
* Total time spent by all maps in occupied slots (ms)=40326
> 所有map任务在被占用的slots中所用的时间.在yarn中,程序打成jar包提交给resourcemanager，nodemanager向resourcemanager申请资源，然后在nodemanager上运行,而划分资源(cpu,io，网络，磁盘)的单位叫容器container，每个节点上资源不是无限的,因此应该将任务划分为不同的容器,job在运行的时候可以申请job的数量,之后由nodemanager确定哪些任务可以执行map,那些可以执行reduce等,从而由slot表示,表示槽的概念.任务过来就占用一个槽
* Total time spent by all reduces in occupied slots (ms)=89505 
> 所有reduce任务在被占用的slots中所用的时间
* Total time spent by all map tasks (ms)=40326 (所有map执行时间
* Total time spent by all reduce tasks (ms)=89505 (所有reduce执行的时间
* Total vcore-milliseconds taken by all map tasks=40326
* Total vcore-milliseconds taken by all reduce tasks=89505
* Total megabyte-milliseconds taken by all map tasks=41293824
* Total megabyte-milliseconds taken by all reduce tasks=91653120
###### Map-Reduce Framework MapReduce框架计数器
* Map input records=6 
> 所有map task从HDFS读取的文件总行数)
* Map output records=11 
> map task的直接输出record是多少,就是在map方法中调用context.write的次数,也就是未经过Combine时的原生输出条数
* Map output bytes=66 
> Map的输出结果key/value都会被序列化到内存缓冲区中,所以这里的bytes指序列化后的最终字节之和
* Map output materialized bytes=78
* Input split bytes=105
* Combine input records=11 
> Combiner是为了减少尽量减少需要拉取和移动的数据,所以combine输入条数与map的输出条数是一致的
* Combine output records=9 
> 经过Combiner后,相同key的数据经过压缩,在map端自己解决了很多重复数据,表示最终在map端中间文件中的所有条目数
* Reduce input groups=9 
> Reduce总共读取了多少个这样的groups
* Reduce shuffle bytes=78 
> Reduce端的copy线程总共从map端抓取了多少的中间数据,表示各个map task最终的中间文件总和
* Reduce input records=9 
> 如果有Combiner的话,那么这里的数值就等于map端Combiner运算后的最后条数,如果没有,那么就应该等于map的输出条数
* Reduce output records=9 
> 所有reduce执行后输出的总条目数
* Spilled Records=18 
> spill过程在map和reduce端都会发生,这里统计在总共从内存往磁盘中spill了多少条数据
* Shuffled Maps =1 
> 每个reduce几乎都得从所有map端拉取数据,每个copy线程拉取成功一个map的数据,那么增1,所以它的总数基本等于 reduce number * map number
* Failed Shuffles=0 
> copy线程在抓取map端中间数据时,如果因为网络连接异常或是IO异常,所引起的shuffle错误次数
* Merged Map outputs=1 
> 记录着shuffle过程中总共经历了多少次merge动作
* GC time elapsed (ms)=172 
> 通过JMX获取到执行map与reduce的子JVM总共的GC时间消耗
* CPU time spent (ms)=1600
* Physical memory (bytes) snapshot=331804672
* Virtual memory (bytes) snapshot=1755283456
* Total committed heap usage (bytes)=168562688
###### Shuffle Errors Shuffle错误计数器
* BAD_ID=0 
> 每个map都有一个ID,如attempt_201703016150_0254_m_000000_0,如果reduce的copy线程抓取过来的元数据中这个ID不是标准格式,那么此Counter增加
* CONNECTION=0 
> 表示copy线程建立到map端的连接有误的次数
* IO_ERROR=0 
> Reduce的copy线程如果在抓取map端数据时出现IOException,那么这个值相应增加
* WRONG_LENGTH=0 
> map端的那个中间结果是有压缩好的有格式数据,所以它有两个length信息:源数据大小与压缩后数据大小.如果这两个length信息传输的有误(负值),那么此Counter增加
* WRONG_MAP=0 
> 每个copy线程当然是有目的:为某个reduce抓取某些map的中间结果,如果当前抓取的map数据不是copy线程之前定义好的map,那么就表示把数据拉错了
* WRONG_REDUCE=0 
> 与上面描述一致,如果抓取的数据表示它不是为此reduce而准备的,那还是拉错数据了
###### File Input Format Counters 文件输入格式计数器
* Bytes Read=22 
> Map task的所有输入数据(字节),等于各个map task的map方法传入的所有value值字节之和
###### File Output Format Counters 文件输出格式计数器
* Bytes Written=36 
> Reduce task的所有输出数据(字节),等于各个reduce task的reduce方法传入的所有value值字节之和
