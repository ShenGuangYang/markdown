# zookeeper 原理

## zookeeper 下载安装
开源软件，[官网](https://zookeeper.apache.org/)下载。


## zookeeper 使用
### zookeeper 简单测试
部署在linux 环境下
```shell
## 加压
tar -zxvf zookeeper-3.4.14.tar.gz
## 切换到配置文件
cd zookeeper-3.4.14/conf/
## 复制核心配置文件信息
cp zoo_sample.cfg zoo.cfg
## 切换到bin目录下
cd ../bin/
## 启动zkServer
sh zkServer.sh start
## 启动zkCli.sh
sh zhCli.sh
```

### zookeeper 集群搭建

1. 配置核心zoo.cfg文件
```shell
vi conf/zoo.cfg

## 加上以下内容
server.1=192.168.1.200:2888:3888
server.2=192.168.1.201:2888:3888
server.3=192.168.1.202:2888:3888
```
2. 在data目录下创建myid（myid一点要和上面的id对应起来）
每一台服务器对应执行
```shell
vi /tmp/zookeeper/myid
## 对应加上id
1
2
3
```


## zookeeper 相关知识

### zookeeper 节点特性
1. 同级节点唯一性
2. 持久化节点和临时节点
3. 有序节点和无序节点
4. 临时节点下不能存在子节点

### zookeeper 配置文件详解
tickTime 定义时间单位规则 2s
initLimit 单位* tickTIme
syncLimit 同步时间 单位* tickTIme
dataDir 持久化存储数据的地址
clientPort 客户端与服务端通信的端口

### ZAB 协议
支持崩溃恢复的原子广播协议、主要用于实现数据一致性

zxid组成：低32位表示消息计数器（自增），高32位（epoch编号，选举后发生改变）
#### 消息广播（改进版的2PC）
1. 如果是读请求，可以在任意节点去处理
2. 如果是写请求，那这个请求会转发给leader去处理
   1. 对每个消息生成一个zxid
   2. 把带有zxid的消息作为一个提案分发给集群中的每一个节点
   3. 把提案的这个事务写入到磁盘，返回ask
   4. leader收到合法数量的ask请求以后，再进行commit请求

#### 奔溃恢复
1. 当leader失去了过半的follower节点的联系
2. 当leader服务挂了
集群就会进入奔溃恢复阶段
对数据恢复来说
1. 已经被处理的消息不能丢失
   当leader收到合法的follower的ask以后，就会向各个follower节点进行广播（commit命令），同时自己也会commit这条事务，如果follower节点收到commit命令之前，leader挂了，会导致部分节点收到commit，部分节点收不到。***那么zab协议需要保证已经被处理的消息不能丢失。***
2. 被丢弃的消息不能再次出现
   当leader收到事务请求，并且还未发起事务投票之前，leader挂了。


### leader 选举机制

zxid最大会设置为leader （事务id，越大表示越新）
epoch（每一轮投票，epoch都会递增）
myid（服务器id）【myid越大，在leader选举机制中权重越大】

#### 选举算法

　1. 服务器启动时期的Leader选举
　　若进行Leader选举，则至少需要两台机器，这里选取3台机器组成的服务器集群为例。在集群初始化阶段，当有一台服务器Server1启动时，其单独无法进行和完成Leader选举，当第二台服务器Server2启动时，此时两台机器可以相互通信，每台机器都试图找到Leader，于是进入Leader选举过程。选举过程如下
　　(1) 每个Server发出一个投票。由于是初始情况，Server1和Server2都会将自己作为Leader服务器来进行投票，每次投票会包含所推举的服务器的myid和ZXID，使用(myid, ZXID)来表示，此时Server1的投票为(1, 0)，Server2的投票为(2, 0)，然后各自将这个投票发给集群中其他机器。
　　(2) 接受来自各个服务器的投票。集群的每个服务器收到投票后，首先判断该投票的有效性，如检查是否是本轮投票、是否来自LOOKING状态的服务器。
　　(3) 处理投票。针对每一个投票，服务器都需要将别人的投票和自己的投票进行PK，PK规则如下
   * 优先检查ZXID。ZXID比较大的服务器优先作为Leader。
   * 如果ZXID相同，那么就比较myid。myid较大的服务器作为Leader服务器。
　　对于Server1而言，它的投票是(1, 0)，接收Server2的投票为(2, 0)，首先会比较两者的ZXID，均为0，再比较myid，此时Server2的myid最大，于是更新自己的投票为(2, 0)，然后重新投票，对于Server2而言，其无须更新自己的投票，只是再次向集群中所有机器发出上一次投票信息即可。
　　(4) 统计投票。每次投票后，服务器都会统计投票信息，判断是否已经有过半机器接受到相同的投票信息，对于Server1、Server2而言，都统计出集群中已经有两台机器接受了(2, 0)的投票信息，此时便认为已经选出了Leader。
　　(5) 改变服务器状态。一旦确定了Leader，每个服务器就会更新自己的状态，如果是Follower，那么就变更为FOLLOWING，如果是Leader，就变更为LEADING。

　　2. 服务器运行时期的Leader选举

　　在Zookeeper运行期间，Leader与非Leader服务器各司其职，即便当有非Leader服务器宕机或新加入，此时也不会影响Leader，但是一旦Leader服务器挂了，那么整个集群将暂停对外服务，进入新一轮Leader选举，其过程和启动时期的Leader选举过程基本一致。假设正在运行的有Server1、Server2、Server3三台服务器，当前Leader是Server2，若某一时刻Leader挂了，此时便开始Leader选举。选举过程如下
　　(1) 变更状态。Leader挂后，余下的非Observer服务器都会讲自己的服务器状态变更为LOOKING，然后开始进入Leader选举过程。
　　(2) 每个Server会发出一个投票。在运行期间，每个服务器上的ZXID可能不同，此时假定Server1的ZXID为123，Server3的ZXID为122；在第一轮投票中，Server1和Server3都会投自己，产生投票(1, 123)，(3, 122)，然后各自将投票发送给集群中所有机器。
　　(3) 接收来自各个服务器的投票。与启动时过程相同。
　　(4) 处理投票。与启动时过程相同，此时，Server1将会成为Leader。
　　(5) 统计投票。与启动时过程相同。
　　(6) 改变服务器的状态。与启动时过程相同。
　　2.2 Leader选举算法分析

　　在3.4.0后的Zookeeper的版本只保留了TCP版本的FastLeaderElection选举算法。当一台机器进入Leader选举时，当前集群可能会处于以下两种状态

* 集群中已经存在Leader。
* 集群中不存在Leader。

　　对于集群中已经存在Leader而言，此种情况一般都是某台机器启动得较晚，在其启动之前，集群已经在正常工作，对这种情况，该机器试图去选举Leader时，会被告知当前服务器的Leader信息，对于该机器而言，仅仅需要和Leader机器建立起连接，并进行状态同步即可。而在集群中不存在Leader情况下则会相对复杂，其步骤如下

　　(1) 第一次投票。无论哪种导致进行Leader选举，集群的所有机器都处于试图选举出一个Leader的状态，即LOOKING状态，LOOKING机器会向所有其他机器发送消息，该消息称为投票。投票中包含了SID（服务器的唯一标识）和ZXID（事务ID），(SID, ZXID)形式来标识一次投票信息。

　　(2) 变更投票。每台机器发出投票后，也会收到其他机器的投票，每台机器会根据一定规则来处理收到的其他机器的投票，并以此来决定是否需要变更自己的投票，这个规则也是整个Leader选举算法的核心所在，其中术语描述如下
* vote_sid：接收到的投票中所推举Leader服务器的SID。
* vote_zxid：接收到的投票中所推举Leader服务器的ZXID。
* self_sid：当前服务器自己的SID。
* self_zxid：当前服务器自己的ZXID。
　　每次对收到的投票的处理，都是对(vote_sid, vote_zxid)和(self_sid, self_zxid)对比的过程。

(3) 确定Leader。经过第二轮投票后，集群中的每台机器都会再次接收到其他机器的投票，然后开始统计投票，如果一台机器收到了超过半数的相同投票，那么这个投票对应的SID机器即为Leader。此时Server3将成为Leader。

　　由上面规则可知，通常那台服务器上的数据越新（ZXID会越大），其成为Leader的可能性越大，也就越能够保证数据的恢复。如果ZXID相同，则SID越大机会越大。 



### 集群搭建为什么需要奇数节点（2n+1）
内部一个投票机制，需要过半的机器正常工作，彼此通信完成投票结果。

# zookeeper 实践

## 有效的连接zookeeper
```java
public static void main(String[] args) throws Exception {
    final CountDownLatch countDownLatch = new CountDownLatch(1);
    ZooKeeper zooKeeper = new ZooKeeper("192.168.1.200:2181", 4000,
            (WatchedEvent event)->{
                //连接状态有很多种（CONNECTING, CONNECTED,CLOSED, NOT_CONNECTED），判断必须CONNECTED才能操作zookeeper
                if (Watcher.Event.KeeperState.SyncConnected == event.getState()) {
                    countDownLatch.countDown();
                }
            });
    countDownLatch.await();
    System.out.println(zooKeeper.getState());
    zooKeeper.close();
}
```
## zookeeper 增删改的操作

```java
public static void main(String[] args) throws Exception {
    final CountDownLatch countDownLatch = new CountDownLatch(1);
    ZooKeeper zooKeeper = new ZooKeeper("192.168.1.200:2181", 20000000,
            (WatchedEvent event) -> {
                if (Watcher.Event.KeeperState.SyncConnected == event.getState()) {
                    countDownLatch.countDown();
                }
            });
    countDownLatch.await();
    System.out.println(zooKeeper.getState());
    // 添加节点
    zooKeeper.create("/zk-demo", "abc".getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
    // 获取节点
    Stat stat = new Stat();
    byte[] data = zooKeeper.getData("/zk-demo", null, stat);
    System.out.println(new String(data));

    // 修改节点
    zooKeeper.setData("/zk-demo", "ef".getBytes(), stat.getVersion());
    byte[] data1 = zooKeeper.getData("/zk-demo", null, stat);
    System.out.println(new String(data1));

    // 删除节点
    zooKeeper.delete("/zk-demo", stat.getVersion());

    zooKeeper.close();
}
```


## 事件机制

watcher特性:当数据发生变化的时候，zookeeper 会产生一个watcher事件，并会发送到客户端。但是客户端只会受到一次通知。如果后续这个节点再次发生变化，那么之前设置watcher不会再次受到消息。（watcher是一次性的）。可以循环监听这个事件达到永久监听效果。

### 如何注册事件机制
通过这三个操作来绑定事件：getData、Exist、getChilren

如何触发事件？凡是事务类型的操作，都会触发监听事件。create、delete、setData

### watcher 事件类型：
None (-1),                客户端连接状态发生变化的时候，会受到none的事件
NodeCreated (1),          创建节点的事件
NodeDeleted (2),          删除节点的事件
NodeDataChanged (3),      节点数据发生变化
NodeChildrenChanged (4);  子节点被创建、被删除、被修改



## 使用apche curator 操作zookeeper

### 简单操作增删改
```java
public static void main(String[] args) throws Exception {

    //连接客户端
    CuratorFramework curatorFramework = CuratorFrameworkFactory.builder()
            .connectString("192.168.1.200:2181").sessionTimeoutMs(20000)
            .retryPolicy(new ExponentialBackoffRetry(100, 3))
            .namespace("curator").build();
    //启动
    curatorFramework.start();
    //创建
    curatorFramework.create().creatingParentsIfNeeded().withMode(CreateMode.PERSISTENT)
            .forPath("/shen/test1", "shen".getBytes());
    //获取数据
    Stat stat = new Stat();
    curatorFramework.getData().storingStatIn(stat).forPath("/shen/test1");
    // 修改数据
    curatorFramework.setData().withVersion(stat.getVersion())
            .forPath("/shen/test1", "test".getBytes());
    //删除数据
    //删除数据
    curatorFramework.delete().deletingChildrenIfNeeded()
            .forPath("/shen/test1");
    curatorFramework.close();
}
```

### watcher监听机制的使用

1. PathChildrenCache 监听一个节点下子节点的创建、删除、更新
2. NodeCache 监听一个节点的更新和创建事件
3. TreeCache 综合PathChildCache和NodeCache的特性

```java
public static void main(String[] args) throws Exception {

    //连接客户端
    CuratorFramework curatorFramework = CuratorFrameworkFactory.builder()
            .connectString("192.168.1.200:2181").sessionTimeoutMs(200000000)
            .retryPolicy(new ExponentialBackoffRetry(100, 3))
            .namespace("curator").build();
    curatorFramework.start();
//        addListenWithNodeCache(curatorFramework, "/shen");
//        addListenWithPathChildrenCache(curatorFramework, "/shen");
    addListenWithTreeCache(curatorFramework, "/shen");
    System.in.read();
}

//监听一个节点的更新和创建事件
public static void addListenWithNodeCache(CuratorFramework curatorFramework, String path) throws Exception {
    final NodeCache nodeCache = new NodeCache(curatorFramework, path, false);
    NodeCacheListener nodeCacheListener =
            () -> System.out.println("Event： " + nodeCache.getCurrentData().getPath());
    nodeCache.getListenable().addListener(nodeCacheListener);
    nodeCache.start();
}

//监听一个节点下子节点的创建、删除、更新
public static void addListenWithPathChildrenCache(CuratorFramework curatorFramework, String path) throws Exception {
    final PathChildrenCache pathChildrenCache = new PathChildrenCache(curatorFramework, path, true);

    PathChildrenCacheListener pathChildrenCacheListener =
            (client, event) -> System.out.println("Event: " + event.getType());

    pathChildrenCache.getListenable().addListener(pathChildrenCacheListener);
    pathChildrenCache.start(PathChildrenCache.StartMode.NORMAL);
}
//综合PathChildrenCache和NodeCache的特性
public static void addListenWithTreeCache(CuratorFramework curatorFramework,String path) throws Exception {
    final TreeCache treeCache = new TreeCache(curatorFramework, path);
    TreeCacheListener treeCacheListener =
            (client, event) -> System.out.println("Event: " + event.getType() + "->" + event.getData().getPath());

    treeCache.getListenable().addListener(treeCacheListener);
    treeCache.start();
}

```


