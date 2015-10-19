# mongodb副本集

副本集就是在多个不同的服务器之间同步数据的过程。
副本集提供了简易的方法来实现增减数据可用性，因为在多个不同的服务器上有相同的数据备份，副本集也能够避免单台机器数据丢失的问题。
副本集也能够兼容硬盘损坏、服务中断等异常情况；有多份数据拷贝，可以拿其中一份作为灾备方案。

mongodb的副本集是一组 mongod 运行实例，这些实例包含着相同的数据集。其中有一个 primary（主），用来接收写操作；剩下的其他实例，作为从属实例，从主实例中接收操作同步数据。

副本集分成几种功能模式：

* 仅仅作为数据备份，一台主机读写数据，有复本集同步备份数据

![副本集](https://docs.mongodb.org/manual/_images/replica-set-read-write-operations-primary.png)


* 多份数据，主机出问题，复本集可以自动变成主机

![多备](https://docs.mongodb.org/manual/_images/replica-set-primary-with-two-secondaries.png)

* 主机、复本集、仲裁机

![仲裁模式](https://docs.mongodb.org/manual/_images/replica-set-primary-with-secondary-and-arbiter.png)

三种模式有各自的适用场景，其中第三种相对可用性最高，有主机群，有备份机，一台或几台不浪费空间、性能不用很高的机器作为仲裁机，判断和维护集群状态。
