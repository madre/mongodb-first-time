# SQL和 Aggregation 对照表

MongoDB的聚合管理操作有很多都能够和SQL的聚合操作相对应。
下面给出了一个大致的表格说明了SQL中的聚合方法 和 MongoDB中的对照关系:


| SQL术语、函数和概念 | MongoDB聚合操作 |
| -- | -- |
| WHERE | $match |
| GROUP BY | $group |
| HAVING | $match |
| SELECT | $project |
| ORDER BY | $sort |
| LIMIT | $limit |
| SUM() | $sum |
| COUNT() | $sum |
| join | 没有直接对应的操作； $unwind操作在一些情况下功能似类，但是会把数据变成嵌套的文档。 |


## 具体示例

下面的表格展示了一个mongodb和SQL语句之间的快速的参照系。
表中的所有的示例都假设符合以下条件：

* SQL 示例假设存在两个表，orders 和 order_lines。orders.id和 order_lineitem.order_id 可以执行join操作。
* MongoDB 示例假设一个数据集orders，包括以下文档内容：

		{
	  	cust_id: "abc123",
	  	ord_date: ISODate("2012-11-02T17:04:11.102Z"),
	  	status: 'A',
	  	price: 50,
	  	items: [ { sku: "xxx", qty: 25, price: 1 },
	   	        { sku: "yyy", qty: 25, price: 1 } ]
		}


| SQL示例 | MongoDB示例 | 描述 |
| -- | -- | -- |
| SELECT COUNT(*) AS count <br>FROM orders | db.orders.aggregate(<br> [{ <br> $group: { <br>_id: null, <br>count: { $sum: 1 }<br>}<br>}<br>]] ) ) | 统计orders总量 |
|SELECT SUM(price) AS total <br>FROM orders | db.orders.aggregate( [ {<br> $group: {<br>        _id: null,<br>        total: { $sum: "$price" }<br>     }<br>   } ] )| 计算所有orders价格总和|
|SELECT cust_id,<br>SUM(price) AS total<br>FROM orders<br>GROUP BY cust_id|db.orders.aggregate( [{<br>$group: {<br>_id: "$cust_id",<br>total: { $sum: "$price" }     }}<br>] )| 对每个唯一的cust_id，计算价格总和 |
|--|--|--|
|--|--|--|
|--|--|--|
|--|--|--|
|--|--|--|
|--|--|--|

