---
layout: post
title:  "Kafka Cleanup Policy"
date:   2019-07-12 14:25:00
categories: Kafka
tags: Kafka
---
#### Compaction
Log Compaction的大部分功能由CleanerThread完成, 核心逻辑在Cleaner的clean方法
``` java
class LogCleaner(val config: CleanerConfig,
                 val logDirs: Array[File],
                 val logs: Pool[TopicPartition, Log],
                 time: Time = Time.SYSTEM) extends Logging with KafkaMetricsGroup {
                 ......
  /**
   * The cleaner threads do the actual log cleaning. Each thread processes does its cleaning repeatedly by
   * choosing the dirtiest log, cleaning it, and then swapping in the cleaned segments.
   */
  private class CleanerThread(threadId: Int)
    extends ShutdownableThread(name = "kafka-log-cleaner-thread-" + threadId, isInterruptible = false){
    ......
    val cleaner = new Cleaner(id = threadId,
                              offsetMap = new SkimpyOffsetMap(memory = math.min(config.dedupeBufferSize / config.numThreads, Int.MaxValue).toInt, 
                                                              hashAlgorithm = config.hashAlgorithm),
                              ioBufferSize = config.ioBufferSize / config.numThreads / 2,
                              maxIoBufferSize = config.maxMessageSize,
                              dupBufferLoadFactor = config.dedupeBufferLoadFactor,
                              throttler = throttler,
                              time = time,
                              checkDone = checkDone)
    ......
  }
  ......
}
```
注意SkimpyOffsetMap的参数hashAlgorithm = config.hashAlgorithm. 而config为CleanerConfig.
``` java
/**
 * This class holds the actual logic for cleaning a log
 * @param id An identifier used for logging
 * @param offsetMap The map used for deduplication
 * @param ioBufferSize The size of the buffers to use. Memory usage will be 2x this number as there is a read and write buffer.
 * @param maxIoBufferSize The maximum size of a message that can appear in the log
 * @param dupBufferLoadFactor The maximum percent full for the deduplication buffer
 * @param throttler The throttler instance to use for limiting I/O rate.
 * @param time The time instance
 * @param checkDone Check if the cleaning for a partition is finished or aborted.
 */
private[log] class Cleaner(val id: Int,
                           val offsetMap: OffsetMap,
                           ioBufferSize: Int,
                           maxIoBufferSize: Int,
                           dupBufferLoadFactor: Double,
                           throttler: Throttler,
                           time: Time,
                           checkDone: (TopicPartition) => Unit) extends Logging {
                ......
  /**
   * Clean the given log
   *
   * @param cleanable The log to be cleaned
   *
   * @return The first offset not cleaned and the statistics for this round of cleaning
   */
  private[log] def clean(cleanable: LogToClean): (Long, CleanerStats) = {
    val stats = new CleanerStats()

    info("Beginning cleaning of log %s.".format(cleanable.log.name))
    val log = cleanable.log

    // build the offset map
    info("Building offset map for %s...".format(cleanable.log.name))
    val upperBoundOffset = cleanable.firstUncleanableOffset
    buildOffsetMap(log, cleanable.firstDirtyOffset, upperBoundOffset, offsetMap, stats)
    val endOffset = offsetMap.latestOffset + 1
    stats.indexDone()
    
    // figure out the timestamp below which it is safe to remove delete tombstones
    // this position is defined to be a configurable time beneath the last modified time of the last clean segment
    val deleteHorizonMs = 
      log.logSegments(0, cleanable.firstDirtyOffset).lastOption match {
        case None => 0L
        case Some(seg) => seg.lastModified - log.config.deleteRetentionMs
    }

    // determine the timestamp up to which the log will be cleaned
    // this is the lower of the last active segment and the compaction lag
    val cleanableHorizonMs = log.logSegments(0, cleanable.firstUncleanableOffset).lastOption.map(_.lastModified).getOrElse(0L)

    // group the segments and clean the groups
    info("Cleaning log %s (cleaning prior to %s, discarding tombstones prior to %s)...".format(log.name, new Date(cleanableHorizonMs), new Date(deleteHorizonMs)))
    for (group <- groupSegmentsBySize(log.logSegments(0, endOffset), log.config.segmentSize, log.config.maxIndexSize))
      cleanSegments(log, group, offsetMap, deleteHorizonMs, stats)

    // record buffer utilization
    stats.bufferUtilization = offsetMap.utilization
    
    stats.allDone()

    (endOffset, stats)
  }
  .......
}
```
可以发现log compaction通过两次遍历所有数据来实现, 两次遍历之间就个OffsetMap交互, 这个OffsetMap的实现是SkimpyOffsetMap.
``` java
/**
 * An hash table used for deduplicating the log. This hash table uses a cryptographicly secure hash of the key as a proxy for the key
 * for comparisons and to save space on object overhead. Collisions are resolved by probing. This hash table does not support deletes.
 * @param memory The amount of memory this map can use
 * @param hashAlgorithm The hash algorithm instance to use: MD2, MD5, SHA-1, SHA-256, SHA-384, SHA-512
 */
@nonthreadsafe
class SkimpyOffsetMap(val memory: Int, val hashAlgorithm: String = "MD5") extends OffsetMap{}
```
注意到hashAlgorithm默认是MD5, 前面Cleaner初始化SkimpyOffsetMap时, 这个参数是通过CleanerConfig获取.   
看看CleanerConfig结构.
``` java
/**
 * Configuration parameters for the log cleaner
 * 
 * @param numThreads The number of cleaner threads to run
 * @param dedupeBufferSize The total memory used for log deduplication
 * @param dedupeBufferLoadFactor The maximum percent full for the deduplication buffer
 * @param maxMessageSize The maximum size of a message that can appear in the log
 * @param maxIoBytesPerSecond The maximum read and write I/O that all cleaner threads are allowed to do
 * @param backOffMs The amount of time to wait before rechecking if no logs are eligible for cleaning
 * @param enableCleaner Allows completely disabling the log cleaner
 * @param hashAlgorithm The hash algorithm to use in key comparison.
 */
case class CleanerConfig(numThreads: Int = 1,
                         dedupeBufferSize: Long = 4*1024*1024L,
                         dedupeBufferLoadFactor: Double = 0.9d,
                         ioBufferSize: Int = 1024*1024,
                         maxMessageSize: Int = 32*1024*1024,
                         maxIoBytesPerSecond: Double = Double.MaxValue,
                         backOffMs: Long = 15 * 1000,
                         enableCleaner: Boolean = true,
                         hashAlgorithm: String = "MD5") {
}
```
搜索了初始化LogCleaner的地方.
``` java
val cleaner: LogCleaner =
    if(cleanerConfig.enableCleaner)
      new LogCleaner(cleanerConfig, liveLogDirs, currentLogs, logDirFailureChannel, time = time)
    else
      null
```
可以认为hashAlgorithm是MD5无疑了.   
顺便提一句, Kafka有个DynamicBrokerConfig的Object.
``` java
/**
  * Dynamic broker configurations are stored in ZooKeeper and may be defined at two levels:
  * <ul>
  *   <li>Per-broker configs persisted at <tt>/configs/brokers/{brokerId}</tt>: These can be described/altered
  *       using AdminClient using the resource name brokerId.</li>
  *   <li>Cluster-wide defaults persisted at <tt>/configs/brokers/&lt;default&gt;</tt>: These can be described/altered
  *       using AdminClient using an empty resource name.</li>
  * </ul>
  * The order of precedence for broker configs is:
  * <ol>
  *   <li>DYNAMIC_BROKER_CONFIG: stored in ZK at /configs/brokers/{brokerId}</li>
  *   <li>DYNAMIC_DEFAULT_BROKER_CONFIG: stored in ZK at /configs/brokers/&lt;default&gt;</li>
  *   <li>STATIC_BROKER_CONFIG: properties that broker is started up with, typically from server.properties file</li>
  *   <li>DEFAULT_CONFIG: Default configs defined in KafkaConfig</li>
  * </ol>
  * Log configs use topic config overrides if defined and fallback to broker defaults using the order of precedence above.
  * Topic config overrides may use a different config name from the default broker config.
  * See [[kafka.log.LogConfig#TopicConfigSynonyms]] for the mapping.
  * <p>
  * AdminClient returns all config synonyms in the order of precedence when configs are described with
  * <code>includeSynonyms</code>. In addition to configs that may be defined with the same name at different levels,
  * some configs have additional synonyms.
  * </p>
  * <ul>
  *   <li>Listener configs may be defined using the prefix <tt>listener.name.{listenerName}.{configName}</tt>. These may be
  *       configured as dynamic or static broker configs. Listener configs have higher precedence than the base configs
  *       that don't specify the listener name. Listeners without a listener config use the base config. Base configs
  *       may be defined only as STATIC_BROKER_CONFIG or DEFAULT_CONFIG and cannot be updated dynamically.<li>
  *   <li>Some configs may be defined using multiple properties. For example, <tt>log.roll.ms</tt> and
  *       <tt>log.roll.hours</tt> refer to the same config that may be defined in milliseconds or hours. The order of
  *       precedence of these synonyms is described in the docs of these configs in [[kafka.server.KafkaConfig]].</li>
  * </ul>
  *
  */
object DynamicBrokerConfig {
  ......
  val AllDynamicConfigs = DynamicSecurityConfigs ++
    LogCleaner.ReconfigurableConfigs ++
    DynamicLogConfig.ReconfigurableConfigs ++
    DynamicThreadPool.ReconfigurableConfigs ++
    Set(KafkaConfig.MetricReporterClassesProp) ++
    DynamicListenerConfig.ReconfigurableConfigs ++
    SocketServer.ReconfigurableConfigs
    
    .......
}
```

