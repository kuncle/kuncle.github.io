---
layout: post
title:  "MapReduce的shuffle阶段问题解惑"
date:   2017-03-17 13:40:00
categories: Hadoop
tags: Hadoop
---
#### Shuffle产生的意义是什么?
Shuffle过程的期望可以有:
完整地从map task端拉取数据到reduce 端.
在跨节点拉取数据时，尽可能地减少对带宽的不必要消耗.
减少磁盘IO对task执行的影响.

#### 每个map task都有一个内存缓冲区,存储着map的输出结果,当缓冲区快满的时候需要将缓冲区的数据该如何处理?
当缓冲区快满的时候需要将缓冲区的数据以一个临时文件的方式存放到磁盘,当整个map task结束后再对磁盘中这个map task产生的所有临时文件做合并,生成最终的正式输出文件,然后等待reduce task来拉数据.

#### MapReduce提供Partitioner接口,它的作用是什么?
Partitioner它的作用就是根据key或value及reduce的数量来决定当前的这对输出数据最终应该交由哪个reduce task处理.默认对key hash后再以reduce task数量取模.默认的取模方式只是为了平均reduce的处理能力,如果用户自己对Partitioner有需求,可以订制并设置到job上. 

#### 什么是溢写?
在一定条件下将缓冲区中的数据临时写入磁盘,然后重新利用这块缓冲区/这个从内存往磁盘写数据的过程被称为Spill,中文可译为溢写.

#### 溢写是为什么不影响往缓冲区写map结果的线程?
溢写线程启动时不应该阻止map的结果输出,所以整个缓冲区有个溢写的比例spill.percent.这个比例默认是0.8,也就是当缓冲区的数据已经达到阈值（buffer size * spill percent = 100MB * 0.8 = 80MB）,溢写线程启动，锁定这80MB的内存,执行溢写过程.Map task的输出结果还可以往剩下的20MB内存中写,互不影响.

#### 当溢写线程启动后,需要对这80MB空间内的key做排序(Sort).排序是MapReduce模型默认的行为,这里的排序也是对谁的排序?
需要对这80MB空间内的key做排序(Sort).排序是MapReduce模型默认的行为,这里的排序也是对序列化的字节做的排序. 

#### 溢写过程中如果有很多个key/value对需要发送到某个reduce端去,那么如何处理这些key/value值?
如果有很多个key/value对需要发送到某个reduce端去,那么需要将这些key/value值拼接到一块,减少与partition相关的索引记录.

#### 哪些场景才能使用Combiner呢?
Combiner的输出是Reducer的输入,Combiner绝不能改变最终的计算结果.所以Combiner只应该用于那种Reduce的输入key/value与输出key/value类型完全一致,且不影响最终结果的场景.比如累加,最大值等.Combiner的使用一定得慎重,如果用好,它对job执行效率有帮助,反之会影响reduce的最终结果. 

#### Merge的作用是什么?
最终磁盘中会至少有一个这样的溢写文件存在(如果map的输出结果很少,当map执行完成时,只会产生一个溢写文件),因为最终的文件只有一个,所以需要将这些溢写文件归并到一起,这个过程就叫做Merge.

#### 每个reduce task不断的通过什么协议从JobTracker那里获取map task是否完成的信息?
RPC协议

#### reduce中Copy过程采用是什么协议?
HTTP协议

#### reduce中merge过程有几种方式?
merge有三种形式：1)内存到内存  2)内存到磁盘  3)磁盘到磁盘.


