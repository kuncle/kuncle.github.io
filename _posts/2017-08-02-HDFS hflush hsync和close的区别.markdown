---
layout: post
title:  "HDFS hflush hsync和close的区别"
date:   2017-08-02 21:45:00
categories: Hadoop
tags: Hadoop
---
* hflush: 语义是保证flush的数据被新的reader读到，但是不保证数据被datanode持久化.
* hsync: 与hflush几乎一样，不同的是hsync保证数据被datanode持久化。
* close: 关闭文件.除了做到以上2点，还保证文件的所有block处于completed状态，并且将文件置为closed
场景是写一个字节(append或者create)
* hflush
涉及到几个线程，一个是调FSDataOutputStream的write的线程，它是应用程序自己的线程,write会将数据以packet的形式一个个的丢入data queue中。
另外两个是HDFS客户端代码中的线程，其中一个是DataStreamer，负责从data queue中取出一个个的packet发出去，过程是为每个block建立一个pipeline，然后发packet，最后关闭pipeline，接着下一个block。另外一个是ResponseProcessor，负责处理下游节点的ack。   
实际上，packet是由chunk组成的，每个chunk对应一个checksum，一个packet大概64KB左右，一个chunk通常512字节。通常情况下，每512字节算一个checksum，写入到packet中。但是最后一个chunk通常是不满512字节。hflush实际上，就是将最后不满一个chunk的数据算checksum，然后写入packet，最后将这个packet放入data queue队列.在我们只写一个字节的场景下，一个字节不够一个chunk，故data queue中始终每个packet，DataStreamer始终等待着没有建立pipeline，调用hflush后，往data queue塞入一个packet，DataStreamer终于从data queue中取到一个packet，然后建立pipeline，接着发送packet。调完hflush的应用程序线程一直在等待最后一个packet的ack被收到，轮到ResponseProcessor上场。他不断的处理从datanode收到的packet ack，不断更新block的长度。接着，执行hflush的应用程序线程终于等到了最后一个packet的ack，然后它告诉namenode最后一个block的长度，namenode更新内存状态，实际上是根据文件名找到INodeFile，将block长度写入，并且记一条edit log.
* FSDataOutputStream的close
一开始也是和hflush一样，将最后一个packet进data queue，不同的是还会生成一个特殊的packet入data queue，lastPacketInBlock标记设为true,意思是告诉datanode这是block的最后一个packet，然后等最后这个包的ack收到。接着关闭DataStreamer和ResponseProcessor线程。然后调用completeFile(),最后结束file lease.     
看看completeFile()：通知namenode，namenode会做一些检查：   
根据文件名从目录树中拿出INode，检查文件是否处于under construction状态，如果不是，则complete file失败.
从INode中拿出修改这个文件的lease holder和当前completeFile()这个客户端比较，看是否是同一个client，如果不是，则complete file失败(namenode从目录树中得到当前打开文件的信息，会定期检查打开的文件的lease是否超过hard limit，默认1小时，如果超过了，会强行将文件的lease设置为namenode，这样，client 就不能向namenode commit block了。)
namenode会检查文件的倒数第二个block是否已经是completed状态，如果不是客户端重试，否则，将最后一个block变成completed状态，其实就是修改一下内存中数据结构，写一条edit log。一个block是completed状态的条件是满足最低副本数要求，默认配置1,配置项DFS_NAMENODE_REPLICATION_MIN_KEY.当datanode收到一个block后，会向namenode汇报，只要有一个datanode汇报成功，namenode就将block置为completed.最后namenode将file置为closed状态。
* hsync
hsync()执行时，实际上会在对应Datanode的机器上产生一个fsync的系统调用，从而将内存中的相关文件的数据更新到磁盘。   
Client端执行hsync时，Datanode端会识别到Client发送过来的数据包中的syncBlock_字段为true，从而判定需要将内存中的数据更新到磁盘。此时会在BlockReceiver.java的flushOrSync()中执行如下语句：   
((FileOutputStream)cout).getChannel().force(true);    
而FileChannel的force(boolean metadata)方法在JDK中，底层为于FileDispatcherImpl.c中调用fsync或fdatasync。metadata为true时执行fsync，为false时执行fdatasync。   
当Datanode将数据持久化到磁盘上后，会发ack响应给Client端。当收到所有Datanode的ack响应时，hsync()的调用结束。   
值得注意的是，fsync或fdatasync本身是一个非常耗时的调用，因为磁盘的读写速度远低于内存的读写速度。在不调用fsync或fdatasync的情况下，数据可能保存在各级cache中。

