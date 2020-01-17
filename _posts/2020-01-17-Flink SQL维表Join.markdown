---
layout: post
title:  "Flink SQL维表Join"
date:   2020-01-17 12:00:00
categories: Flink
tags: Flink
---
#### 背景
> 在流计算中我们会经常需要涉及数据流补齐字段。因为一条流的数据比较单一，维度有限，而在使用的时候需要更多地维度，就要先将所需的维度信息补全。比如某条流里是交易日志中商品id，但是在做业务时需要根据店铺维度或者行业纬度进行聚合，这就需要先将交易日志与商品维表进行关联，补全所需的维度信息。这里所说的维表与数据仓库中的概念类似，是维度属性的集合，比如商品维，地点维，用户维等等。

#### 维表JOIN语法
> 由于维表是一张不断变化的表（静态表只是动态表的一种特例）。如果用传统的JOIN语法SELECT * FROM T JOIN dim_table on T.id = dim_table.id
> 来表达维表JOIN是不完整的。因为维表是一直在更新变化的，这个语法关联上的哪个时刻的维表是确定的。所以Flink SQL的维表JOIN语法引入了SQL:2011 
> Temporal Table的标准语法，用来声明关联的是维表哪个时刻的快照。维表 JOIN 语法/示例如下。

> 假设我们有一个Orders订单数据流，希望根据产品ID补全流上的产品维度信息，所以需要跟Products维度表进行关联。Orders和Products的DDL声明语句如下：
``` sql
CREATE TABLE Orders (
  orderId VARCHAR,          -- 订单 id
  productId VARCHAR,        -- 产品 id
  units INT,                -- 购买数量
  orderTime TIMESTAMP       -- 下单时间
) with (
  'connector.type' = 'kafka',  -- kafka数据源
  'connector.version' = '0.11',
  'connector.topic' = 'topic_name',
)

CREATE TABLE Products (
  productId VARCHAR,        -- 产品 id
  name VARCHAR,             -- 产品名称
  unitPrice DOUBLE          -- 单价
  PERIOD FOR SYSTEM_TIME,   -- 这是一张随系统时间而变化的表，用来声明维表
  PRIMARY KEY (productId)   -- 维表必须声明主键
) with (
  'connector.type' = 'hbase', -- required: specify this table type is hbase
  'connector.version' = '1.4.3',          -- required: valid connector versions are "1.4.3"
  'connector.table-name' = 'hbase_table_name', 
  ...
)
```

#### JOIN当前维表
``` sql
SELECT *
FROM Orders AS o
[LEFT] JOIN Products FOR SYSTEM_TIME AS OF PROCTIME() AS p
ON o.productId = p.productId
```
> Flink SQL支持LEFT JOIN和INNER JOIN的维表关联。如上语法所示的，维表JOIN语法与传统的JOIN语法并无二异。
> 只是Products维表后面需要跟上FOR SYSTEM_TIME AS OF PROCTIME()的关键字，其含义是每条到达的数据所关联上的是到达时刻的维表快照，
> 也就是说，当数据到达时，我们会根据数据上的key去查询远程数据库，拿到匹配的结果后关联输出。这里的PROCTIME即processing time。
> 使用JOIN当前维表功能需要注意的是，如果维表插入了一条数据能匹配上之前左表的数据时，JOIN的结果流，不会发出更新的数据以弥补之前的未匹配。
> JOIN行为只发生在处理时间（processing time），即使维表中的数据都被删了，之前JOIN流已经发出的关联上的数据也不会被撤回或改变。

#### JOIN历史维表
``` sql
SELECT *
FROM Orders AS o
[LEFT] JOIN Products FOR SYSTEM_TIME AS OF o.orderTime AS p
ON o.productId = p.productId
```
> 有时候想关联上的维度数据，并不是当前时刻的值，而是某个历史时刻的值。比如，产品的价格一直在发生变化，订单流希望补全的是下单时的价格，
> 而不是当前的价格，那就是JOIN历史维表。语法上只需要将上文的PROCTIME()改成o.orderTime即可。含义是关联上的是下单时刻的Products维表。
> Flink在获取维度数据时，会根据左流的时间去查对应时刻的快照数据。因此JOIN 历史维表需要外部存储支持多版本存储，如HBase，或者存储的数据中带有多版本信息。
