开发者关心的是数据库的操作使用。本章分为以下几个内容：
1. 增删改查CRUD操作
2. 数据集合Aggregation函数使用
3. 利用索引加速查询，基于地理位置查询，文本搜索

在介绍这几项内容之前，首先要明确一样内容：mongodb作为数据库，包括了各种语言版本的数据连接操作。

1. mongo Shell Edition
2. Node.JS Edition
3. Python Edition
4. C++ Edition
5. Java Edition
6. C# Edition

无论哪种连接方式，步骤均是：

1. 建立数据库连接，如python的 client = MongoClient("mongodb://mongodb0.example.net:27019")
2. 选择并访问数据库对象, 如python的 db = client.test
3. 在数据库对象上操作文档对象，或者执行数据库命令，如python的 cursor = db.restaurants.find()

但不同的是，其他语言版的数据操作更多的集中在数据库的增删改查，而shell版本的操作集成了数据库管理、配置和安全机制。所以shell版本比其他语言版本具有更多直接使用的命令，其他语言往往要通过db.runCommands() 来达到效果。
