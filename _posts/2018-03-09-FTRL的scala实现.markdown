---
layout: post
title:  "FTRL的scala实现"
date:   2018-03-09 15:06:00
categories: Spark
tags: Spark
---
#### FTRL的scala实现
``` scala 
import breeze.linalg.{DenseVector, _}
import breeze.numerics._
import org.apache.spark.sql.{Dataset, SparkSession}
import org.apache.log4j.Logger

/**
  * Created by nickliu on 11/29/2017.
  */
object LR{

  val logger = Logger.getLogger(LR.getClass.getName)

  def fn(w: DenseVector[Double], x: DenseVector[Double]): Double ={
    /** 决策函数为sigmoid函数
      * exp 对矩阵a中每个元素取指数函数
      * dot 按照矩阵乘法的规则来运算的 Breeze a.t * b => python dot(a,b)
      */
    val sigmoid = 1.0 / (1.0 + exp(-w.t * x))
    return sigmoid
  }

  def loss(y: Double, y_hat: Double): Double ={
    /** 交叉熵损失函数 */
    return sum(DenseVector[Double](-y * log(y_hat) - (1 - y) * log(1 - y_hat)))
  }

  def grad(y: Double, y_hat: Double, x: DenseVector[Double]): DenseVector[Double] ={
    /** 交叉熵损失函数对权重w的一阶导数 */
    return (y_hat - y) * x
  }

}

object FTLR{

  val dim = 4
  var l1: Double = 0.0
  var l2: Double = 0.0
  var alpha: Double = 0.5
  var beta: Double = 1.0
  var z, n, w = DenseVector.zeros[Double](dim)//z更新权重用到的,存放梯度累加的n，最后的w

  private def predict(x: DenseVector[Double]): Double ={
    return LR.fn(w, x)
  }

  var correct = 0
  var wrong = 0
  def transform(dataSet: Dataset[String]): Double ={
    val dataList = dataSet.rdd.map({
      line =>
        val row = line.split("\\s+")
        (row(0).toDouble, row(1).toDouble, row(2).toDouble, row(3).toDouble, row(4).toDouble)
    }).collect().toList
    for (line <- dataList) {
      val predictValue = predict(DenseVector(line._1.toDouble, line._2.toDouble, line._3.toDouble, line._4.toDouble))
      val y_hat = if(predictValue > 0.5) 1.0 else 0.0
      val y = line._5.toDouble
      if(y == y_hat) correct += 1 else wrong += 1
    }
    val precision = 1.0 * correct / (correct + wrong)
    return precision
  }

  def update(x: DenseVector[Double], y: Double): Double ={
    for(i <- 0 until dim){
      var y: Double = 0
      if(abs(z(i)) > l1){
        /**
          * Breeze signum(a) => Numpy sign(a)
          * 大于0的返回1.0
          * 小于0的返回-1.0
          * 等于0的返回0.0
          *
          * Numpy sqrt()
          * 计算各元素的平方根
          */
        y = (signum(z(i)) * l1 - z(i)) / (l2 + (beta + sqrt(n(i))) / alpha)
      }
      w(i) = y
    }
    val y_hat = predict(x)
    val g = LR.grad(y, y_hat, x)
    val sigma = (sqrt(n + g * g) - sqrt(n)) / alpha
    z += g - sigma * w
    n += g * g
    return LR.loss(y, y_hat)
  }

  def fit(dataSet: Dataset[String], max_itr: Int = 100000000, eta: Double = 0.01, epochs: Int = 100): DenseVector[Double] ={
    var itr = 0
    var n = 0
    val dataList = dataSet.rdd.map({
      line =>
        val row = line.split("\\s+")
        (row(0).toDouble, row(1).toDouble, row(2).toDouble, row(3).toDouble, row(4).toDouble)
    }).collect().toList

    while(true) {
      for (line <- dataList) {
        val loss = update(DenseVector(line._1, line._2, line._3, line._4), line._5)
        if (loss < eta) {
          itr += 1
        } else {
          itr = 0
        }
        if (itr >= epochs) {
          // 损失函数已连续epochs次迭代小于eta
          println("loss have less than", eta, " continuously for ", itr, "iterations")
          return w
        } else {
          n += 1
          if (n >= max_itr) {
            println("reach max iteration", max_itr)
            return w
          }
        }
      }
    }
    w
  }

}

object Application{
  val warehouseLocation = "hdfs://nameservice1/user/hive/warehouse"

  val spark = SparkSession
    .builder
    .master("local")
    .appName("ALSExample")
    .config("spark.sql.warehouse.dir",warehouseLocation)
    .config("spark.serializer", "org.apache.spark.serializer.KryoSerializer")
    .enableHiveSupport()
    .getOrCreate()

  val d = 4
  def main(args: Array[String]): Unit = {
    val data = spark.read.textFile("file:///D:/git_project/phoenix/data/mllib/train.txt").cache()
    val Array(trainData, testData) = data.randomSplit(Array(0.7,0.3))
    val ftlr = FTLR
    val startTime = System.currentTimeMillis
    val w = ftlr.fit(trainData, 100000, 0.01, 100)
    println(w)

    val precision = ftlr.transform(testData)
    val currentTime = System.currentTimeMillis
    val minutes = (currentTime - startTime).toDouble / 60000
    println("precision -> ", precision)
    println("minutes -> ", minutes)
  }
}
``` 
