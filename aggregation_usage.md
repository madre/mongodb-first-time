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


### 使用聚合操作邮政编码

在网页 http://media.mongodb.org/zips.json 中有一系列的邮编数据，你可以使用 mongoimport 将相关的json数据导入你的mongodb数据实例中。［mongoimport在后面章节有介绍］

#### 数据实例

每一列的邮编数据如下所示：

	{
	  "_id": "10280",
	  "city": "NEW YORK",
	  "state": "NY",
	  "pop": 5574,
	  "loc": [
	    -74.016323,
	    40.710537
	  ]
	}
	
在接下一来的一系列示例中，我们将核心使用 aggregate()方法，聚合操作计算数据。 aggregate方法 使用聚合管道处理文档得到统计结果。

### 返回所有人口(pop)值超过1千万的州

> db.zipcodes.aggregate( [
>
>    { $group: { _id: "$state", totalPop: { $sum: "$pop" } } },
>
>    { $match: { totalPop: { $gte: 10*1000*1000 } } }
>
> ] )

上面的mongo shell命令由 $group 和 $ match 组成而成。

* $group 阶段将zipcode文档以state字段进行分组，计算每个state字段的 totalPop值，并且为每一个唯一的state输出一个文档。
* 新生成的 state 文档有两个字段: _id, totalPop。 _id字段包含着state之前的值，也是分组的字段。 totalPop字段是计算后得出的字段，包含了每个state对应的总人口。 为了计算结果， $group 使用了 $sum 操作来将不同的state对应的值叠加在一起。
* 在 $group 阶段之后, 在管道中的文档集类似于下面的结构:

	>{"_id" : "AK", "totalPop" : 550043}

* $match 阶段过滤分组后的文档集，输出那些totalPop值大于等于1千万的文档。$match 阶段并不会改变匹配的文档，仅仅是输出匹配文档。 

上面的 mongo shell 命令等同于下面的SQL操作:

	SELECT state, SUM(pop) AS totalPop
	FROM zipcodes
	GROUP BY state
	HAVING totalPop >= (10*1000*1000)

#### 各州城市人口平均值

下面的mongo shell命令，将会返回各州城市人口平均值：
	
	db.zipcodes.aggregate( [
	   { $group: { _id: { state: "$state", city: "$city" }, pop: { $sum: "$pop" } } },
	   { $group: { _id: "$_id.state", avgCityPop: { $avg: "$pop" } } }
	] )

* 第一个 $group 阶段， 将文档按city和state组合分组，使用 $sum 计算出每一个组合的城市人口，然后输出每个 城市和州 的组合数据。经过这阶段的处理，将得到如下结构的数据:

		{
		  "_id" : {
		    "state" : "CO",
		    "city" : "EDGEWATER"
		  },
		  "pop" : 13154
		}
		
* 第二阶段的 $group 将上面得到的文档按 _id.state 字段再进行分组，然后使用 $avg 表达式计算出各州每个城市的平均人口中，然后输出各州的文档。 得到的文档如下所示:

		{
		  "_id" : "MN",
		  "avgCityPop" : 5335
		}

#### 统计各州人口最大和最小的城市

下面的命令有点长，用来统计各州人口最大和最小的城市:

	db.zipcodes.aggregate( [
	   { $group:
	      {
	        _id: { state: "$state", city: "$city" },
	        pop: { $sum: "$pop" }
	      }
	   },
	   { $sort: { pop: 1 } },
	   { $group:
	      {
	        _id : "$_id.state",
	        biggestCity:  { $last: "$_id.city" },
	        biggestPop:   { $last: "$pop" },
	        smallestCity: { $first: "$_id.city" },
	        smallestPop:  { $first: "$pop" }
	      }
	   },
	
	  // the following $project is optional, and
	  // modifies the output format.
	
	  { $project:
	    { _id: 0,
	      state: "$_id",
	      biggestCity:  { name: "$biggestCity",  pop: "$biggestPop" },
	      smallestCity: { name: "$smallestCity", pop: "$smallestPop" }
	    }
	  }
	] )


* 第一个 $group 阶段， 将文档按city和state组合分组，使用 $sum 计算出每一个组合的城市人口，然后输出每个 城市和州 的组合数据。经过这阶段的处理，将得到如下结构的数据:

		{
		  "_id" : {
		    "state" : "CO",
		    "city" : "EDGEWATER"
		  },
		  "pop" : 13154
		}

* $sort 阶段把管道中的文档按pop字段的值从小到大进行排序，这个操作不会改变文档数据。
* 之后的 $group 阶段把新排序后的文档按 _id.state字段进行分组，并且输出每个state 对应的文档。这个阶段也计算了每个state对应的最大最小值。使用 $last 表达式，$group 操作创建 biggestCity和biggestPop 两个字段并存储城市名和城市的最大人口数。 使用 $first 表达式，$group 操作创建 smallestCity和smallestPop 两个字段并存储城市名和城市的最小人口数。 结果输出如下:

		{
		  "_id" : "WA",
		  "biggestCity" : "SEATTLE",
		  "biggestPop" : 520096,
		  "smallestCity" : "BENGE",
		  "smallestPop" : 2
		}

* 最后的 $project 阶段重命名了 _id 字段为 state，把 biggestCity, biggestPop, smallestCity, smallestPop, biggestCity和smallestCity 几个字段存入嵌套文档内。

最终输出结果如下：
	
	{
	  "state" : "RI",
	  "biggestCity" : {
	    "name" : "CRANSTON",
	    "pop" : 176404
	  },
	  "smallestCity" : {
	    "name" : "CLAYVILLE",
	    "pop" : 45
	  }
	}


### 用户偏爱数据介绍方法 project, sort, group, unwind, limit

上面的邮编数据统计给出了一些数据的基本示例，下面会再给出一些更复杂的用法示例：

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

执行下面的命令，能够让所有的用户名字转为大写并且以字母顺序重排序。

	db.users.aggregate(
	  [
	    { $project : { name:{$toUpper:"$_id"} , _id:0 } },
	    { $sort : { name : 1 } }
	  ]
	)

* 操作 $project:
	* 创建了一个新字段 name
	* 把字段 _id 的值使用 $toUpper操作 转为大写，然后$project创建了一个新字段命令为name存储了该值
	* 隐藏 _id 字段，默认情况下 $project将会传到 _id字段，除非明确的隐藏字段
* 操作 $sort 按名字的字母顺序重新排序了users数据

上面命令执行后的输出结果为:

	{
	  "name" : "JANE"
	},
	{
	  "name" : "JILL"
	},
	{
	  "name" : "JOE"
	}

### 按加入月份顺序返回用户名

下面的聚合操作命令能够按加入的月份顺序返回用户名，这种聚合的操作常常用来辅助生成会员续费提醒。
	
	db.users.aggregate(
	  [
	    { $project :
	       {
	         month_joined : { $month : "$joined" },
	         name : "$_id",
	         _id : 0
	       }
	    },
	    { $sort : { month_joined : 1 } }
	  ]
	)

上面的管道将所有的在users集合中的文档传递给下面的操作：

* $project 操作:
	* 生成两个新字段: month_joined 和 name
	* 从结果中隐藏id字段，id字段必须明文标识才能隐藏
* $month 操作将 joined 字段的值转换为月份数字，然后$project 操作将这个月份数据值分配给 moth_joined字段。
* $sort 操作把结果按照 month_joined 字段重新排序

上面的操作返回的结果如下：

> { "month_joined" : 3, "name" : "jane" }
>
> { "month_joined" : 7, "name" : "joe" }

### 返回每个月加入的总数

下面的命令能够显示出每年中每个月有多少人入会。你可以会用这个聚合数据去调整招募和市场策略。

	db.users.aggregate(
	  [
	    { $project : { month_joined : { $month : "$joined" } } } ,
	    { $group : { _id : {month_joined:"$month_joined"} , number : { $sum : 1 } } },
	    { $sort : { "_id.month_joined" : 1 } }
	  ]
	)

上面的管道将所有的在users集合中的文档传递给下面的操作：

* $project 操作创建 month_joined 新字段
* $month 操作将 joined 字段的值转换为月份数字，然后$project 操作将这个月份数据值分配给 moth_joined字段。
* $group 操作按给出的month_joined值整理文档，统计出有各month_joined值有多少项文档。特别指出的是，对于每一个唯一的值，$group 操作创建了一个新的 per-month 文档包含以下两个字段:
	* \_id, 包括一个带有 month_joined的嵌套的文档
	* number, 一个生成字段，每有一个文档包含给出的 month_joined字段值，$sum 操作就会将number 自增1。
* $sort 操作从 $group生成的文档中按 month_joined 的内容进行排序

上面的操作返回的结果如下：

	{
	  "_id" : {
	    "month_joined" : 1
	  },
	  "number" : 3
	},
	{
	  "_id" : {
	    "month_joined" : 2
	  },
	  "number" : 9
	},
	{
	  "_id" : {
	    "month_joined" : 3
	  },
	  "number" : 5
	}

### 返回五项最受欢迎的

下面的聚合操作将会整理所有数据中最受欢迎的前五项活动。 这种类型的分析能够帮助确认计划和未来的发展。

	db.users.aggregate(
	  [
	    { $unwind : "$likes" },
	    { $group : { _id : "$likes" , number : { $sum : 1 } } },
	    { $sort : { number : -1 } },
	    { $limit : 5 }
	  ]
	)

上面的聚合管道将所有的在users集合中的文档传递给下面的操作：

* $unwind 操作将所有的值拆开成列表，并且为第一个在列表中的项创建一个新版本的文档源

	> 举例来说
	给出下例中的一个 users 文档集：
	
		{
		  _id : "jane",
		  joined : ISODate("2011-03-02"),
		  likes : ["golf", "racquetball"]
		}
		
	> $unwind 操作 db.users.aggregate({$unwind : "$likes"}) 将会创建如下的新文档集：
		
		{
		  _id : "jane",
		  joined : ISODate("2011-03-02"),
		  likes : "golf"
		}
		{
		  _id : "jane",
		  joined : ISODate("2011-03-02"),
		  likes : "racquetball"
		}

* $group 操作按给出的month_joined值整理文档，统计出有各month_joined值有多少项文档。特别指出的是，对于每一个唯一的值，$group 操作创建了一个新的 per-month 文档包含以下两个字段:
	* \_id, 包括一个带有 month_joined的嵌套的文档
	* number, 一个生成字段，每有一个文档包含给出的 month_joined字段值，$sum 操作就会将number 自增1。
* $sort 操作从 $group生成的文档中按 month_joined 的内容进行排序
* $limit 操作将仅显示出前 5 项结果

上面的操作返回的结果如下：

	{
	  "_id" : "golf",
	  "number" : 33
	},
	{
	  "_id" : "racquetball",
	  "number" : 31
	},
	{
	  "_id" : "swimming",
	  "number" : 24
	},
	{
	  "_id" : "handball",
	  "number" : 19
	},
	{
	  "_id" : "tennis",
	  "number" : 18
	}


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

### 结果大小限制
在mongodb 2.6版之前，结果的数据集不能超过 16M；但在 2.6版之后可以返回任意大小的数据结果。

### 内存使用限制
在mongodb2.6之后，聚集管道每个阶段能够使用的最大内存是 100M，如果超过这个使用量，会直接报错。
如果需要处理更大的数据集，可以开启 allowDiskUse， 在硬盘中建立临时文件处理数据。
