# mongodb MapReduce

Map-reduce 是一个数据处理的过程，在大数据量处理时能够有效聚合结果。对于map-reduce操作，Mongodb提供了相关的命令，结果如下图所示：

![mapreduce](https://docs.mongodb.org/manual/_images/map-reduce.png)

在上面的map-reduce操作中，mongodb对每个输入的文档应用map操作（比如，符合过滤条件的数据文档集）。Map方法将 key-value映身成队。对于那些拥有多个value的key值，Mongodb执行reduce操作，收集和简化聚合数据。之后Mongodb存储result在文档集合中。可选的，reduce操作可以把结果传递给一个finalize函数进一步的简化或者处理聚合的结果。

在Mongodb中的所有map-reduce都是Javascript函数并且运行在mongod处理之内。 Map-reducer操作取出每个单独的文档作为一个输入，并且能够在开始map阶段之前做一些基本的排序和limit限定。 MapReduce能够返回一个map-reduce操作文档结果，也可以写入结果到文档集中。输入和输出也可以共享。


## Map-Reduce javascript函数

在Mongodb中map-reduce操作使用定制的javascript函数去map或者联合同到一个key中。 如果一个key有多个值，会把它们映身在一起，最终reduce操作每个key的value组成单个对象。

定制的javascript函数提供灵活的map-reduce操作。比如，当处理一个文档时，map函数不仅能够创建key和value的映射或非映射。 map-reduce操作也能够使用一个定制的javascript函数，在map-reduce操作之后，进一步做结果的最终修正，比如添加附加计算。

## Map-Reduce 行为

在Mongodb中，map-reduce操作能够向一个集合写入结果，也能够返回结果列表。 如果你想要写一个map-reduce 输出一个集合，你可以执行一系列的map-reduce后续操作在同一输入集合中，比如合并替换，合并，或reduce之前的结果产生新结果。

当map-reduce操作返回一个集合结果时，结果文档必须是BSON格式而且满足文档集的大小限制，默认情况下结果文档集在 16MB内。 如果想要超越限制，在后续的文档中会有介绍。








