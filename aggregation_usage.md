mongodb的集合计算从2.2版开始引进了新的框架结构，目前版本已经到3.0.7，旧的集合结构不再这里说明。
集合计算分为两种类型：

1. 聚合管道操作 (aggregation pipeline)
> 聚合管道，是仿照数据管道处理的概念实现的，能够组合多种数据操作，执行多重聚合任务得到最终结果

2. 单一目的聚合操作 (Single Purpose Aggregation)
> 单一目的聚合操作，一般指db.records.count() 这样直接对数据集进行操作，不需要联合查询、过滤、组合等其他数据操作

## 聚合管道 (Aggregation Pipeline)
Mongodb的聚合操作是有一组分步骤的操作组成，每个步骤会改变数据然后传递给下一个管道，但并不是每个管道操作都会对每一组数据产生一个文档输出的，比如一些管道操作只是过滤数据。

> db.collection.aggregate( [ { \<stage> }, ... ] )

在mongo shell中，MongoDB 提供了方法 db.collection.aggregate()去执行聚合操作，集合计算的方法/步骤包括以下几种：

1. Grouping
2. Matching/Filtering
3. projection
4. sorting
5. unwind
6. limit
7. skip
8. Out
9. Redact
10. geoNear
11. MapReduce

其中第10项geoNear需要和地理位置索引一起说明，之后的章节会单独讲解；
第11项mapreduce相对复杂独立成章，会在后面详细说明。

其他几部分会在本章一一说明。

### 管道表达式
在进一步说明各聚合计算方法之前，我们先要说明清楚，管道能够执行的表达式。一些管道操作方法是需要接收管理表达式作为操作符，这些表达式主要用于描述输入的数据转换的过程。

需要注意的是，管道内的表达式只能操作数据集本身，不能从其他数据集中联合查询。

一般来说这些表达式都是无状态只能被聚合操作执行计算，但有一个例外:  accumulator 表达式（sum, avg, last等累加计数的表达式）。这些计数操作，一般和group方法一起使用。

### 用户偏爱数据介绍方法 project, sort, group, unwind, limit

#### 数据模型
假设我们有一组运动俱乐部的用户偏爱数据users，包括用户的加入时间和偏爱的运动项目。大致的数据结构如下：

	{
	  _id : "jane",
	  joined : ISODate("2011-03-02"),
	  likes : ["golf", "racquetball"]
	}
	{
	  _id : "joe",
	  joined : ISODate("2012-07-02"),
	  likes : ["tennis", "golf", "swimming"]
	}

### 规范化和重排数据



db.users.aggregate(
  [
    { $project : { name:{$toUpper:"$_id"} , _id:0 } },
    { $sort : { name : 1 } }
  ]
)


## 单一目的聚合操作 (Single Purpose Aggregation)
单一目的的聚合操作种类少，只是将几种应用在一个数据值的常用聚合操作直接并入数据集的属性下方便使用。mongodb提供的一组在一些在一组数据中执行特定聚合操作的方法。包括：

### Count

使用count能够返回的文档中与查询条件相匹配的数量。

下列有一组数据，在mongo shell命令中使用 db.records.insert({ a: 1, b: 0 }) 一一输入数据集records中；

	{ a: 1, b: 0 }
	{ a: 1, b: 1 }
	{ a: 1, b: 4 }
	{ a: 2, b: 2 }

使用命令 db.records.count(), 能够计算出当时数据集records中有 4 个数据项；

使用命令 db.records.count( { a: 1 } ) 过滤 a值为1 的数据项，可以统计出存在3项数据。

### Distinct

distinct操作能够从查询出的一系列文档集中，按某数据据返回数据不重复的数据集合。

依旧按刚刚的records数据集为例，我们数据集中有以下数据项：

	{ a: 1, b: 0 }
	{ a: 1, b: 1 }
	{ a: 1, b: 1 }
	{ a: 1, b: 4 }
	{ a: 2, b: 2 }
	{ a: 2, b: 2 }

在 mongo shell命令行中，执行db.records.distinct( "b" ) 返回结果不重复的数据项b值如下：

> [ 0, 1, 4, 2 ]

### Group

group操作按查询条件取出一组数据，然后将它们按照某字段或某值进行分组统计。

我们仍以records为例，将之前的数据清理掉 db.records.remove({})，重新一一输入以下数据（db.records.insert({ a: 1, count: 4 })）：

	{ a: 1, count: 4 }
	{ a: 1, count: 2 }
	{ a: 1, count: 4 }
	{ a: 2, count: 3 }
	{ a: 2, count: 1 }
	{ a: 1, count: 5 }
	{ a: 4, count: 4 }

执行下面的命令，以字段a为分组，查询a的值小于3的数据项，计算count的总和。

	db.records.group( {
	   key: { a: 1 },
	   cond: { a: { $lt: 3 } },
	   reduce: function(cur, result) { result.count += cur.count },
	   initial: { count: 0 }
	} )

得到结果: 

> [
  { a: 1, count: 15 },
  { a: 2, count: 4 }
]

## 聚合管理的限制 (Aggregation Pipeline Limits)
