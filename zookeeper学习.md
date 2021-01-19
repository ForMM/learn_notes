# zookeeper学习

### zookeeper使用场景

1. 统一配置：把配置放在ZooKeeper的节点中维护，当配置变更时，客户端可以收到变更的通知，并应用最新的配置。
2. 集群管理：集群中的节点，创建ephemeral的节点，一旦断开连接，ephemeral的节点会消失，其它的集群机器可以收到消息。
3. 分布式锁：多个客户端发起节点创建操作，只有一个客户端创建成功，从而获得锁。

### zookeeper特性

1. 强一致性

   每个server保存一份相同的数据副本，client无论连接到哪个server，数据都是一致的

2. 可靠性

   具有简单的，健壮，良好的性能，如果消息Message被一台服务器接收，那么就会被所有的服务器进行接收（每个服务器都和leader做了数据同步，而zookeeper是使用一致性来保持数据的一致性，以及维护视图的特性，由此来进行理解。）

3. 实时性

   zookeeper在保证客户端在一个时间间隔范围里面内获得服务器的更新信息。

4. 原子性

   数据要么更新失败，要么更新成功，没有中间状态，也不会去产生脏数据。

5. 顺序性

   来自同一个client的更新请求按其发送顺序依次执行

### zookeeper是什么

zookeeper是一个分布式协调服务，可以实现统一命名服务、状态同步服务、集群管理、分布式应用配置项的管理等。简单点说zookeeper=文件系统+监听通知服务。

- 文件系统

  ![zookeeper-1](./img/zookeeper-1.png)

  每个子目录项如 NameService 都被称作为 znode(目录节点)，和文件系统一样，我们能够自由的增加、删除znode，在一个znode下增加、删除子znode，唯一的不同在于znode是可以存储数据的。

  1. Znode有两种类型：

     短暂（ephemeral）（断开连接自己删除）

     持久（persistent）（断开连接不删除）

  2. Znode有四种形式的目录节点（默认是persistent ）

     - **PERSISTENT-持久化目录节点**

       客户端与zookeeper断开连接后，该节点依旧存在

     - **PERSISTENT_SEQUENTIAL-持久化顺序编号目录节点**

       客户端与zookeeper断开连接后，该节点依旧存在，只是Zookeeper给该节点名称进行顺序编号

     - **EPHEMERAL-临时目录节点**

       客户端与zookeeper断开连接后，该节点被删除

     - **EPHEMERAL_SEQUENTIAL-临时顺序编号目录节点**

       客户端与zookeeper断开连接后，该节点被删除，只是Zookeeper给该节点名称进行顺序编号

  3. 

- 监听通知机制

  ZooKeeper提供的接口使得所有的分布式进程的执行都是异步非阻塞的（WaitFree算法），内部是基于Version的CAS操作。

  客户端注册监听它关心的目录节点，当目录节点发生变化（数据改变、被删除、子目录节点增加删除）时，zookeeper会通知客户端。

  **zookeeper客户端和服务端的连接状态**

  1. KeeperState.Expired：客户端和服务器在ticktime的时间周期内，是要发送心跳通知的。这是租约协议的一个实现。客户端发送request，告诉服务器其上一个租约时间，服务器收到这个请求后，告诉客户端其下一个租约时间是哪个时间点。当客户端时间戳达到最后一个租约时间，而没有收到服务器发来的任何新租约时间，即认为自己下线（此后客户端会废弃这次连接，并试图重新建立连接）。这个过期状态就是Expired状态
  2. KeeperState.Disconnected：当客户端断开一个连接（可能是租约期满，也可能是客户端主动断开）这是客户端和服务器的连接就是Disconnected状态
  3. KeeperState.SyncConnected：一旦客户端和服务器的某一个节点建立连接（注意，虽然集群有多个节点，但是客户端一次连接到一个节点就行了），并完成一次version、zxid的同步，这时的客户端和服务器的连接状态就是SyncConnected
  4. KeeperState.AuthFailed：zookeeper客户端进行连接认证失败时，发生该状态

  **zookeeper的watch事件**

  | zookeeper事件                 | 事件含义                                                     |
  | ----------------------------- | ------------------------------------------------------------ |
  | EventType.NodeCreated         | 当node-x这个节点被创建时，该事件被触发                       |
  | EventType.NodeChildrenChanged | 当node-x这个节点的直接子节点被创建、被删除、子节点数据发生变更时，该事件被触发。 |
  | EventType.NodeDataChanged     | 当node-x这个节点的数据发生变更时，该事件被触发               |
  | EventType.NodeDeleted         | 当node-x这个节点被删除时，该事件被触发。                     |
  | EventType.None                | 当zookeeper客户端的连接状态发生变更时，即KeeperState.Expired、KeeperState.Disconnected、KeeperState.SyncConnected、KeeperState.AuthFailed状态切换时，描述的事件类型为EventType.None |

  ![zookeeper-2](.\img\zookeeper-2.png)

Znode发生变化（Znode本身的增加，删除，修改，以及子Znode的变化）可以通过Watch机制通知到客户端。那么要实现Watch，就必须实现org.apache.zookeeper.Watcher接口，并且将实现类的对象传入到可以Watch的方法中。Zookeeper中所有读操作（getData()，getChildren()，exists()）都可以设置Watch选项。Watch事件具有one-time trigger（一次性触发）的特性，如果Watch监视的Znode有变化，那么就会通知设置该Watch的客户端。

zk提供的原语包含：

1. create
2. delete
3. exists
4. get data
5. set data
6. get chiledren
7. sync

### zookeeper单机模式安装

~~~shell
wget http://mirror.bit.edu.cn/apache/zookeeper/stable/apache-zookeeper-3.5.8-bin.tar.gz
tar -zxvf apache-zookeeper-3.5.8-bin.tar.gz 
cp ./conf/zoo_sample.cfg ./conf/zoo.cfg
bin/zkServer.sh start
./zkServer.sh status

#检测是否成功启动，用zookeeper客户端连接下服务端
./zkCli.sh

#使用客户端命令操作zookeeper

#使用 ls 命令来查看当前 ZooKeeper 中所包含的内容
ls /
#创建一个新的 znode ，使用 create /zkPro myData
create /zkPro/myData
#通过 set 命令来对 zk 所关联的字符串进行设置
set /zkPro/myData 1111
#将刚才创建的 znode 删除
delete /zkPro/myData

~~~

配置说明：

- tickTime：这个时间是作为 Zookeeper 服务器之间或客户端与服务器之间维持心跳的时间间隔，也就是每个 tickTime 时间就会发送一个心跳。
- initLimit：这个配置项是用来配置 Zookeeper 接受客户端（这里所说的客户端不是用户连接 Zookeeper 服务器的客户端，而是 Zookeeper 服务器集群中连接到 Leader 的 Follower 服务器）初始化连接时最长能忍受多少个心跳时间间隔数。当已经超过 10个心跳的时间（也就是 tickTime）长度后 Zookeeper 服务器还没有收到客户端的返回信息，那么表明这个客户端连接失败。总的时间长度就是 10*2000=20 秒
- syncLimit：这个配置项标识 Leader 与 Follower 之间发送消息，请求和应答时间长度，最长不能超过多少个 tickTime 的时间长度，总的时间长度就是 5*2000=10秒
- dataDir：顾名思义就是 Zookeeper 保存数据的目录，默认情况下，Zookeeper 将写数据的日志文件也保存在这个目录里。
- clientPort：这个端口就是客户端连接 Zookeeper 服务器的端口，Zookeeper 会监听这个端口，接受客户端的访问请求。
- server.A=B：C：D：其中 A 是一个数字，表示这个是第几号服务器；B 是这个服务器的 ip 地址；C 表示的是这个服务器与集群中的 Leader 服务器交换信息的端口；D 表示的是万一集群中的 Leader 服务器挂了，需要一个端口来重新进行选举，选出一个新的 Leader，而这个端口就是用来执行选举时服务器相互通信的端口。如果是伪集群的配置方式，由于 B 都是一样，所以不同的 Zookeeper 实例通信端口号不能一样，所以要给它们分配不同的端口号。

### zookeeper集群模式安装

~~~shell
wget http://mirror.bit.edu.cn/apache/zookeeper/stable/apache-zookeeper-3.5.8-bin.tar.gz
tar -zxvf apache-zookeeper-3.5.8-bin.tar.gz 

#重命名 zoo_sample.cfg文件
cp conf/zoo_sample.cfg conf/zoo-1.cfg

#修改配置文件zoo-1.cfg，原配置文件里有的，修改成下面的值，没有的则加上

# vim conf/zoo-1.cfg
dataDir=/tmp/zookeeper-1
clientPort=2181
server.1=127.0.0.1:2888:3888
server.2=127.0.0.1:2889:3889
server.3=127.0.0.1:2890:3890

#其他节点也要配置这些，只需修改dataDir和clientPort不同即可
# cp conf/zoo-1.cfg conf/zoo-2.cfg
# cp conf/zoo-1.cfg conf/zoo-3.cfg
# vim conf/zoo-2.cfg
dataDir=/tmp/zookeeper-2
clientPort=2182
# vim conf/zoo-2.cfg
dataDir=/tmp/zookeeper-3
clientPort=2183

#标识server id
创建三个文件夹/tmp/zookeeper-1，/tmp/zookeeper-2，/tmp/zookeeper-2，在每个目录中创建文件myid 文件，写入当前实例的server id，即1.2.3

# cd /tmp/zookeeper-1
# vim myid
1
# cd /tmp/zookeeper-2
# vim myid
2
# cd /tmp/zookeeper-3
# vim myid

#启动三个zookeeper实例
# bin/zkServer.sh start conf/zoo-1.cfg
# bin/zkServer.sh start conf/zoo-2.cfg
# bin/zkServer.sh start conf/zoo-3.cfg

#检测集群状态，也可以直接用命令“zkCli.sh -server IP:PORT”连接zookeeper服务端检测
 ./zookeeper2/bin/zkServer.sh status ./zookeeper2/conf/zoo2.cfg
 
 ZooKeeper JMX enabled by default
Using config: ./zookeeper2/conf/zoo2.cfg
Client port found: 2182. Client address: localhost.
Mode: leader

~~~

**注意：**

**1.在集群模式下，建议至少部署3个zk进程，或者部署奇数个zk进程。**

**2.半数机制，集群中半数以上的机器存活，集群可用。**

### zookeeper主从同步机制

zookeeper的读写机制，提供一个非锁机制的Wait Free的用于分布式系统同步的核心服务。提供简单的文件创建、读写操作接口，其系统核心本身对**文件读写**并不提供加锁互斥的服务，但是提供**基于版本比对的更新操作**，客户端可以基于此自己实现加锁逻辑。

ZK集群中每个Server，都保存一份数据副本。Zookeeper使用简单的同步策略，通过以下两条基本保证来实现数据的一致性：

① 全局**串行化**所有的**写操作**

② 保证**同一客户**端的指令被FIFO执行（以及消息通知的FIFO）

所有的读请求由Zk Server 本地响应，所有的更新请求将转发给Leader，由Leader实施。

![zookeeper-3](./img/zookeeper-3.png)

（ZAB）**广播模式**ZooKeeper Server会接受Client请求，所有的写请求都被转发给**领导者**，再由领导者将更新广播给**跟随者**。当半数以上的跟随者已经将修改**持久化**之后，领导者才会提交这个更新，然后客户端才会收到一个更新成功的响应。这个用来达成共识的协议被设计成具有原子性，因此每个修改要么成功要么失败。

![zookeeper-4](./img/zookeeper-4.png)

ZAB协议的消息广播使用原子广播协议， **类似一个二阶段提交的过程** ，但又有所不同。

1. 二阶段提交中，需要所有参与者反馈ACK后再发送Commit请求。要求所有参与者要么成功，要么失败。这样会产生严重的阻塞问题
2. ZAB协议中，Leader等待半数以上的Follower成功反馈ACK即可，不需要收到全部的Follower反馈ACK。

**消息广播过程：**

1. 客户端发起写请求
2. Leader将客户端请求信息转化为事务Proposal，同时为每个Proposal分配一个事务ID（Zxid）
3. Leader为每个Follower单独分配一个FIFO的队列，将需要广播的Proposal依次放入到队列中
4. Follower接收到Proposal后，首先将其以事务日志的方式写入到本地磁盘中，写入成功后给Leader反馈一个ACK响应
5. Leader接收到半数以上Follower的ACK响应后，即认为消息发送成功，可以发送Commit消息
6. Leader向所有Follower广播Commit消息，同时自身也会完成事务提交。Follower接收到Commit消息后也会完成事务的提交

https://www.cnblogs.com/crazylqy/p/7132133.html

### zookeeper内部选举机制

leader节点作用：

1. 处理所有的写请求并同步给Follower
2. 启动时同步数据给Follewer节点 

**初次选举**

在集群初始化阶段，当有一台服务器Server1启动时，其单独无法进行和完成Leader选举，当第二台服务器Server2启动时，

两台机器此时可以相互通信，每台机器都试图找到Leader，于是进入选举过程。

　　选举过程如下：

　　（1）每个Server发出一个投票，由于是初始情况，Server1和server2都会将自己作为Leader服务器来进行投票。

每台服务器会往其他服务器发送投票信息，这个投票信息包括了SID和ZXID，其中SID就是该台机器的唯一标识（myid）;

ZXID是事务id，该ID是64位的，分为高32位和低32位。

　　（2）由于是初次投票，此时的ZXID相同，所以比较的就是SID，SID越大，获得的Leader的可能越大（为了严谨，

本文针对任何情况都只说可能，不说绝对）。

　　（3）两台服务器发出自己的投票信息后，再根据自己收到的其他服务器的投票信息决定自己的投票信息是否变更，第一台服务器SID为1，第二台服务器SID为2，所以Server2的投票变更为2，

即有两票，由于一共三台服务器，此时Server2已经处于半数以上，所以决定出来的Leader为Server2；（半数投票）即使Server3启动，

由于Leader已经决定出来，所以不需要在进行投票，Server3只需要与Leader建立连接并进行状态同步即可。

**Leader突然宕机，重新选举**

服务器具有四种状态，分别是LOOKING、FOLLOWING、LEADING、OBSERVING。

**LOOKING**：寻找Leader状态。当服务器处于该状态时，它会认为当前集群中没有Leader，因此需要进入Leader选举状态。

**FOLLOWING**：跟随者状态。表明当前服务器角色是Follower。

**LEADING**：领导者状态。表明当前服务器角色是Leader。

**OBSERVING**：观察者状态。表明当前服务器角色是Observer。

快速选举的完成只需要几百毫秒。

https://www.cnblogs.com/liuqijia/p/11456106.html

### 分布式锁的实现

1. zookeeper的每一个节点都是一个天然的顺序信号发送器。在每一个节点下面创建子节点时，只要选择的创建类型是有序（EPHEMERAL_SEQUENTIAL 临时有序或者PERSISTENT_SEQUENTIAL 永久有序）类型，那么，新的子节点后面，会加上一个次序编号。这个次序编号，是上一个生成的次序编号加一。

2. zookeeper节点递增性，可以规定节点编号最小的那个获取锁。一个zookeeper分布式锁，首先需要创建一个父节点，尽量是持久节点（PERSISTENT类型），然后每个要获得锁的线程都会在这个节点下创建个临时顺序节点，由于序号的递增性，可以规定排号最小的那个获得锁。所以，每个线程在尝试占用锁之前，首先判断自己是排号是不是当前最小，如果是，则获取锁。

3. zookeeper节点监听机制，可以保障占有锁的方式有序而且高效。每个线程抢占锁之前，先抢号创建自己的ZNode。同样，释放锁的时候，就需要删除抢号的Znode。抢号成功后，如果不是排号最小的节点，就处于等待通知的状态。等谁的通知呢？不需要其他人，只需要等前一个Znode 的通知就可以了。当前一个Znode 删除的时候，就是轮到了自己占有锁的时候。

   Zookeeper的内部机制，能保证后面的节点能够正常的监听到删除和获得锁。在创建取号节点的时候，尽量创建临时znode 节点而不是永久znode 节点，一旦这个 znode 的客户端与Zookeeper集群服务器失去联系，这个临时 znode 也将自动删除。排在它后面的那个节点，也能收到删除事件，从而获得锁。

   具体实现方式可以回顾工程：Apache curator的使用及zk分布式锁实现，Apache开源的curator的使用,有了curator,利用Java对zookeeper的操作变得极度便捷。

   https://github.com/yujiasun/Distributed-Kit/blob/master/src/main/java/com/distributed/lock/zk/ZkReentrantLock.java
   
   https://www.cnblogs.com/windpoplar/p/11964314.html

### 分布式id的实现

​	可以通过通过节点版本号实现；