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
   负责监听事件并把事件消息发送给listenerBus  但是将要removed了           
   _jobProgressListener = new JobProgressListener(_conf)
  listenerBus.addListener(jobProgressListener)
```
接着创建SparkEnv
``` shell
    // Create the Spark execution environment (cache, map output tracker, etc)
    _env = createSparkEnv(_conf, isLocal, listenerBus)
    SparkEnv.set(_env)
   ......
  // This function allows components created by SparkEnv to be mocked in unit tests:
  private[spark] def createSparkEnv(
      conf: SparkConf,
      isLocal: Boolean,
      listenerBus: LiveListenerBus): SparkEnv = {
      实际是创建的driverEnv
    SparkEnv.createDriverEnv(conf, isLocal, listenerBus, SparkContext.numDriverCores(master))
  }
  ......
  /**
   * Create a SparkEnv for the driver.
   */
  private[spark] def createDriverEnv(
      conf: SparkConf,
      isLocal: Boolean,
      listenerBus: LiveListenerBus,
      numCores: Int,
      mockOutputCommitCoordinator: Option[OutputCommitCoordinator] = None): SparkEnv = {
     断言driver host & port
    assert(conf.contains(DRIVER_HOST_ADDRESS),
      s"${DRIVER_HOST_ADDRESS.key} is not set on the driver!")
    assert(conf.contains("spark.driver.port"), "spark.driver.port is not set on the driver!")
    val bindAddress = conf.get(DRIVER_BIND_ADDRESS)
    val advertiseAddress = conf.get(DRIVER_HOST_ADDRESS)
    val port = conf.get("spark.driver.port").toInt
    是否传输加密
    val ioEncryptionKey = if (conf.get(IO_ENCRYPTION_ENABLED)) {
      Some(CryptoStreamUtils.createKey(conf))
    } else {
      None
    }
    调用SparkEnv的create
  /**
   * Helper method to create a SparkEnv for a driver or an executor.
   */
    create(
      conf,
      SparkContext.DRIVER_IDENTIFIER,
      bindAddress,
      advertiseAddress,
      Option(port),
      isLocal,
      numCores,
      ioEncryptionKey,
      listenerBus = listenerBus,
      mockOutputCommitCoordinator = mockOutputCommitCoordinator
    )
    这个create包含SecurityManager，Serializer，BroadcastManager，registerOrLookupEndpoint，
    ShuffleManager，useLegacyMemoryManager，BlockManager，MetricsSystem等的创建
   }
  
```
然后是低级别状态报告api，负责监听job和stage的进度
``` shell
      /**
       * Low-level status reporting APIs for monitoring job and stage progress.
       *
       * These APIs intentionally provide very weak consistency semantics; consumers of these APIs should
       * be prepared to handle empty / missing information.  For example, a job's stage ids may be known
       * but the status API may not have any information about the details of those stages, so
       * `getStageInfo` could potentially return `None` for a valid stage id.
       *
       * To limit memory usage, these APIs only provide information on recent jobs / stages.  These APIs
       * will provide information for the last `spark.ui.retainedStages` stages and
       * `spark.ui.retainedJobs` jobs.
       *
       * NOTE: this class's constructor should be considered private and may be subject to change.
       */
    _statusTracker = new SparkStatusTracker(this)
```
接着是进度条，ui，hadoop conf，executor memory，心跳 等配置
``` shell
    // We need to register "HeartbeatReceiver" before "createTaskScheduler" because Executor will
    // retrieve "HeartbeatReceiver" in the constructor. (SPARK-6640)
    _heartbeatReceiver = env.rpcEnv.setupEndpoint(
        /**
         * Retrieve the [[RpcEndpointRef]] represented by `address` and `endpointName`.
         * This is a blocking action.
         * 注册heartbeatReceiver的Endpoint到rpcEnv上面并返回他对应的Reference
         */
      HeartbeatReceiver.ENDPOINT_NAME, new HeartbeatReceiver(this))
```
然后最重要的TaskScheduler & DAGScheduler
``` shell
    // Create and start the scheduler
    会根据master匹配对应的SchedulerBackend和TaskSchedulerImpl创建方式
    val (sched, ts) = SparkContext.createTaskScheduler(this, master, deployMode)
    // Create and start the scheduler
    _schedulerBackend = sched
    _taskScheduler = ts
    创建DAGScheduler
    _dagScheduler = new DAGScheduler(this)
    心跳
    _heartbeatReceiver.ask[Boolean](TaskSchedulerIsSet)

    // start TaskScheduler after taskScheduler sets DAGScheduler reference in DAGScheduler's
    // constructor
    启动TaskScheduler
    _taskScheduler.start()
    获取appid 不同模式不一样
    local模式为："local-" + System.currentTimeMillis
    _applicationId = _taskScheduler.applicationId()
    _applicationAttemptId = taskScheduler.applicationAttemptId()
    _conf.set("spark.app.id", _applicationId)
    if (_conf.getBoolean("spark.ui.reverseProxy", false)) {
      System.setProperty("spark.ui.proxyBase", "/proxy/" + _applicationId)
    }
    _ui.foreach(_.setAppId(_applicationId))
      /**
       * Initializes the BlockManager with the given appId. This is not performed in the constructor as
       * the appId may not be known at BlockManager instantiation time (in particular for the driver,
       * where it is only learned after registration with the TaskScheduler).
       *
       * This method initializes the BlockTransferService and ShuffleClient, registers with the
       * BlockManagerMaster, starts the BlockManagerWorker endpoint, and registers with a local shuffle
       * service if configured.
       */
    _env.blockManager.initialize(_applicationId)
```
接下来metrics system 测量系统，提供个ui展示
``` shell
    // The metrics system for Driver need to be set spark.app.id to app ID.
    // So it should start after we get app ID from the task scheduler and set spark.app.id.
    _env.metricsSystem.start()
    // Attach the driver metrics servlet handler to the web ui after the metrics system is started.
    _env.metricsSystem.getServletHandlers.foreach(handler => ui.foreach(_.attachHandler(handler)))

```
然后_eventLogger和动态资源分配模式
``` shell
    // Optionally scale number of executors dynamically based on workload. Exposed for testing.
    通过spark.dynamicAllocation.enabled参数开启后就会启动ExecutorAllocationManager
    val dynamicAllocationEnabled = Utils.isDynamicAllocationEnabled(_conf)
    _executorAllocationManager =
      if (dynamicAllocationEnabled) {
        schedulerBackend match {
          case b: ExecutorAllocationClient =>
            // An agent that dynamically allocates and removes executors based on the workload.
            根据集群资源动态触发增加或者删除资源策略
            Some(new ExecutorAllocationManager(
              schedulerBackend.asInstanceOf[ExecutorAllocationClient], listenerBus, _conf))
          case _ =>
            None
        }
      } else {
        None
      }
    _executorAllocationManager.foreach(_.start())

```
然后cleaner
``` shell
   _cleaner =
      if (_conf.getBoolean("spark.cleaner.referenceTracking", true)) {
      /**
       * An asynchronous cleaner for RDD, shuffle, and broadcast state.
       *
       * This maintains a weak reference for each RDD, ShuffleDependency, and Broadcast of interest,
       * to be processed when the associated object goes out of scope of the application. Actual
       * cleanup is performed in a separate daemon thread.
       */
        Some(new ContextCleaner(this))
      } else {
        None
      }
    _cleaner.foreach(_.start())
```
最后shutdown hook
``` shell
    // Make sure the context is stopped if the user forgets about it. This avoids leaving
    // unfinished event logs around after the JVM exits cleanly. It doesn't help if the JVM
    // is killed, though.
    logDebug("Adding shutdown hook") // force eager creation of logger
    // ShutdownHookManager相比JVM本身的执行Hook方式具有如下两种特性（默认JVM执行，无序，并发）
    // 1.顺序  2.有优先级
    _shutdownHookRef = ShutdownHookManager.addShutdownHook(
      ShutdownHookManager.SPARK_CONTEXT_SHUTDOWN_PRIORITY) { () =>
      logInfo("Invoking stop() from shutdown hook")
      stop()
    }
```
