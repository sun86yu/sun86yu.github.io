---
layout: post
book: true
background-image: http://ot1cc1u9t.bkt.clouddn.com/17-7-16/91630214.jpg
category: 架构
title: Zookeeper
tags:
- Zookeeper
- 分布式
---

Zookeeper
===
>ZooKeeper是一个开放源码的分布式协调服务。设计目标是将那些复杂且容易出错的分布式一致性服务封装起来，提供一些简单的接口给用户使用。

>其提供的服务类似于注册中心的服务，各个机器向 ZooKeeper 注册，ZooKeeper进行登录并生成唯一的自增的ID。ZooKeeper 集群里各机器共享这些信息。

>分布式应用程序可以基于它实现诸如：数据发布/订阅，负载均衡，命名服务，分布式协调/通知，集群管理，Master 选举，分布式锁，分布式队列等功能。

Zookeeper 可以保证如下分布式一致性的特性:

1. 顺序一致性.从同一个客户端发起的事务请求，最终将会严格地按照其发起顺序被应用到 ZooKeeper 中去。

2. 原子性.所有事务请求的处理结果在整个集群中所有机器上应用情况是一致的，也就是说，要么整个集群所有机器都成功的应用某一个事务要么都没有应用，一定不会出现集群中部分机器应用了事务，另一部分没有的情况。

3. 单一视图.无论客户端连接的是哪个 ZooKeeper 服务器，其看到的服务端数据模型都是一致的。

4. 可靠性.一旦服务端成功的应用了一个事务，并完成对客户端的响应，那么该事务所引起的服务端状态变更将会被一直保留下来，除非有另一个事务又对其进行了变更。

5. 实时性.通常人们对实时性的理解是一个事务被成功应用后，客户端能立即从服务端上读到这个事务变更后的最新数据状态。但___ZooKeeper仅保证在一定的时间段内，客户端最终一定能够从服务端上读取取最新的数据状态。___类似银行转帐的功能，转帐后不保证对方马上收到，但最终一定会收到。

简单的数据模型
===
>ZooKeeper提供一个简单的***树型***数据模型，类似电脑上的文件系统。ZooKeeper将这些数据全部放在内存中，以此来实现提高服务器吞吐，减少延迟的目标。

树型结构的每个节点叫作 znode。每个节点可能包含数据，也可能不包含。zookeeper 提供对外的API，让我们对节点进行操作，提供的操作有: 

1. ```create/path data``` 创建一个名为 path 的节点, 并包含数据 data。 
2. ```delete/path``` 删除名为 path 的节点。
3. ```exists/path``` 检测节点是否存在。
4. ```setData/path data``` 将 path 节点的数据设置为 data。
5. ```getData/path``` 获取节点 path 的数据。
6. ```getChildren/path``` 获得节点 path 的子节点。

节点类型
---

创建节点时，需要指定节点的类型。不同类型有不同的处理方式。类型有：

***持久节点***

持久节点创建后，只有通过 ```delete```删除后才会消失。用来存放持久的数据，就算节点被移出系统，数据也不会消失。比如在主从结构中，如果主节点崩溃了，系统会重新选举主节点，将崩溃的节点移除，但存放在原节点上的数据还会存在。

***临时节点***

临时可以用来检测节点是否正常运行。比如在主从结构中，如果我们创建的节点是临时的，当主节点崩溃时，该节点就消失了。这时我们就知道崩溃了，需要重新选举主节点。

临时节点也可以通过 ```delete```方式删除，当节点崩溃时也会自动删除。临时节点是不能有子节点的。

***有序节点***

节点可以被设置成为有序的。一个有序节点被分配一个单调递增的整数。当节点创建时，一个序号会被追回到路径后，如我们创建 ```/task/order```，则它会自动创建为 ```/task/order1```，我们再创建时，它又会创建 ```/task/order2```。

监视与通知
---

为了减少zookeeper 的请求，但又能让客户端能第一时间更新节点中的数据。zookeeper 提供了监视和通知的功能。客户端可以监听某节点，当节点有变化时，服务端会主动通知客户端。客户端接收通知后需要重新监听，因为之前的监听已经被消费了。

可以监听的事件有：节点数据变化，节点子节点变化，节点的创建或删除。

多版本控制
---

每个节点都有版本号，它随着每次数据变化而递增。```setData, delete```都可以接收版本号参数。只有版本号和服务器上的一致时操作才会成功。当有多个客户端同时操作时，版本号就显示出重要性了。

如，客户端 c1 对 ```znode/config```写入一些配置信息，初始版本号是 1，修改成功后，版本号是 2。这时如果 c2 同时发起了更新，且它传入的版本号是 1，则它的修改不会成功。

构建集群
---
ZooKeeper集群的每台机器都会在内存中维护当前整个集群的状态，集群中每台机器之间都互相保持着通信。只要集群中存在超过一半的机器能够正常工作，那么整个集群就能正常对外服务。

ZooKeeper的客户端程序会选择和集群中的做生意一台机器创建一个TCP连接，而它和服务器断开连接后，客户端会自动连接到集群中其它机器上。

顺序访问
---
对客户端的每个更新请求，ZooKeeper都会分配一个全局唯一的递增编号，它反应了所有事务操作的选后顺序。

高性能
---
ZooKeeper 将所有数据都存储在内存中，并直接服务于客户端所有非事务请求，因此它适合以读操作为主的应用场景。

简单示例
===

安装
---

```
wget http://apache.dataguru.cn/zookeeper/zookeeper-3.4.6/zookeeper-3.4.6.tar.gz
```
或:
```
wget http://apache.fayea.com/zookeeper/stable/zookeeper-3.4.6.tar.gz
```

其它地址列表: ___http://www.apache.org/dyn/closer.cgi/zookeeper/___

```
tar -zxf zookeeper-3.4.6.tar.gz
cd zookeeper-3.4.6
cp conf/zoo_sample.cfg conf/zoo.cfg

cd src/
./configure --prefix=/usr/local/zookeeper/
make
make install
```

设置简单的配置文件 conf/zoo.cfg:

***单机模式:***

```
dataDir=/var/lib/zookeeper/
clientPort=2181
initLimit=5
syncLimit=2
server.1=192.168.190.190:2888:3888
```

***伪集群模式***

```
dataDir=/var/lib/zookeeper/
clientPort=2181
initLimit=5
syncLimit=2
server.1=192.168.190.190:2888:3888
server.2=192.168.190.190:2889:3889
server.3=192.168.190.190:2890:3890
```

启动:

```
bin/zkServer.sh start
netstat -openlut | grep 2181
```

发现 2181 端口已经在监听了

***客户端连接:***

```
bin/zkCli.sh
```

输入命令:

```
ls /
create /test 1
ls /
[test, zookeeper]
```

PHP客户端
---
***PHP 扩展***

扩展源址: ___https://github.com/andreiz/php-zookeeper___

```
git clone https://github.com/andreiz/php-zookeeper.git
cd php-zookeeper
phpize
./configure --with-php-config=/usr/local/php/bin/php-config --with-libzookeeper-dir=/usr/local/zookeeper/
make
make install
```

更改 php.ini 添加:

```
extension = zookeeper.so
```

测试安装结果:

```
php -m | grep zook
zookeeper
```

已经有 zookeeper 模块.安装成功

API使用
---
vim demo1.php

```
<?php

class ZookeeperDemo extends Zookeeper {

  public function watcher( $i, $type, $key ) {
    echo "Insider Watcher\n";

    // Watcher gets consumed so we need to set a new one
    $this->get( '/test', array($this, 'watcher' ) );
  }

}

$zoo = new ZookeeperDemo('127.0.0.1:2181');
$zoo->get( '/test', array($zoo, 'watcher' ) );

while( true ) {
  echo '.';
  sleep(2);
}
```

运行:

```
php demo1.php
```

>此处应该会每隔2秒产生一个点。现在切换到ZooKeeper客户端，并更新 /test 值:

```
[zk: localhost:2181(CONNECTED) 4] set /test foo
cZxid = 0x7
ctime = Tue Jul 21 16:43:30 CST 2015
mZxid = 0x9
mtime = Tue Jul 21 16:44:03 CST 2015
pZxid = 0x7
cversion = 0
dataVersion = 1
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 3
numChildren = 0
```

demo1.php 输出:

```
Insider Watcher
```

>原理:
ZooKeeper提供了可以绑定在znode的监视器。如果监视器发现znode发生变化，该service会立即通知所有相关的客户端。

>这就是PHP脚本如何知道变化的。Zookeeper::get方法的第二个参数是回调函数。当触发事件时，监视器会被消费掉，所以我们需要在回调函数中再次设置监视器。

典型应用场景
===

主/从结构中的Master 选举
---
>应用中往往需要一个 Master 来进行统一协调。或者需要一个唯一的机器来进行一些特殊业务的处理。如：数据库的写功能。

在该模式中，涉及到三个角色：主节点、从节点、客户端。

1. 主节点负责监视新的从节点和任务，分配任务给可用的从节点。
2. 从节点通过注册自己，确保主节点知道它们可以接受任务，然后开始监视新任务。
3. 客户端创建新任务并等待系统响应。

设计细节是：

1. 创建临时主节点 ```/master```，值是```master1.example.com:3306```, 使用临时节点是为了在该节点崩溃时能马上被监测到。
2. 在 ```/master```节点上添加```NodeDelete```的监听事件。
3. 创建备份节点```/master```，内容是```master2.example.com:3306```。
4. 创建业务上用到的工作节点```/workers, /tasks, /assign```，这三个节点是持久性的。```/workers```表示有哪些工作节点，```/tasks```表示有哪些任务要执行,```/assign```表示任务的分配情况。
5. 主节点需要监听```/workers 和 /tasks```因为它需要进行任务的分配。所以它得知道有哪些工作节点和哪些待执行的任务。
6. 系统初始化时，工作节点需要在 ```/workers```下创建临时性的子节点来表示自己。如：```/workers/server1```，值是 ```server1.example:8888```，同时在 ```/assign```下创建持久节点 ```/assign/server1```来接收任务，并监听它的子节点变化，等待任务分配。
7. 主节点监听到 ```/workers```的子节点发生了变化，知道有工作节点加入。从```/tasks```获得子节点，即要执行的任务列表。并将任务分配给工作节点，即在 ```/assign/server1```下创建有序节点 ```/assign/server1/task0001```。系统初始化时任务列表是空的，所以就没有这一步了，但主节点除了监听工作节点列表，也监听了任务列表。所以当客户端提交任务（添加 /tasks 子节点）时，主节点也能知道，它也就可以执行这一步。将任务分配给某个节点。
8. 当主节点分配了任务后，由于工作节点监听了任务分配节点```/assign```，它就会接收到通知，这时执行自己的业务逻辑即可。

>注册：监听事件接收到通知后，一定要再次监听。因为监听事件是消费型的，接收通知会就失效了。

数据发布/订阅
---
>数据发布/订阅系统就是发布者将数据发布到 ZooKeeper 的一个或一系列节点上，供订阅者进行数据订阅，进而达到动态获取数据的目的。

>发布/订阅一般有两种模式：推和拉。
>在推模式中，服务器主动将数据更新发送给所有订阅的客户端；而拉模式则由客户端主动发出请求获取最新数据。

>ZooKeeper采用的是推拉结合：客户端向服务端注册自己需要关注的节点，一旦该节点的数据发生变更，那么服务端就会向相应的客户端发送事件通知，客户端接收到通知后，再主动到服务端获取最新的数据。

***案例: 配置获取***

服务集群在启动初始化阶段，先从上面的 ZooKeeper 配置节点上读取配置信息，同时客户端还要在该配置节点上注册一个数据变更的 Watcher 监听，一旦发生数据节点变更，所有订阅的客户端能够得到通知，以便更新配置。

***案例: 配置变更***

在系统运行时，可能出现变更配置的情况，这时候可以直接在 zkclient 上或者通过其它客户端连接 zookeeper 变更数据，Zookeeper 会通知各个已添加数据变更监听的客户端。

***案例: 负载均衡***

通常负载均衡的方式有: LVS, Nginx 。但 Zookeeper 也可以实现。

***案例: 域名配置***

系统初始化之前，先在 ZooKeeper 上创建一个节点来进行域名配置，如：/dnsconfit/app1/server.app1.domain.com
节点内容是：
```
192.168.0.1:9999,192.168.0.2:9999,192.168.0.3:9999
```

***案例: 域名解析***

传统的DNS解析中，解析过程是操作系统和IP映射机制（本地 Host 绑定）或者专门的域名解析服务器完成的。但这里有很大的区别，它的过程由各个应用自己负责，ZooKeeper 里维护的是域名的解析数据。然后程序得到数据后可以通过相应的算法，解析为对应的IP。

当然，各个客户端通过ZooKeeper 得到域名数据后也得监听节点，以便得到数据变更的通知。

***案例: MySQL数据复制总线***

复制总线是一个实时数据复制框架，用在不同的 MySQL数据库实例间进行异步数据复制和数据变化的通知。整个系统由 ___MySQL 数据库集群___，___消息队列系统___，___任务管理监控平台___，___ZooKeeper 集群___等组件共同构成。

ZooKeeper 负责进行一系列的分布式协调工作。在应用中，根据功能将组件分成三个模块：Core, Server, Monitor，每个模块分别为一个单独的进程，通过 ZooKeeper 进行数据交换。

1. ___Core___: 实现数据复制的核心逻辑，将数据复制封装成管道，并抽象出生产者和消费者两个概念，生产者通常是 MySQL 的 Binlog 日志。
2. ___Server___: 负责启动和停止复制任务。
3. ___Monitor___: 监控任务的运行状态 ，如果复制期间发生异常或出现故障，进行报警。

每个模块在独立的进程中运行，运行时的数据和配置信息均保存在 ZooKeeper 上，WEB管理端通过 ZooKeeper 获取到后台进程的数据，同时发布控制信息。

1. 任务注册
Core进程启动时，向 /mysql_replicator/tasks 节点注册任务。如：复制热门商品的任务，Task 所在机器在启动的时候，会先在节点上注册如: /mysql_replicator/tasks/copy_hot_item 的节点。如果注册的时候发现该节点已经存在，表示有其它机器注册了该任务，就不用再执行了。

2. 任务热备份
为了应对复制任务故障，复制采用“热备份”的容灾方式。即：用同一个复制任务部署在不同的主机上。主、备任务机通过 ZooKeeper 互相检测运行健康状况。
如果是这样，注册任务时，就算检测到/mysql_replicator/tasks/copy_hot_item 节点已经存在 ，也要在它的下级把自己注册上去，如：/mysql_replicator/tasks/copy_hot_item/instances/[Hostname]_2
完成该子节点的创建后，每台任务机都可以获取到自己创建的节点以及所有的子节点列表。通过对比判断自己是否是所有节点中序号最小的，如果是，那么就将自己的运行状态设置为 Running, 其它机器将自己设置为 Standby ，这样依次来实现备份。

3. 热备切换
当任务机把自己设置为 Running 时，开始进行正常的数据复制，Standby 状态的机器进行等待。一旦标记为 Running 的机器出现故障，停止了任务，那么就需要在所有标记为 Standby 的客户端机器中再将按“最小序号优先”的策略来选出 Running 机器。
这就需要所有的 Standby 机器都需要在/mysql_replicator/tasks/copy_hot_item/instances/ 节点上注册一个 “子节点列表变更”的监听。一旦 Running机器挂了，与 ZooKeeper 断开连接后，对应的节点消失，其它机器会收到该变更通知，从而开始新一轮的 Running 选举。

4. 记录执行状态
被标记为 Running 的机器 需要记录运行时的状态 ，如：执行 Binlog 的位置。可以放在节点：/mysql_replicator/tasks/copy_hot_item/lastCommit/ 上。

5. 控制台协调
Server 组件是用来控制复制的启动和停止的。它会将每个复制任务的数据（库名，表名，用户名，密码等）写入节点 /mysql_replicator/tasks/copy_hot_item/ 中，以便 该任务所有机器都能得到该配置。

6. 冷备切换
上面的复制任务热备要求每个复制任务都有多台机器，这就形成了资源的浪费。所以这里出现了冷备切换。
所谓冷备切换其实就是用一些机器不停的扫描，看哪些任务没有任务节点。如果发现了，就立即创建自己的节点。

当然，这时可能有多个机器同时发现它，并都创建了自己的节点，这时候就需要各机器在创建后都进行一次序号的比较，如果序号是最小的，就将状态设置成 Running，并执行复制；其它的机器则删除自己刚才创建的节点，并继续扫描任务。

***冷热备份对比***

>可以看到热备份资源浪费严重，冷备份实时性差。

集群管理
---
所谓集群管理主要包括集群监控和集群控制两大块。前者要进行各机器状态的收集，后者要对集群进行操作与控制。通常系统会对集群有如下要求:
1. 知道当前集群中有多少台机器
2. 集群中各机器运行的状态数据收集
3. 对集群进行上下线操作

***案例:分布式日志收集系统***

该系统核心工作就是收集分布在不同机器上的系统日志。

系统的主要问题在于：
1. 日志源机器会变更。
2. 日志收集的机器变更。

>归根结底，我们的目标是：构建快速，合理，动态的日志收集系统。

1. 注册收集器机器
在 ZooKeeper 上创建一个节点，用来标记所有的收集机器：/logs/collector
每个收集机器启动时，会到该节点下注册，如：/logs/collector/[Hostname]

2. 任务分发
所有收集机器都创建好自己的节点后，系统根据子节点的个数，将所有日志源机器分成对应的若干组，然后将分组后的机器列表分别写到各收集机器节点下。
这样，各收集机器就能够从自己的节点上获取日志源机器列表，进行进行收集工作。

3. 状态汇报
任务注册和分发后，各机器开始收集。但这些机器可能随时都会挂掉。因为需要对收集状态进行监控，具体的做法是对每个收集节点下级创建一个 status 状态节点，收集器把进度信息写入该节点。

4. 动态分配
如果收集机器挂掉了，或者是要添加新的机器，就需要动态的对收集任务进行分配。
无论是 /logs/connector/节点下加入了新的节点，还是节点消失，日志系统都要进行任务的重新分配。

注意事项:
1. 各收集机器都是建立的临时节点，当机器挂掉后，节点会消失。但这时候，机器恢复的时候，它下面的日志源信息就消失了，任务无法恢复。除非再重新进行任务分配。但这对其它已经分配任务的机器有影响。所以，这里应该用持久节点，通过 status 下级节点去标记运行状态。
2. 各个收集节点实时更新自己的 status节点数据。系统定时轮询收集机器，查看各机器的状态。这里不用监听机制，因为状态的更新频率太高，如果用监听，会占用不少的流量。

分布式锁
---
>分布式锁是控制分布式系统之间同步访问共享资源的一种方式。不同系统或不同机器访问同一资源时，往往需要一些互斥手段来防止彼此之间的干扰，以保证一致性。

***排他锁***

又称写锁或独占锁。表示加上后，其它任何事务都不能再对这个资源进行任何操作。

定义锁:

可以创建一个节点用来表示某资源，然后再给它的下级创建一个 lock节点，表示已经锁定。

获取锁:

和前面选举 Master 类似，多个机器请求资源时，同时给资源节点下创建 lock 节点，最终只有一个机器成功，即它获得锁。没有获得到的节点需要监听该节点，以便在锁释放时再去获取。

释放锁:

获取锁时，创建的节点是临时节点。该节点消失时就表示锁释放。节点消失有两种情况：
1. 机器挂了，临时节点自动被删除。
2. 逻辑完成后，客户端主动删除节点。

>要注意的是，没有获取到锁的节点一定要监听锁节点，以便在锁释放时去获得锁。

***共享锁***

又称读锁。如果事务对资源加了锁，那么当前事务只能对该资源进行读取操作；其它事务也只能对资源加共享锁。只有当所有共享锁都释放后才能进行其它的操作，如加排他锁。

共享锁和排他锁的区别在于，加上排他锁后，数据只对一个事务可见；加上排他锁后数据对所有事务可见。

定义锁:

建立一个节点表示锁。如：/shared_lock/

获取锁:

在锁节点下创建自己的节点，如 /shared_lock/[Host]_1

判断读写顺序:

不同事务都可以对同一数据进行读取操作，但更新操作必须在数据上没有任何事务进行读写操作的情况下进行。所以，ZooKeeper 确定分布式读写的顺序可分为如下4个步骤：

1. 创建完节点后，获取 /shared_lock 节点下所有的节点，并对该节点注册子节点变更的监听。
2. 确定自己的节点序号在所有子节点中的顺序
3. 对于读请求，如果没有比自己序号更小的节点，或者比自己序号小的节点都是读请求，表示自己已经成功获得共享锁，开始读取逻辑。如果比自己序号小的节点中有写请求，需要等待。
对于写请求，如果自己不是序号最小的节点，进入等待。
4. 接收到监听的通知时，开始重复 1 操作。

释放锁:

同排他锁

分布式队列
---

***FIFO:先入先出***

>先进入队列的请求操作先完成。然后开始处理后面的请求。
>ZooKeeper处理步骤如下：

1. 所有客户端都在 /queue_fifo 下创建临时节点。
2. 创建完毕后，获得 /queue_fifo 下所有的子节点。
3. 确定自己的顺序。如果自己不是序号最小的，等待。同时对离自己最近的，比自己小的节点添加监听。
4.  接收到监听通知后，重复步骤 1。

***分布式屏障***

>指的是一个队列的元素必须都齐全后，才能统一进行安排，否则一直等待。这往往用在统计，合并计算的场景下。步骤如下:

1. 系统初始化前先创建根节点 /queue_barrier，节点数据为需要合并的数据和，如：10
2. 各个客户端在 /queue_barrier 下创建自己的临时节点，如 /queue_barrier/192.168.0.1
3. 获取当前所有的节点，得到个数；并获得根节点下需要的节点数。添加对子节点列表变更的监听。
4. 如果当前节点数达不到需求数，进入等待。
5. 接收到监听通知后，重复步骤 3。


php 示例主从
===

编辑 ```worker.php```

```php

<?php
ini_set('display_errors', '1');
error_reporting(E_ALL);

class Worker extends Zookeeper {
	const CONTAINER = '/cluster';
	protected $acl = array(
		array(
			'perms' => Zookeeper::PERM_ALL,
			'scheme' => 'world',
			'id' => 'anyone'
		)
	);
	private $isLeader = false;
	private $znode;

	public function __construct($host = '', $watcher_cb = null, $recv_timeout = 1000) {
		parent::__construct($host, $watcher_cb, $recv_timeout );
	}

	public function register() {
		if(!$this->exists(self::CONTAINER)) {
			print_r("Not Exists node. To Create! ".self::CONTAINER."\n");
			$this->create(self::CONTAINER, 1, $this->acl);
		}

		if($this->exists(self::CONTAINER)){
			print_r("Created Base Node Success! ".self::CONTAINER."\n");
			print_r("To Created Sub Node!\n");
			$this->znode = $this->create(self::CONTAINER.'/w-',
				1,
				$this->acl,
				Zookeeper::EPHEMERAL | Zookeeper::SEQUENCE
			);
		}else{
			print_r("Created Failed!\n");
		}

		$this->znode = str_replace(self::CONTAINER.'/', '', $this->znode);

		printf("I'm registred as: %s\n", $this->znode);

		$watching = $this->watchPrevious();

		if($watching == $this->znode) {
			printf("Reg: Nobody here, I'm the leader\n");
			$this->setLeader(true);
		}
		else {
			printf("Reg: I'm watching %s\n", $watching);
		}
	}

	public function watchPrevious() {
		$workers = $this->getChildren(self::CONTAINER);
		sort($workers);
		print_r($workers);
		$size = sizeof($workers);
		echo "Now Running ".$size." workers!\n";
		for( $i = 0 ; $i < $size ; $i++) {
			if($this->znode == $workers[$i]) {
				if($i > 0) {
					print_r("Ready to Watch! ".$workers[$i - 1]."\n");
					$this->get(self::CONTAINER.'/'.$workers[$i - 1], array($this, 'watchNode'));
					return $workers[$i - 1];
				}

				return $workers[$i];
			}
		}

		throw new Exception(sprintf("Something went very wrong! I can't find myself: %s/%s",
			self::CONTAINER,
			$this->znode)
		);
	}

	public function watchNode($i, $type, $name) {
		$watching = $this->watchPrevious();
		if($watching == $this->znode) {
			printf("Watch: I'm the new leader!\n");
			$this->setLeader( true );
		}else {
			printf("Watch: I'm %s\n", $watching);
		}
	}

	public function isLeader() {
		return $this->isLeader;
	}

	public function setLeader($flag) {
		$this->isLeader = $flag;
	}

	public function run() {
		$this->register();

		while(true) {
			if($this->isLeader() ) {
				$this->doLeaderJob();
			}
			else {
				$this->doWorkerJob();
			}

			sleep(2);
		}
	}

	public function doLeaderJob() {
		echo "Job: Leading\n";
	}

	public function doWorkerJob() {
		printf("Job: I'm %s\n", $this->znode );
	}

}

$worker = new Worker('localhost:2181');
$worker->run();
?>
```

打开多个终端都运行:

```bash
php worker.php
```

第一个终端:

```bash
root@dev-localhost # php worker.php
I'm registred as: w-0000000000
Nobody here, I'm the leader
Leading
Leading
```

第二个终端:

```bash
I'm registred as: w-0000000001
I'm watching w-0000000000
Working
Working
```

第三个终端:

```bash
I'm registred as: w-0000000002
I'm watching w-0000000001
Working
Working
```

现在模拟Leader崩溃的情形。使用Ctrl+c或其他方法退出第一个脚本。

刚开始不会有任何变化，worker可以继续工作。后来，ZooKeeper会发现超时，并选举出新的leader(未成功)。

第二个终端:

```bash
Now Running 2 workers!
Watch: I'm the new leader!
Job: Leading
Job: Leading
```

虽然这些脚本很容易理解，但是还是有必要对已使用的Zookeeper标志作注释:

```php
$this->znode = $this->create( self::CONTAINER . '/w-',
                              null,
                              $this->acl,
                              Zookeeper::EPHEMERAL | Zookeeper::SEQUENCE );
```
每个znode都是 EPHEMERAL 和 SEQUENCE 的。

常量 ```EPHEMRAL``` 代表当客户端失去连接时移除该znode。这就是为何PHP脚本会知道超时。

常量 ```SEQUENCE``` 代表在每个znode名称后添加顺序标识。我们通过这些唯一标识来标记worker。