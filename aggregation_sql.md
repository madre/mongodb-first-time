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


| # | SQL示例 | MongoDB示例 | 描述 |
| -- | -- | -- | -- |
|1| SELECT COUNT(*) AS count <br>FROM orders | db.orders.aggregate(<br> [{ <br> $group: { <br>_id: null, <br>count: { $sum: 1 }<br>}<br>}<br>]] ) ) | 统计orders总量 |
|2|SELECT SUM(price) AS total <br>FROM orders | db.orders.aggregate( [ {<br> $group: {<br>        _id: null,<br>        total: { $sum: "$price" }<br>     }<br>   } ] )| 计算所有orders价格总和|
|3|SELECT cust_id,<br>SUM(price) AS total<br>FROM orders<br>GROUP BY cust_id|db.orders.aggregate( [{<br>$group: {<br>_id: "$cust_id",<br>total: { $sum: "$price" }     }}<br>] )| 对每个唯一的cust_id，<br>计算价格总和 |
|4|SELECT cust_id, SUM(price) AS total<br> FROM orders <br>GROUP BY cust_id ORDER BY total|db.orders.aggregate( [{<br>$group: {<br>_id: "$cust_id",<br>total: { $sum: "$price" }}<br>},<br>{ $sort: { total: 1 } }] )|对每一个唯一的cust_id，<br>计算价格总和，<br>结果按总和排序|
|5|SELECT cust_id,<br>ord_date,<br>SUM(price) AS total<br>FROM orders<br>GROUP BY cust_id, ord_date|db.orders.aggregate( [{<br>$group: {<br>_id: {<br>cust_id: "$cust_id",<br>ord_date: {<br>month: { $month: "$ord_date" },<br>day: { $dayOfMonth: "$ord_date" },<br>year: { $year: "$ord_date"}}}, <br>total: { $sum: "$price" }}}<br>] )|对每一个唯一的cust_id和ord_date，<br>计算价格总和|
|6|SELECT cust_id,<br>count(1) FROM orders<br>GROUP BY cust_id <br>HAVING count(1) > 1| db.orders.aggregate( [<br>{ $group: { <br>_id: "$cust_id",<br>count: { $sum: 1 }}},<br>{ $match: { count: { $gt: 1 } } } <br>] ) |如果 cust_id 对应多条记录，<br>返回cust_id 和它对应的记录数量。|
|7| SELECT cust_id, ord_date,<br> SUM(price) AS total <br>FROM orders <br>GROUP BY <br>cust_id,ord_date <br>HAVING total > 250 |db.orders.aggregate( [ {<br>$group: {<br> _id: { <br>cust_id: "$cust_id",<br>ord_date: {<br>month: { $month: "$ord_date" },<br>day: { $dayOfMonth: "$ord_date" },<br>year: { $year: "$ord_date"}}},<br>total: { $sum: "$price" }}},<br>{ $match: { total: { $gt: 250 } } }<br>] )| 对于每个唯一的 cust_id和 ord_date 分组, <br>计算价格总和；<br> 返回总各超过 250的分组。|
|8|SELECT cust_id, <br>SUM(price) as total<br>FROM orders<br>WHERE status = 'A'<br>GROUP BY cust_id|db.orders.aggregate( [<br>{ $match: { status: 'A' } },{<br>$group: {<br>_id: "$cust_id",<br>total: { $sum: "$price" }}}<br>] )|对于每个唯一的cust_id并且status值为 A，<br> 计算价格总和|
|9|SELECT cust_id,<br>SUM(price) as total<br>FROM orders<br>WHERE status = 'A'<br>GROUP BY cust_id<br>HAVING total > 250| db.orders.aggregate( [{ <br>$match: { status: 'A' } },<br>{$group: {<br>_id: "$cust_id",<br>total: { $sum: "$price" }}},<br>{ $match: { total: { $gt: 250 } } }<br>] ) |对于每个唯一的cust_id并且status值为 A，<br> 计算价格总和，<br>但只返回总和大于 250的|
|10|SELECT cust_id, <br>SUM(li.qty) as qty<br>FROM orders o, order_lineitem li<br>WHERE li.order_id = o.id<br> GROUP BY cust_id| db.orders.aggregate( [ <br>{ $unwind: "$items" },{<br>$group: {<br>_id: "$cust_id",<br>qty: { $sum: "$items.qty" }}}<br>] ) | 对于每个唯一的 cust_id，<br>计算qty字段的总和，<br>并连接返回order的记录 |
|11|SELECT COUNT(*)<br>FROM (SELECT cust_id, ord_date<br>FROM orders<br>GROUP BY cust_id,ord_date)<br>as DerivedTable| db.orders.aggregate( [{<br>$group: {<br>_id: {cust_id: "$cust_id",<br>ord_date: {<br>month: { $month: "$ord_date" },<br>day: { $dayOfMonth: "$ord_date" },<br>year: { $year: "$ord_date"}}<br>}}},<br>{$group: {<br>_id: null, <br>count: { $sum: 1 }}}<br>] ) |统计不同的 cust_id 数据，<br>按ord_date进行分组|



