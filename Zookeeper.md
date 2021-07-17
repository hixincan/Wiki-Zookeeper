*书山有路勤为径，学海无涯苦作舟*





# 认识

zoo keeper 动物管理员，都有哪些动物

* 大象 hadoop，大象集群
* pig  猪群
* hive 蜂巢，蜂群
* 其他大数据生态环境



管理动物，也需要管理员是一个群体



**作用**

1. 中间件，提供协调服务

中间件的作用：计算、存储



2. 作用于分布式系统，发挥其优势，可以为大数据服务



3. 支持 java，提供java和c语言的客户端api





**什么是分布式系统？**

1. 很多台计算机组成一个整体，一个整体一致对外并且处理同一请求

2. 内部的每台计算机都可以相互通信（rest/rpc）

restful 是 websocket 通信

3. 客户端到服务端的一次请求到响应结束会历经多台计算机



特性

* 主机、备机







**分布式系统的瓶颈**

大并发场景下，先来先处理，需要分布式锁（同步）来处理，保证数据的一致性

例子：交互拥堵时，通常需要有一个交警来指挥车辆



**zookeeper的特性**

* 一致性：数据一致性，数据按照顺序分批入库

* 原子性：事务要么成功要么失败，不会局部化
* 单一视图：客户端连接集群中的任一zk节点，数据都是一致的 - 访问任一节点的数据都是一致的
* 可靠性：每次对zk的操作状态都会保存在服务端 —— 日志，有操作记录，可以回滚
* 实时性：客户端可以读取到zk服务端的最新数据 —— 10 - 20 秒之间；大数据下，断开再连接能拿到最新的数据







## 安装

zk 的版本 3.7.0



配置环境变量

```shell
export JAVA_HOME=/usr/java/jdk1.8.0_291-amd64 #其他软件依赖
export ZK_HOME=/opt/apache-zookeeper-3.7.0-bin
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar # .为当前目录，用于JVM查找类
export PATH=$PATH:$JAVA_HOME/bin:$ZK_HONE/bin
```





## zoo.cfg配置

**1、tickTime：**

用于计算的==基本时间单元==

默认 2000 毫秒，即 2秒

initLimit 为 10 —— 表示 10 * 2 = 20秒，tickTime 是基本时间单元

syncLimit 为 5 —— 表示 5*2 =  10秒



**2、initLimit：**

用于集群，允许从节点连接并同步到master节点的==初始化连接时间==，以tickTime的==倍数==来表示



**3、syncLimit：**

用于集群，master主节点与从节点之间发送消息，==请求和应答时间长度==。（心跳机制）



**4、dataDir：**

必须配置，存储事务文件等

建议：/usr/local/zookeeper/dataDir



**5、dataLogDir：**

日志目录，如果不配置会和dataDir公用

建议：/usr/local/zookeeper/dataLogDir



**6、clientPort：**

连接服务器的端口，默认2181











## 常用命令



zkServer.sh  直接输入，系统会给出命令的相关提示

```shell
[root@centos7 apache-zookeeper-3.7.0-bin]# bin/zkServer.sh
ZooKeeper JMX enabled by default
Using config: /opt/apache-zookeeper-3.7.0-bin/bin/../conf/zoo.cfg
Usage: bin/zkServer.sh [--config <conf-dir>] {start|start-foreground|stop|version|restart|status|print-cmd}
```



查看zk的状态（目前是单机）

```shell
[root@centos7 apache-zookeeper-3.7.0-bin]# bin/zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /opt/apache-zookeeper-3.7.0-bin/bin/../conf/zoo.cfg
Client port found: 2181. Client address: localhost. Client SSL: false.
Mode: standalone
```





# 数据模型

## 基本概念

1. 是一个树形结构，类似于  linux 的文件目录

2. 每一个节点都称之为znode，它可以有子节点（相当于父目录下的子目录），也可以有数据（每个目录中有一些目录数据）

3. 每个节点分为==临时节点==和==永久节点==，临时节点在客户端断开后消失

* 永久节点：持久化的过程，只有人为情况下才能删除，例如session丢失或超时，数据还是存在
* 临时节点：也可以认为操作，例如 session 失效数据就会丢失

4. 每个zk节点都各自的版本号，可以通过命令行来显示节点信息（节点详情，包含版本号）

* 每当节点数据发生变化，那么该节点的版本号会累加（乐观锁）
* 删除/修改过时节点，版本号不匹配则会报错 —— 查询节点时，节点初始为1，经过修改版本变为 2、3，当我们还是传入旧的版本号时，会报错提示版本不匹配（与数据库中使用乐观锁的表现）

5. 每个zk节点存储的数据不宜过大，几K即可

6. 节点可以设置权限acl，可以通过权限来限制用户的访问





## 基本操作



客户端连接 `./zkCli.sh`

```shell
# 表示已经连接成功
[zk: localhost:2181(CONNECTED) 0]
```

输入 help 会有命令

```shell
[zk: localhost:2181(CONNECTED) 0] help
ZooKeeper -server host:port -client-configuration properties-file cmd args
	addWatch [-m mode] path # optional mode is one of [PERSISTENT, PERSISTENT_RECURSIVE] - default is PERSISTENT_RECURSIVE
	addauth scheme auth
	close
	config [-c] [-w] [-s]
	connect host:port
	create [-s] [-e] [-c] [-t ttl] path [data] [acl]
	delete [-v version] path
	deleteall path [-b batch size]
	delquota [-n|-b|-N|-B] path
	get [-s] [-w] path
	getAcl [-s] path
	getAllChildrenNumber path
	getEphemerals path
	history
	listquota path
	ls [-s] [-w] [-R] path
	printwatches on|off
	quit
	reconfig [-s] [-v version] [[-file path] | [-members serverID=host:port1:port2;port3[,...]*]] | [-add serverId=host:port1:port2;port3[,...]]* [-remove serverId[,...]*]
	redo cmdno
	removewatches path [-c|-d|-a] [-l]
	set [-s] [-v version] path data
	setAcl [-s] [-v version] [-R] path acl
	setquota -n|-b|-N|-B val path
	stat [-w] path
	sync path
	version
	whoami
```



通过 ls 命令查看zk的初始目录

```shell
[zk: localhost:2181(CONNECTED) 1] ls /
[admin, brokers, cluster, config, consumers, controller_epoch, feature, isr_change_notification, latest_producer_id_block, log_dir_event_notification, zookeeper]
[zk: localhost:2181(CONNECTED) 2] ls /zookeeper
[config, quota]
[zk: localhost:2181(CONNECTED) 3] ls /zookeeper/quota
[]
```























































































































































