---
layout: post
title:  "SparkSubmit"
date:   2017-08-21 21:40:00
categories: Spark
tags: Spark
---
当我们按照官网的介绍，执行
``` shell
export HADOOP_CONF_DIR=XXX
./bin/spark-submit \
  --class org.apache.spark.examples.SparkPi \
  --master yarn \
  --deploy-mode cluster \  # can be client for client mode
  --executor-memory 20G \
  --num-executors 50 \
  /path/to/examples.jar \
  1000
```
时，Spark内部是如何提交这个job的呢？   
那就看看SparkSubmit.scala做了什么    
``` shell
  override def main(args: Array[String]): Unit = {
    val appArgs = new SparkSubmitArguments(args)
    if (appArgs.verbose) {
      // scalastyle:off println
      printStream.println(appArgs)
      // scalastyle:on println
    }
    // 根据传入的参数匹配对应的执行方法
    appArgs.action match {
      /**
       * Submit the application using the provided parameters.
       *
       * This runs in two steps. First, we prepare the launch environment by setting up
       * the appropriate classpath, system properties, and application arguments for
       * running the child main class based on the cluster manager and the deploy mode.
       * Second, we use this launch environment to invoke the main method of the child
       * main class.
       * 二步：prepareSubmitEnvironment 和 doRunMain
       */
      case SparkSubmitAction.SUBMIT => submit(appArgs)
      // Kill an existing submission using the REST protocol. Standalone and Mesos cluster mode only.
      case SparkSubmitAction.KILL => kill(appArgs) 
      // Request the status of an existing submission using the REST protocol.
      // Standalone and Mesos cluster mode only.
      case SparkSubmitAction.REQUEST_STATUS => requestStatus(appArgs) 
    }
  }
```
然后看看submit方法详细内容：
``` shell
 private def submit(args: SparkSubmitArguments): Unit = {
    /**
     * Prepare the environment for submitting an application.
     * This returns a 4-tuple:
     *   (1) the arguments for the child process,
     *   (2) a list of classpath entries for the child,
     *   (3) a map of system properties, and
     *   (4) the main class for the child
     * Exposed for testing.
     * 前面提交job脚本里面的master，deploy-mode等参数 全在这个方法里面会触发不同的执行操作
     */
    val (childArgs, childClasspath, sysProps, childMainClass) = prepareSubmitEnvironment(args)

    def doRunMain(): Unit = {
      if (args.proxyUser != null) {
        val proxyUser = UserGroupInformation.createProxyUser(args.proxyUser,
          UserGroupInformation.getCurrentUser())
        try {
          proxyUser.doAs(new PrivilegedExceptionAction[Unit]() {
            override def run(): Unit = {
              /**
               * Run the main method of the child class using the provided launch environment.
               *
               * Note that this main class will not be the one provided by the user if we're
               * running cluster deploy mode or python applications.
               */
              runMain(childArgs, childClasspath, sysProps, childMainClass, args.verbose)
            }
          })
        } catch {
          case e: Exception =>
            // Hadoop's AuthorizationException suppresses the exception's stack trace, which
            // makes the message printed to the output by the JVM not very helpful. Instead,
            // detect exceptions with empty stack traces here, and treat them differently.
            if (e.getStackTrace().length == 0) {
              // scalastyle:off println
              printStream.println(s"ERROR: ${e.getClass().getName()}: ${e.getMessage()}")
              // scalastyle:on println
              exitFn(1)
            } else {
              throw e
            }
        }
      } else {
          /**
           * Run the main method of the child class using the provided launch environment.
           *
           * Note that this main class will not be the one provided by the user if we're
           * running cluster deploy mode or python applications.
           */
        runMain(childArgs, childClasspath, sysProps, childMainClass, args.verbose)
      }
    }
```
其中prepareSubmitEnvironment最重要的代码：
``` shell
    if (deployMode == CLIENT || isYarnCluster) {
      childMainClass = args.mainClass
      ...
    }
    
    if (args.isStandaloneCluster) {
      if (args.useRest) {
        childMainClass = "org.apache.spark.deploy.rest.RestSubmissionClient"
        ...
      } else {
        // In legacy standalone cluster mode, use Client as a wrapper around the user class
        childMainClass = "org.apache.spark.deploy.Client"
        ...
      }
      ...
    }
    
    if (isYarnCluster) {
      childMainClass = "org.apache.spark.deploy.yarn.Client"
      ...
    }
      
    if (isMesosCluster) {
      childMainClass = "org.apache.spark.deploy.rest.RestSubmissionClient"
      ...
    }
```
runMain所需的参数就是prepareSubmitEnvironment的返回值
``` shell
  // runMain里面通过java的反射得到mainClass
  mainClass = Utils.classForName(childMainClass)
  // 得到main方法
  val mainMethod = mainClass.getMethod("main", new Array[String](0).getClass)
  // 执行main方法
  mainMethod.invoke(null, childArgs.toArray)
```
