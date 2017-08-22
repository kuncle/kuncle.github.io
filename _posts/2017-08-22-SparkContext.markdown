---
layout: post
title:  "SparkContext"
date:   2017-08-22 21:40:00
categories: Spark
tags: Spark
---
SparkContext是在Driver端创建，除了和ClusterManager通信，进行资源的申请、任务的分配和监控等以外还会在创建的时候会初始化各个核心组件，包括DAGScheduler，TaskScheduler，SparkEnv等。
``` shell
/**
 * Main entry point for Spark functionality. A SparkContext represents the connection to a Spark
 * cluster, and can be used to create RDDs, accumulators and broadcast variables on that cluster.
 *
 * Only one SparkContext may be active per JVM.  You must `stop()` the active SparkContext before
 * creating a new one.  This limitation may eventually be removed; see SPARK-2243 for more details.
 *  目前一个jvm只能存在一个SparkContext，未来可能会支持 可以看看https://issues.apache.org/jira/browse/SPARK-2243的讨论
 * @param config a Spark Config object describing the application configuration. Any settings in
 *   this config overrides the default configs as well as system properties.
 */
class SparkContext(config: SparkConf) extends Logging {

  // The call site where this SparkContext was constructed.
  获取当前SparkContext的当前调用堆栈，将栈里最靠近栈底的属于spark或者Scala核心的类压入callStack的栈顶，   
  并将此类的方法存入lastSparkMethod；将栈里最靠近栈顶的用户类放入callStack，将此类的行号存入firstUserLine，   
  类名存入firstUserFile，最终返回的样例类CallSite存储了最短栈和长度默认为20的最长栈的样例类
  private val creationSite: CallSite = Utils.getCallSite()

  // If true, log warnings instead of throwing exceptions when multiple SparkContexts are active
  private val allowMultipleContexts: Boolean =
    config.getBoolean("spark.driver.allowMultipleContexts", false)
```
接着是配置信息的获取与设置
``` shell
  /**
   * Return a copy of this SparkContext's configuration. The configuration ''cannot'' be
   * changed at runtime.  运行时不能修改configuration
   */
  def getConf: SparkConf = conf.clone()

  def jars: Seq[String] = _jars
  def files: Seq[String] = _files
  def master: String = _conf.get("spark.master")
  def deployMode: String = _conf.getOption("spark.submit.deployMode").getOrElse("client")
  def appName: String = _conf.get("spark.app.name")

  private[spark] def isEventLogEnabled: Boolean = _conf.getBoolean("spark.eventLog.enabled", false)
  private[spark] def eventLogDir: Option[URI] = _eventLogDir
  private[spark] def eventLogCodec: Option[String] = _eventLogCodec
  // 是否本地运行
  def isLocal: Boolean = Utils.isLocalMaster(_conf)
  
  // Set Spark driver host and port system properties. This explicitly sets the configuration
  // instead of relying on the default value of the config constant.
  设置driver host 和 port 以及executor.id等
  _conf.set(DRIVER_HOST_ADDRESS, _conf.get(DRIVER_HOST_ADDRESS))
  _conf.setIfMissing("spark.driver.port", "0")

  _conf.set("spark.executor.id", SparkContext.DRIVER_IDENTIFIER)

  _jars = Utils.getUserJars(_conf)
  _files = _conf.getOption("spark.files").map(_.split(",")).map(_.filter(_.nonEmpty))
    .toSeq.flatten
```
然后比较重要的是事件监听
``` shell
/**
 * Asynchronously passes SparkListenerEvents to registered SparkListeners.
 *
 * Until `start()` is called, all posted events are only buffered. Only after this listener bus
 * has started will events be actually propagated to all attached listeners. This listener bus
 * is stopped when `stop()` is called, and it will drop further events after stopping.
 */
 listenerBus里已经注册了很多监听者（listener），通常listenerBus会启动一个线程异步的调用这些listener去消费这个Event   
 （其实就是触发事先设计好的回调函数来执行譬如信息存储等动作）
  _listenerBus = new LiveListenerBus(_conf)
   
   // "_jobProgressListener" should be set up before creating SparkEnv because when creating
   // "SparkEnv", some messages will be posted to "listenerBus" and we should not miss them.
   负责监听事件并把事件消息发送给listenerBus
  _jobProgressListener = new JobProgressListener(_conf)
  listenerBus.addListener(jobProgressListener)
```
接着创建SparkEnv
``` shell
  // This function allows components created by SparkEnv to be mocked in unit tests:
  private[spark] def createSparkEnv(
      conf: SparkConf,
      isLocal: Boolean,
      listenerBus: LiveListenerBus): SparkEnv = {
      实际是创建的driverEnv
    SparkEnv.createDriverEnv(conf, isLocal, listenerBus, SparkContext.numDriverCores(master))
  }
```
