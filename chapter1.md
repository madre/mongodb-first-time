mongodb包结构
=====

mongodb有以下包文件：

* mongodb-org

 > 这个包是元数据包，安装它会自动安装下面列出的4个组件包。

* mongodb-org-server

 > 这个包是mongod后台守护进程、关联配置、初始化脚本。

* mongodb-org-mongos
 > 这个包是mongos守护程序，mongos是用于mongodb的sharding配置。

* mongodb-org-shell
 > 这个是mongo shell 客户端程序包

* mongodb-org-tools
 > 这个是mongodb的工具集包，包括: mongoimport bsondump, mongodump, mongoexport, mongofiles, mongooplog, mongoperf, mongorestore, mongostat, and mongotop.

可以单独安装其中任一包。

ubuntu 安装 mongodb
=====

* 使用ubuntu的包管理器导入mongodb的GPG公钥
 
 > sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 7F0CEB10

* 创建mongodb列表文件
 > echo "deb http://repo.mongodb.org/apt/ubuntu "$(lsb_release -sc)"/mongodb-org/3.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.0.list
 
* 重新加载本地包
 > sudo apt-get update
 
* 安装mongodb包

 > sudo apt-get install -y mongodb-org
 >
 > 也可以指定版本安装： sudo apt-get install -y mongodb-org=3.0.6 mongodb-org-server=3.0.6 mongodb-org-shell=3.0.6 mongodb-org-mongos=3.0.6 mongodb-org-tools=3.0.6

* mongodb启动
 > sudo service mongod start
 >
 > sudo service mongod stop
 >
 > sudo service mongod restart
   
* 卸载mongodb
 > 卸载需要先停止服务: sudo service mongod stop
 >
 > sudo apt-get purge mongodb-org*
 >
 > 删除日志文件：sudo rm -r /var/log/mongodb
 >
 > 删除数据文件：sudo rm -r /var/lib/mongodb

Red Hat Enterprise Linux 或 CentOS安装
=====

* 配置yum包管理系统

	> 创建文件 /etc/yum.repos.d/mongodb-org-3.0.repo 以便能够直接使用yum安装mongodb
	> 
	> 文件内容如下：
	> 
		[mongodb-org-3.0]
			name=MongoDB Repository
			baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/3.0/x86_64/
			gpgcheck=0
			enabled=1


* 安装mongodb包

	> sudo yum install -y mongodb-org
	> 
	> 或指定版本安装: sudo yum install -y mongodb-org-3.0.6 mongodb-org-server-3.0.6 mongodb-org-shell-3.0.6 mongodb-org-mongos-3.0.6 mongodb-org-tools-3.0.6


* mongodb启动
	
	> sudo service mongod start
	>
	> sudo service mongod stop
	>
	> sudo service mongod restart
 
* 卸载mongodb
 > 卸载需要先停止服务: sudo service mongod stop
 >
 > sudo yum erase $(rpm -qa | grep mongodb-org)
 >
 > 删除日志文件：sudo rm -r /var/log/mongodb
 >
 > 删除数据文件：sudo rm -r /var/lib/mongodb
