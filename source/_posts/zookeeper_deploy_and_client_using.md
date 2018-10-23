---
title:  "Zookeeper的安装部署与客户端使用"
date: 2016-07-23 00:00:00
categories: Zookeeper
tags: Zookeeper
toc:
  on: true
  max_depth: 3
  nowrap: false
  list_number: true
toc_list_number: true
---

zookeeper官方网站: http://zookeeper.apache.org/

## Zookeeper的安装与部署
Zookeeper通过Java编写，因此运行前需要配置好JRE

### 下载
zookeeper下载地址为http://mirrors.cnnic.cn/apache/zookeeper/
<!-- more -->

### 安装
Zookeeper官方下载的文件为.tar.gz格式，我们只需解压即可完成安装

```linux
tar -xzvf zookeeper-3.4.6.tar.gz
```

### 配置

#### 创建zoo.cfg文件
我们进入到解压后的Zookeeper目录下，找到/conf/zoo_sample.cfg文件，重命名为zoo.cfg

```
# The number of milliseconds of each tick
# 基本事件单元，以毫秒为单位。它用来指示心跳，最小的 session 过期时间为两倍的 tickTime.
tickTime=2000
# The number of ticks that the initial 
# synchronization phase can take
initLimit=10
# The number of ticks that can pass between 
# sending a request and getting an acknowledgement
syncLimit=5
# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just 
# example sakes.
# 存储内存中数据库快照的位置，如果不设置参数，更新事务日志将被存储到默认位置
dataDir=/home/yumao/zookeeper/data
dataLogDir=/home/yumao/zookeeper/log
# the port at which the clients will connect
# 监听客户端连接的端口
clientPort=2181
# the maximum number of client connections.
# increase this if you need to handle more clients
#maxClientCnxns=60
#
# Be sure to read the maintenance section of the 
# administrator guide before turning on autopurge.
#
# http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance
#
# The number of snapshots to retain in dataDir
#autopurge.snapRetainCount=3
# 格式为server.id=host:port:port, 其中id为在dataDir目录下创建的myid文件中的数字, 每个机器的数字必须保证唯一, id的范围是1~255
# 以下配置为单机配置
server.1=127.0.0.1:2888:3888

# Set to "0" to disable auto purge feature
#autopurge.purgeInterval=1
```

#### 创建myid文件
由于在zoo.cfg配置中指定了dataDir位置，进入dataDir=/home/yumao/zookeeper/data目录下，创建文件myid，内容为1

#### 启动服务

```
yumao@ubuntu:/opt/zookeeper-3.4.6/bin$ sudo ./zkServer.sh start
JMX enabled by default
Using config: /opt/zookeeper-3.4.6/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED

```

#### 停止服务
```
yumao@ubuntu:/opt/zookeeper-3.4.6/bin$ sudo ./zkServer.sh stop
JMX enabled by default
Using config: /opt/zookeeper-3.4.6/bin/../conf/zoo.cfg
Stopping zookeeper ... STOPPED
yumao@ubuntu:/opt/zookeeper-3.4.6/bin$ 
```

#### 设置环境变量

在/etc/profile文件中添加

```
#Set ZooKeeper Enviroment
export ZOOKEEPER_HOME=/opt/zookeeper-3.4.6
export PATH=$PATH:$ZOOKEEPER_HOME/bin:$ZOOKEEPER_HOME/conf
```

---



## Zookeeper客户端的使用

### 客户端启动

```
sh zkCli.sh 
```

注：在执行该脚本的时候，系统报了个错

> zkCli.sh: 81: /opt/zookeeper-3.4.6/bin/zkEnv.sh: Syntax error: "(" unexpected (expecting "fi")

一般出现该问题是因为没有配置Zookeeper的环境变量，但还有一个问题是ubuntu本身的配置问题导致，执行以下命令后解决。

```
yumao@ubuntu:/bin$ ls -l /bin/sh
lrwxrwxrwx 1 root root 4 May 17 10:34 /bin/sh -> dash
yumao@ubuntu:/bin$ sudo ln -sf bash /bin/sh
yumao@ubuntu:/bin$ ls -l /bin/sh
lrwxrwxrwx 1 root root 4 Jul 26 03:23 /bin/sh -> bash
```

若客户端脚本成功启动，则系统控制台输出如下

```
yumao@ubuntu:/opt/zookeeper-3.4.6/bin$ sudo sh ./zkServer.sh start
JMX enabled by default
Using config: /opt/zookeeper-3.4.6/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
yumao@ubuntu:/opt/zookeeper-3.4.6/bin$ sudo sh zkCli.sh 
Connecting to localhost:2181
2016-07-26 03:29:22,781 [myid:] - INFO  [main:Environment@100] - Client environment:zookeeper.version=3.4.6-1569965, built on 02/20/2014 09:09 GMT
2016-07-26 03:29:22,789 [myid:] - INFO  [main:Environment@100] - Client environment:host.name=ubuntu
2016-07-26 03:29:22,790 [myid:] - INFO  [main:Environment@100] - Client environment:java.version=1.7.0_51
2016-07-26 03:29:22,791 [myid:] - INFO  [main:Environment@100] - Client environment:java.vendor=Oracle Corporation
2016-07-26 03:29:22,791 [myid:] - INFO  [main:Environment@100] - Client environment:java.home=/usr/jdk1.7.0_51/jre
2016-07-26 03:29:22,791 [myid:] - INFO  [main:Environment@100] - Client environment:java.class.path=/opt/zookeeper-3.4.6/bin/../build/classes:/opt/zookeeper-3.4.6/bin/../build/lib/*.jar:/opt/zookeeper-3.4.6/bin/../lib/slf4j-log4j12-1.6.1.jar:/opt/zookeeper-3.4.6/bin/../lib/slf4j-api-1.6.1.jar:/opt/zookeeper-3.4.6/bin/../lib/netty-3.7.0.Final.jar:/opt/zookeeper-3.4.6/bin/../lib/log4j-1.2.16.jar:/opt/zookeeper-3.4.6/bin/../lib/jline-0.9.94.jar:/opt/zookeeper-3.4.6/bin/../zookeeper-3.4.6.jar:/opt/zookeeper-3.4.6/bin/../src/java/lib/*.jar:/opt/zookeeper-3.4.6/bin/../conf:$CLASSPATH:$JAVA_HOME/lib:$JAVA_HOME/jre/lib
2016-07-26 03:29:22,791 [myid:] - INFO  [main:Environment@100] - Client environment:java.library.path=/usr/java/packages/lib/amd64:/usr/lib64:/lib64:/lib:/usr/lib
2016-07-26 03:29:22,792 [myid:] - INFO  [main:Environment@100] - Client environment:java.io.tmpdir=/tmp
2016-07-26 03:29:22,792 [myid:] - INFO  [main:Environment@100] - Client environment:java.compiler=<NA>
2016-07-26 03:29:22,792 [myid:] - INFO  [main:Environment@100] - Client environment:os.name=Linux
2016-07-26 03:29:22,792 [myid:] - INFO  [main:Environment@100] - Client environment:os.arch=amd64
2016-07-26 03:29:22,793 [myid:] - INFO  [main:Environment@100] - Client environment:os.version=4.4.0-31-generic
2016-07-26 03:29:22,793 [myid:] - INFO  [main:Environment@100] - Client environment:user.name=root
2016-07-26 03:29:22,793 [myid:] - INFO  [main:Environment@100] - Client environment:user.home=/root
2016-07-26 03:29:22,793 [myid:] - INFO  [main:Environment@100] - Client environment:user.dir=/opt/zookeeper-3.4.6/bin
2016-07-26 03:29:22,794 [myid:] - INFO  [main:ZooKeeper@438] - Initiating client connection, connectString=localhost:2181 sessionTimeout=30000 watcher=org.apache.zookeeper.ZooKeeperMain$MyWatcher@4b4cab63
Welcome to ZooKeeper!
JLine support is enabled
2016-07-26 03:29:22,865 [myid:] - INFO  [main-SendThread(localhost:2181):ClientCnxn$SendThread@975] - Opening socket connection to server localhost/127.0.0.1:2181. Will not attempt to authenticate using SASL (unknown error)
2016-07-26 03:29:22,887 [myid:] - INFO  [main-SendThread(localhost:2181):ClientCnxn$SendThread@852] - Socket connection established to localhost/127.0.0.1:2181, initiating session
[zk: localhost:2181(CONNECTING) 0] 2016-07-26 03:29:22,936 [myid:] - INFO  [main-SendThread(localhost:2181):ClientCnxn$SendThread@1235] - Session establishment complete on server localhost/127.0.0.1:2181, sessionid = 0x15626c0466b0000, negotiated timeout = 30000

WATCHER::

WatchedEvent state:SyncConnected type:None path:null


```

若想以指定Zookeeper地址启动，则命令应输入

> sh zkCli.sh -server ip:port

---



### 通过客户端创建一个Zookeeper节点

> create [-s]  [-e] path data acl

其中 -s或-e分别指节点的特性，-s代表顺序节点，-e则代表临时节点。若不添加-e或-s参数，默认创建持久节点。

acl表示用来进行权限控制。

```
WatchedEvent state:SyncConnected type:None path:null
create /test_root test_data
Created /test_root
[zk: localhost:2181(CONNECTED) 1] 
```



### 读取Zookeeper节点

#### ls命令

列出指定节点下的所有子节点。只能看到指定节点下的第一级的所有子节点。

> ls path [watch]



case：在根节点下创建了/test_root持久节点后，使用ls命令查看根节点。然后再/test_root节点下创建了新的持久子节点1，再次使用ls命令查看根节点，结果无变化。

```
WatchedEvent state:SyncConnected type:None path:null
create /test_root test_data
Created /test_root
[zk: localhost:2181(CONNECTED) 1] ls /
[test_root, dubbo, zookeeper]
[zk: localhost:2181(CONNECTED) 2] create /test_root/1 test_data_1
Created /test_root/1
[zk: localhost:2181(CONNECTED) 3] ls /
[test_root, dubbo, zookeeper]
[zk: localhost:2181(CONNECTED) 4] ls /test_root
[1]
[zk: localhost:2181(CONNECTED) 5]
```



#### get命令

获取指定节点数据内容和属性信息

> get path [watch]

case：使用get命令查看根节点test_root

```
[zk: localhost:2181(CONNECTED) 5] get /test_root
test_data
cZxid = 0xbf
ctime = Tue Jul 26 03:35:51 PDT 2016
mZxid = 0xbf
mtime = Tue Jul 26 03:35:51 PDT 2016
pZxid = 0xc0
cversion = 1
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 9
numChildren = 1
[zk: localhost:2181(CONNECTED) 6] 
```

### 更新Zookeeper节点
#### set命令
> set path data [version]

case: 将/test_root节点的值更新为luffy，注意更新后节点version的变化

```
[zk: localhost:2181(CONNECTED) 3] get /test_root
test_data
cZxid = 0xbf
ctime = Tue Jul 26 03:35:51 PDT 2016
mZxid = 0xbf
mtime = Tue Jul 26 03:35:51 PDT 2016
pZxid = 0xc0
cversion = 1
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 9
numChildren = 1
[zk: localhost:2181(CONNECTED) 4] set /test_root luffy
cZxid = 0xbf
ctime = Tue Jul 26 03:35:51 PDT 2016
mZxid = 0xc3
mtime = Tue Jul 26 19:24:32 PDT 2016
pZxid = 0xc0
cversion = 1
dataVersion = 1
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 5
numChildren = 1
[zk: localhost:2181(CONNECTED) 5] 
```


### 删除Zookeeper节点
#### delete 命令
> delete path [version]
> 注意，只能删除叶子节点。

case: 将/test_root下的1节点删除。

```
[zk: localhost:2181(CONNECTED) 5] ls /test_root
[1]
[zk: localhost:2181(CONNECTED) 6] delete /test_root/1
[zk: localhost:2181(CONNECTED) 7] ls /test_root      
[]
[zk: localhost:2181(CONNECTED) 8] 
```

