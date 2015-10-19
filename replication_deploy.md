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

4. 