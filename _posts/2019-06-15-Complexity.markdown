---
layout: post
title:  "Complexity"
date:   2019-06-15 17:00:00
categories: "Data_Structures_and_Algorithms"
tags: "Data Structures and Algorithms"
---
#### 复杂度分析定义
* 复杂度分析包含时间复杂度和空间复杂度，描述的是算法执行时间（或占用空间）与数据规模的增长关系
* 时间复杂度的全称是渐进时间复杂度，表示算法的执行时间与数据规模之间的增长关系
* 空间复杂度全称就是渐进空间复杂度（asymptotic space complexity），表示算法的存储空间与数据规模之间的增长关系
* 数据结构和算法解决是“如何让计算机更快时间、更省空间的解决问题”，因此需从执行时间和占用空间两个维度来评估数据结构和算法的性能。

#### 为什么要进行复杂度分析
1. 和性能测试相比，复杂度分析有不依赖执行环境、成本低、效率高、易操作、指导性强的特点
2. 复杂度分析能帮助编写出性能更优的代码，有利于降低系统开发和维护成本

#### 如何进行复杂度分析
1. 大O复杂度表示法
  + 来源
    + 算法的执行时间与每行代码的执行次数成正比，用T(n) = O(f(n))表示，其中T(n)表示算法执行总时间，f(n)表示每行代码执行总次数，而n往往表示数据的规模
  + 特点
    + 以时间复杂度为例，由于时间复杂度描述的是算法执行时间与数据规模的增长变化趋势，所以常量阶、低阶以及系数实际上对这种增长趋势不产决定性影响，所以在做时间复杂度分析时忽略这些项
2. 复杂度分析法则
  + 单段代码看高频：比如循环
  + 多段代码取最大：比如一段代码中有单循环和多重循环，那么取多重循环的复杂度
  + 嵌套代码求乘积：比如递归、多重循环等
  + 多个规模求加法：比如方法有两个参数控制两个循环的次数，那么这时就取二者复杂度相加

#### 常用的复杂度级别
* 多项式阶：随着数据规模的增长，算法的执行时间和空间占用，按照多项式的比例增长。
  + O(1)（常数阶）
    + O(1) 只是常量级时间复杂度的一种表示方法，并不是指只执行了一行代码。
    + 一般情况下，只要算法中不存在循环语句、递归语句，即使有成千上万行的代码，其时间复杂度也是Ο(1)
  + O(n)（线性阶）
    ``` java
    public void print(int n) {
      int i = 0;
      int[] a = new int[n];
      for (i; i <n; ++i) {
        a[i] = i * i;
      }
      for (i = n-1; i >= 0; --i) {
        System.out.println(a[i])
      }
    }
    ```
    跟时间复杂度分析一样，我们可以看到，第 2行代码中，我们申请了一个空间存储变量 i，但是它是常量阶的，跟数据规模n 没有关系，所以我们可以忽略。第 3行申请了一个大小为 n的   int类型数组，除此之外，剩下的代码都没有占用更多的空间，所以整段代码的空间复杂度就是O(n)
  + O(logn)（对数阶）
    ``` java
      i = 1;
      while(i <= n) {
        i = i * 2
      }
    ```
    从代码中可以看出，变量i的值从1开始取，每循环一次就乘以2。当大于n时，循环结束。实际上，变量i的取值就是一个等比数列。所以，我们只要知道x值是多少，就知道这行代码执行的次数了。x=logn，所以，这段代码的时间复杂度就是O(logn)。
  + O(nlogn)（线性对数阶）
  + O(n^2)（平方阶）
  + O(n^3)（立方阶）
* 非多项式阶：随着数据规模的增长，算法的执行时间和空间占用暴增，这类算法性能极差。
  + O(2^n)（指数阶）
  + O(n!)（阶乘阶）

#### 复杂度分析的4个概念
1. 最坏情况时间复杂度：代码在最理想情况下执行的时间复杂度
2. 最好情况时间复杂度：代码在最坏情况下执行的时间复杂度
3. 平均时间复杂度：用代码在所有情况下执行的次数的加权平均值表示
4. 均摊时间复杂度：在代码执行的所有复杂度情况中绝大部分是低级别的复杂度，个别情况是高级别复杂度且发生具有时序关系时，可以将个别高级别复杂度均摊到低级别复杂度上。基本上均摊结果就等于低级别复杂度

#### 为什么要引入这4个概念
1. 同一段代码在不同情况下时间复杂度会出现量级差异，为了更全面，更准确的描述代码的时间复杂度，所以引入这4个概念
2. 代码复杂度在不同情况下出现量级差别时才需要区别这四种复杂度。大多数情况下，是不需要区别分析它们的。

#### 如何分析平均、均摊时间复杂度
1. 平均时间复杂度代码在不同情况下复杂度出现量级差别，则用代码所有可能情况下执行次数的加权平均值表示
2.均摊时间复杂度两个条件满足时使用：
  + 代码在绝大多数情况下是低级别复杂度，只有极少数情况是高级别复杂度
  + 低级别和高级别复杂度出现具有时序规律。均摊结果一般都等于低级别复杂度

#### 如何掌握好复杂度分析方法
复杂度分析关键在于多练，所谓孰能生巧。
