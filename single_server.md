## 单机部署

在linux系统中安装Mongodb3.0后，mongodb默认的启动方式如下：
	
> sudo service mongod start
>
> sudo service mongod stop
>
> sudo service mongod restart
	
在不修改mongodb任何设置安装的情况下：

mongodb配置文件在:
>/etc/mongod.conf

mongodb数据实例存储在文件目录：
> /var/lib/mongodb

mongodb操作日志存储在:
> /var/log/mongodb

### 修改配置

如果要修改mongodb配置，可以修改 /etc/mongod.conf，重启mongodb服务。

例如，需要修改mongodb的数据存储目录，可以修改 /etc/mongod.conf 文件中的
> dbpath=/var/lib/mongodb

需要修改日志文件路径，请修改 logpath 指定的路径：
> logpath=/var/log/mongodb/mongod.log

### 特别说明
> MAC和windows中单机调试有差异

在mac中，安装后使用 mongod命令指定 dbpath或 config，启动服务；
> mongod --dbpath <path to data directory>

在windows中，安装后使用 mongod.exe指定dbpath或config，启动服务；
> C:\mongodb\bin\mongod.exe --dbpath d:\test\mongodb\data

