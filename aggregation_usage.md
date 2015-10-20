mongodb的集合计算从2.2版开始引进了新的框架结构，目前版本已经到3.0.7，旧的集合结构不再这里说明。
集合计算分为两种类型：

1. 单数据集的集合计算 (Single Purpose Aggregation)
2. 跨库的数据管道集合计算(aggregation pipeline)
 
集合计算方法包括以下几种：

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



## Grouping
