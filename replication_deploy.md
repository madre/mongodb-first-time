# 副本集模式部署

本文的最初预设使用ubuntu server安装mongodb，这里的集群部署主要介绍ubuntu server中的副本集模式：

有三台ubuntu server14.04虚拟主机，三台主机上面均安装了mongodb，现在想要建立一个 primary, secondary, Arbiter三机联合的集群模式。


1. 检查下 /etc/hosts
    
    在搭建mongodb集群之前，我们要先检查下ubuntu server网络设置环境；把需要联合建立集群的三台主机加入彼此的hosts中，避免无法通信。
    
    > 192.68.199.138 mongo1 
    > 
    > 192.68.199.139 mongo2 
    > 
    > 192.68.199.140 mongo3
    >

2. 修改 /etc/mongod.conf
    
    修改各主机的mongod.conf配置，指定相关的配置
 
    > bind_ip=0.0.0.0  # need to set 0.0.0.0
    >
    > port=27017
    
    > \# 添加复本集配置
    >
    > replication:
    >
    >  oplogSizeMB: 500
    >
    >  replSetName: repdata

3. 启动其中一台主机

    > sudo service mongod start
    
    命令行中执行:
    
    > mongo 
    
    进入mongodb shell中，执行 ：
    
    > rs.initiate()

    切记，rs.initiate()命令是初始化集群状态，只能在其中一台机器中执行，如果在三台机器中都执行，会变成你想要建立三个集群。如果在三台主机中都执行了 rs.initiate(), 三台主机就无法通信联合成为一个集群！
    
    如果真的不小心在多台机器中都执行了rs.initiate()，按以下步骤解决：
    1. 关闭正在运行的mongod服务
    2. 删除mongod服务指定的数据路径全部文件
    3. 重启mongodb服务
    4. 重新在主服务器中添加复本机

4. 查看主机当前的配置状态
    
    执行: rs.conf()
    
    > { "_id" : "rs1", "version" : 8, "members" : [ { "_id" : 0, "host" : "10.142.38.138:27017" } ] }

    这里一定要确认下 host对应的是ip地址，还是域名字符地址。
    如果是域名地址，除非是绑定好的，否则最好重新按下面的方式指定成ip地址；
    
    > cfg = rs.conf()
    >
    > cfg.members[0].host = "192.168.199.18"
    > 
    > rs.reconfig(cfg)

5. 添加副本

    执行 rs.add("192.168.199.19:27017")；
    
    如果有多台主机，可以依次添加到副本集中。
    
6. 添加指定仲裁机
    
    rs.addArb("192.168.199.20:27017");

7. 查看配后的副本集群状态

	rs.conf()
	
		{
		"_id" : "repdata",
		"version" : 9,
		"members" : [
			{
				"_id" : 0,
				"host" : "192.168.199.18:27017",
				"arbiterOnly" : false,
				"buildIndexes" : true,
				"hidden" : false,
				"priority" : 1,
				"tags" : {
	
				},
				"slaveDelay" : 0,
				"votes" : 1
			},
			{
				"_id" : 1,
				"host" : "192.168.199.19:27017",
				"arbiterOnly" : false,
				"buildIndexes" : true,
				"hidden" : false,
				"priority" : 1,
				"tags" : {
	
				},
				"slaveDelay" : 0,
				"votes" : 1
			},
			{
				"_id" : 2,
				"host" : "192.168.199.20:27017",
				"arbiterOnly" : true,
				"buildIndexes" : true,
				"hidden" : false,
				"priority" : 1,
				"tags" : {
	
				},
				"slaveDelay" : 0,
				"votes" : 1
			}
		],
		"settings" : {
			"chainingAllowed" : true,
			"heartbeatTimeoutSecs" : 10,
			"getLastErrorModes" : {
	
			},
			"getLastErrorDefaults" : {
				"w" : 1,
				"wtimeout" : 0
			}
		}
		}
		
	1. 确认下集群是有指定好的3个 member组成；
	2. 确认各个子集的host是ip地址，或者为有效的域名地址；如果不是，按上面提到的方法使用 cfg.members[0].host = "192.168.199.18"， rs.reconfig(cfg)重新指定

8. 查看当前运行状态

	rs.status()
	
		{
		"set" : "repdata",
		"date" : ISODate("2015-10-16T18:55:33.785Z"),
		"myState" : 1,
		"members" : [
			{
				"_id" : 0,
				"name" : "192.168.199.18:27017",
				"health" : 1,
				"state" : 1,
				"stateStr" : "PRIMARY",
				"uptime" : 47550,
				"optime" : Timestamp(1444980684, 1),
				"optimeDate" : ISODate("2015-10-16T07:31:24Z"),
				"electionTime" : Timestamp(1444974225, 2),
				"electionDate" : ISODate("2015-10-16T05:43:45Z"),
				"configVersion" : 9,
				"self" : true
			},
			{
				"_id" : 1,
				"name" : "192.168.199.19:27017",
				"health" : 1,
				"state" : 2,
				"stateStr" : "SECONDARY",
				"uptime" : 23008,
				"optime" : Timestamp(1444980684, 1),
				"optimeDate" : ISODate("2015-10-16T07:31:24Z"),
				"lastHeartbeat" : ISODate("2015-10-16T18:55:33.317Z"),
				"lastHeartbeatRecv" : ISODate("2015-10-16T18:55:33.325Z"),
				"pingMs" : 0,
				"syncingTo" : "192.168.199.18:27017",
				"configVersion" : 9
			},
			{
				"_id" : 2,
				"name" : "192.168.199.20:27017",
				"health" : 1,
				"state" : 7,
				"stateStr" : "ARBITER",
				"uptime" : 41049,
				"lastHeartbeat" : ISODate("2015-10-16T18:55:33.286Z"),
				"lastHeartbeatRecv" : ISODate("2015-10-16T18:55:33.374Z"),
				"pingMs" : 0,
				"configVersion" : 9
			}
		],
		"ok" : 1
		}

	核心确认三个members的 stateStr状态码，如果分别是 PRIMARY、SECONDARY、ARBITER，说明设置无误；如果是 STARTUP或者其他状态，说明通信有问题，需要查看网络或者其他相关设置。
	
