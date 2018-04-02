---
layout: post
title:  "Hive On Spark"
date:   2017-12-05 16:30:00
categories: Hive
tags: Hive
---

COALESCE
要用hive的这个函数路径重新注册这个函数
可能这个COALESCE在spark sql中有同名函数

#### exit code 143
``` shell
17/12/08 10:26:59 WARN yarn.YarnAllocator: Container marked as failed: container_1512699337730_0003_01_000003 on host: slave2. Exit status: 143. Diagnostics: Container killed on request. Exit code is 143
Container exited with a non-zero exit code 143
Killed by external signal
```




1. Run spark add conf bellow
``` shell
--conf 'spark.driver.extraJavaOptions=-XX:+UseCompressedOops -verbose:gc -XX:+PrintGCDetails -XX:+PrintGCTimeStamps' \
--conf 'spark.executor.extraJavaOptions=-XX:+UseCompressedOops -verbose:gc -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:+PrintHeapAtGC  ' \
```

2. When jvm GC ,you will get follow message
Heap after GC invocations=157 (full 98)
``` shell
 PSYoungGen      total 940544K, used 853456K [0x0000000781800000, 0x00000007c0000000, 0x00000007c0000000)
  eden space 860160K, 99% used [0x0000000781800000,0x00000007b5974118,0x00000007b6000000)
  from space 80384K, 0% used [0x00000007b6000000,0x00000007b6000000,0x00000007bae80000)
  to   space 77824K, 0% used [0x00000007bb400000,0x00000007bb400000,0x00000007c0000000)
 ParOldGen       total 2048000K, used 2047964K [0x0000000704800000, 0x0000000781800000, 0x0000000781800000)
  object space 2048000K, 99% used [0x0000000704800000,0x00000007817f7148,0x0000000781800000)
 Metaspace       used 43044K, capacity 43310K, committed 44288K, reserved 1087488K
  class space    used 6618K, capacity 6701K, committed 6912K, reserved 1048576K  
}
```

3. Both PSYoungGen and ParOldGen are 99% ,then you will get java.lang.OutOfMemoryError: GC overhead limit exceeded if more object was created

4. Try to add more memory for your executor or your driver when more memory resources are avaliable
``` shell
--executor-memory 10000m \
--driver-memory 10000m \
```

5. For my case : memory for PSYoungGen are smaller then ParOldGen which causes many young object enter into ParOldGen memory area and finaly ParOldGen are not avaliable.So java.lang.OutOfMemoryError: Java heap space error appear.

6. Adding conf for executor
``` shell
'spark.executor.extraJavaOptions=-XX:NewRatio=1 -XX:+UseCompressedOops -verbose:gc -XX:+PrintGCDetails -XX:+PrintGCTimeStamps '
-XX:NewRatio=rate rate = ParOldGen/PSYoungGen
```
It dependends.You can try GC strategy like
``` shell
-XX:+UseSerialGC :Serial Collector
-XX:+UseParallelGC :Parallel Collector
-XX:+UseParallelOldGC :Parallel Old collector
-XX:+UseConcMarkSweepGC :Concurrent Mark Sweep
```
Java Concurrent and Parallel GC

7. If both step 4 and step 6 are done but still get error, you should consider change you code. For example, reduce iterator times in ML model.
