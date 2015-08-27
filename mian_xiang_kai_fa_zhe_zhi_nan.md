开发者关心的是数据库的操作。本章分为以下几个内容：
1. 增删改查CRUD操作
2. 数据集合Aggregation函数使用
3. 利用索引加速查询，基于地理位置查询，文本搜索

CRUD操作
======
mongodb的文档结构如图：

![mongodb struct](http://docs.mongodb.org/manual/_images/crud-annotated-collection.png)

一个文档集，对应传统关系数据库的一个表；一个json元数据，对应关系数据库的一条记录。

#### **查询/读操作**

![query data](http://docs.mongodb.org/manual/_images/crud-query-stages.png)

	* 在mongodb中所有的查询只能指定一个数据集；
	* 可以对查询操作指定 limits, skips, sort和orders等操作；
	* 查询操作默认是无序的，除非指定 sort()；
	* 修改已经存在的记录（比如update操作），使用的检索语法和查询语句相同；
	* 在集合处理中，要使用 $match pipeline 方式来查询数据

mongodb提供了 findOne() 用于获取单条数据记录。

查询的核心方法是find()和findOne()函数；
> db.users.find( { age: { $gt: 18 } }, { name: 1, address: 1 } ).limit(5)

如上句所示，find函数接收两个json参数，第一个json是过滤条件，第二个json定义了需要检索的数据字段。第2个参数不传，则为默认取所有数据项。

查询的语法和技巧会在mongodb VS SQL 文档中具体对比介绍，此处暂略过〜

特别说明一点：

	db.inventory.find(
	   { quantity: { $gte: 100, $lte: 200 } }
	).explain("executionStats")

mongodb也有像SQL一样的查询性能预览功能。

#### **创建/写数据**

mongodb不需要预先创建数据集/数据库，当第一条记录输入时，自动创建数据集；当然也可以使用命令 db.createCollection("users") 创建空数据集。

##### 创建单条记录：
	
	db.inventory.insert(
	   {
	     item: "ABC1",
	     details: {
	        model: "14Q3",
	        manufacturer: "XYZ Company"
	     },
	     stock: [ { size: "S", qty: 25 }, { size: "M", qty: 50 } ],
	     category: "clothing"
	   }
	)
	
    返回结果: WriteResult({ "nInserted" : 1 })
	

##### 创建多条记录：

插入多条记录有两种方法：

1. 创建一个json列表data\_list，然后使用insert(data\_list);

		var mydocuments =
	    [
	      {
	        item: "ABC2",
	        details: { model: "14Q3", manufacturer: "M1 Corporation" },
	        stock: [ { size: "M", qty: 50 } ],
	        category: "clothing"
	      },
	      {
	        item: "MNO2",
	        details: { model: "14Q3", manufacturer: "ABC Company" },
	        stock: [ { size: "S", qty: 5 }, { size: "M", qty: 5 }, { size: "L", qty: 1 } ],
	        category: "clothing"
	      }];
	      
	    db.inventory.insert( mydocuments );

2. 使用bluk()函数，逐条记入然后一并提交bulk.execute();
		
		# 初始化一个bluk操作
		var bulk = db.inventory.initializeUnorderedBulkOp();
		# 逐条insert数据
		bulk.insert(
		   {
		     item: "BE10",
		     details: { model: "14Q2", manufacturer: "XYZ Company" },
		     stock: [ { size: "L", qty: 5 } ],
		     category: "clothing"
		   }
		);
		bulk.insert(
		   {
		     item: "ZYT1",
		     details: { model: "14Q1", manufacturer: "ABC Company"  },
		     stock: [ { size: "S", qty: 5 }, { size: "M", qty: 5 } ],
		     category: "houseware"
		   }
		);
		# 最后一并提交执行
		bulk.execute();

两种方法返回结果一样，均为：

	BulkWriteResult({
	   "writeErrors" : [ ],
	   "writeConcernErrors" : [ ],
	   "nInserted" : 2,
	   "nUpserted" : 0,
	   "nMatched" : 0,
	   "nModified" : 0,
	   "nRemoved" : 0,
	   "upserted" : [ ]
	})


#### **更新数据**
数据更新分为几种情况：

1. 单条数据更新

		db.inventory.update(
	    { item: "MNO2" },
	    {
	      $set: {
	        category: "apparel",
	        details: { model: "14Q3", manufacturer: "XYZ Company" }
	      },
	      $currentDate: { lastModified: true }
	    }
		)
		
		返回结果：WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
		
		默认情况下update()函数仅更新单条数据项；
		接收两个json参数: 
		第一个是过滤条件，第二个是更新操作。 
		$set 指定新数据值， $currentDate指定字段为当前时间。

2. 多文档批量更新

		db.inventory.update(
		   { category: "clothing" },
		   {
		     $set: { category: "apparel" },
		     $currentDate: { lastModified: true }
		   },
		   { multi: true }
		)
		
		返回结果：WriteResult({ "nMatched" : 3, "nUpserted" : 0, "nModified" : 3 })
		
		多文档批量更新，需要传第三个json参数:
		{ multi: true }

3. 内嵌文档更新

		db.inventory.update(
		  { item: "ABC1" },
		  { $set: { "details.model": "14Q2" } }
		)
		
		返回结果：WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
			
		内嵌文档更新，需要指定子级关系: details.model ！

4. 文档内容全量替换

		db.inventory.update(
		   { item: "BE10" },
		   {
		     item: "BE05",
		     stock: [ { size: "S", qty: 20 }, { size: "M", qty: 5 } ],
		     category: "apparel"
		   }
		)
		
		返回结果：WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
		
		不指定 $set操作。
		符合 item = "BE10"的整个文档项，将被替换为第二个json项的值。
		同样的，如果需要多文档替换，也需要指定 { multi: true }


5. 特殊使用upsert：更新操作项不存在则创建数据项

	经常遇到的一个使用场景是：如果数据存在则更新，不存在则创建数据项。
mongodb提供了upsert参数，方便执行此类操作。

		db.inventory.update(
		   { item: "TBD1" },
		   {
		     item: "TBD1",
		     details: { "model" : "14Q4", "manufacturer" : "ABC Company" },
		     stock: [ { "size" : "S", "qty" : 25 } ],
		     category: "houseware"
		   },
		   { upsert: true }
		)
		
	    # { upsert: true } 参数，指定 upsert模式。
	   
	   上面是全量更新，即替换数据项。也可以像下面这样添加 $set 操作，指定数据项更新！
	       
		db.inventory.update(
		   { item: "TBD2" },
		   {
		     $set: {
		        details: { "model" : "14Q3", "manufacturer" : "IJK Co." },
		        category: "houseware"
		     }
		   },
		   { upsert: true }
		)
		

#### **删除数据**

1. 删除整个文档 

	> db.inventory.remove({})

2. 删除匹配项
	
	> db.inventory.remove( { type : "food" } )

3. 删除匹配中的第一项

	> db.inventory.remove( { type : "food" }, 1 )
	>
	> 删除remove和update相反，update默认是更新单项，需要指定全部；
	而删除是默认删除全部匹配项，需要指定单项。

