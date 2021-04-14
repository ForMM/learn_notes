# Redis

### redis为什么效率这么高,可以支持10万QPS

1. 纯内存访问：数据存放在内存中，内存的响应时间大约是100纳秒，这是Redis每秒万亿级别访问的重要基础。
2. 非阻塞I/O：Redis采用epoll做为I/O多路复用技术的实现，再加上Redis自身的事件处理模型将epoll中的连接，读写，关闭都转换为了时间，不在I/O上浪费过多的时间。
3. 单线程避免了线程切换和竞态产生的消耗。
4. Redis采用单线程模型，每条命令执行如果占用大量时间，会造成其他线程阻塞，对于Redis这种高性能服务是致命的，所以Redis是面向高速执行的数据库

### redis的内部数据结构

第一个层面，是从使用者的角度。比如：

- string
- list
- hash
- set
- sorted set

这一层面也是Redis暴露给外部的调用接口。

第二个层面，是从内部实现的角度，属于更底层的实现。比如：

- dict

  dict是一个用于维护key和value映射关系的数据结构，与很多语言中的Map或dictionary类似。Redis的一个database中所有key到value的映射，就是使用一个dict来维护的。不过，这只是它在Redis中的一个用途而已，它在Redis中被使用的地方还有很多。比如，一个Redis hash结构，当它的field较多时，便会采用dict来存储。再比如，Redis配合使用dict和skiplist来共同维护一个sorted set。

  dict本质上是为了解决算法中的查找问题（Searching），一般查找问题的解法分为两个大类：一个是基于各种平衡树，一个是基于哈希表。我们平常使用的各种Map或dictionary，大都是基于哈希表实现的。在不要求数据有序存储，且能保持较低的哈希值冲突概率的前提下，基于哈希表的查找性能能做到非常高效，接近O(1)，而且实现简单。

  在Redis中，dict也是一个基于哈希表的算法。和传统的哈希算法类似，它采用某个哈希函数从key计算得到在哈希表中的位置，采用拉链法解决冲突，并在装载因子（load factor）超过预定值时自动扩展内存，引发重哈希（rehashing）。Redis的dict实现最显著的一个特点，就在于它的重哈希。它采用了一种称为增量式重哈希（incremental rehashing）的方法，在需要扩展内存时避免一次性对所有key进行重哈希，而是将重哈希操作分散到对于dict的各个增删改查的操作中去。这种方法能做到每次只对一小部分key进行重哈希，而每次重哈希之间不影响dict的操作。dict之所以这样设计，是为了避免重哈希期间单个请求的响应时间剧烈增加，这与前面提到的“快速响应时间”的设计原则是相符的。

- sds

  - 可动态扩展内存。sds表示的字符串其内容可以修改，也可以追加。在很多语言中字符串会分为mutable和immutable两种，显然sds属于mutable类型的。
  - 二进制安全（Binary Safe）。sds能存储任意二进制数据，而不仅仅是可打印字符。

- ziplist

  - ziplist是一个经过特殊编码的双向链表，它的设计目标就是为了提高存储效率。ziplist可以用于存储字符串或整数，其中整数是按真正的二进制表示进行编码的，而不是编码成字符串序列。它能以O(1)的时间复杂度在表的两端提供`push`和`pop`操作。

- quicklist

- skiplist

  - skiplist是一种跳跃表结构，用于有序集合中快速查找，大多数情况下它的效率与平衡树差不多，但比平衡树实现简单。redis的作者对普通的跳跃表进行了修改，包括添加span\tail\backward指针、score的值可重复这些设计，从而实现排序功能和反向遍历的功能。

    一般跳跃表的实现，主要包含以下几个部分：

    表头（head）：指向头节点
    表尾（tail）：指向尾节点
    节点（node）：实际保存的元素节点，每个节点可以有多层，层数是在创建此节点的时候随机生成的一个数值，而且每一层都是一个指向后面某个节点的指针。
    层（level）：目前表内节点的最大层数
    长度（length）：节点的数量。
    跳跃表的遍历总是从高层开始，然后随着元素值范围的缩小，慢慢降低到低层。

    跳跃表的实现原理可以参考：https://blog.csdn.net/Acceptedxukai/article/details/17333673

hash底层的数据结构实现有两种：

一种是ziplist，上面已经提到过。当存储的数据超过配置的阀值时就是转用hashtable的结构。这种转换比较消耗性能，所以应该尽量避免这种转换操作。同时满足以下两个条件时才会使用这种结构：
当键的个数小于hash-max-ziplist-entries（默认512）
当所有值都小于hash-max-ziplist-value（默认64）
另一种就是hashtable。这种结构的时间复杂度为O(1)，但是会消耗比较多的内存空间。

### redis线程模型

- **多个 socket**
- **IO 多路复用程序**
- **文件事件分派器**
- **事件处理器（包括：连接应答处理器、命令请求处理器、命令回复处理器）**

<img src=".\img\redis-1.webp" alt="redis-1" style="zoom:200%;" />

多个 socket 可能会并发产生不同的操作，每个操作对应不同的文件事件，但是 IO 多路复用程序会监听多个 socket，会将 socket 产生的事件放入队列中排队，事件分派器每次从队列中取出一个事件，把该事件交给对应的事件处理器进行处理。

### redis的I/O多路复用技术：epoll()

 epoll通过在Linux内核中申请一个简易的文件系统（文件系统一般用什么数据结构？红黑树）。把原先的select/poll调用分成三个部分。

1. 调用epoll_create（）建立一个epoll对象（在epoll文件系统中为这个句柄对象分配资源）；
2. 调用epoll_ctl向epoll对象中添加这100W个连接的套接字；
3. 调用epoll_wait收集在这上面发生的事件连接。

  只要在进程启动的时候创建一个epoll对象，然后在需要的时候向这个epoll对象中添加或者删除socket连接。同时，epoll_wait的效率也是非常高的，因为调用epoll_wait时，并没有一股脑的向操作系统复制这100W个连接的句柄数据，内核也不需要去遍历全部的连接。

一颗红黑树，一张准备就绪的句柄链表，少量的内核cache，就帮我们解决了高并发下的socket处理问题。

​    执行epoll_create时，创建了红黑树和就绪链表，执行epoll_ctl时，如果增加socket句柄，则检查红黑树中是否存在，存在就立即返回，不存在则添加进红黑树，然后向内核注册回调函数，用于当中断事件来临时向准备就绪链表中插入数据。执行epoll_wait时，立刻返回准备就绪链表里的数据即可。

### Reactor反应队设计模式

1. Reactor单线程

   ![reactor-1](\img\reactor-1.png)

   每个客户端发起连接请求都会交给acceptor,acceptor根据事件类型交给线程handler处理，注意acceptor 处理和 handler 处理都在一个线程中处理，所以其中某个 handler 阻塞时, 会导致其他所有的 client 的 handler 都得不到执行, 并且更严重的是, handler 的阻塞也会导致整个服务不能接收新的 client 请求(因为 acceptor 也被阻塞了). 因为有这么多的缺陷, 因此单线程Reactor 模型用的比较少.

2. Reactor多线程

   ![reactor-2](\img\reactor-2.png)

   有专门一个线程, 即 Acceptor 线程用于监听客户端的TCP连接请求.

   客户端连接的 IO 操作都是由一个特定的 NIO 线程池负责. 每个客户端连接都与一个特定的 NIO 线程绑定, 因此在这个客户端连接中的所有 IO 操作都是在同一个线程中完成的.

   客户端连接有很多, 但是 NIO 线程数是比较少的, 因此一个 NIO 线程可以同时绑定到多个客户端连接中.

   缺点：如果我们的服务器需要同时处理大量的客户端连接请求或我们需要在客户端连接时, 进行一些权限的检查, 那么单线程的 Acceptor 很有可能就处理不过来, 造成了大量的客户端不能连接到服务器.

3. Reactor主从模式

   ![reactor-3](\img\reactor-3.png)

   Reactor 的主从多线程模型和 Reactor 多线程模型很类似, 只不过 Reactor 的主从多线程模型的 acceptor 使用了线程池来处理大量的客户端请求.

### 数据持久化

​	将内存中的数据保存到硬盘文件保证数据持久化。启动时从硬盘文件加载到内存恢复数据。
​	方式：RDB、AOF；
​      RDB文件是按一定周期将内存数据快照保存到硬盘的二进制文件。对应的文件名dump.rdb
​      实现方式：fork（）一个子线程，主线程复制数据到子线程的内存中，然后由子线程写入到临时文件中，用这个临时文件替换上次的快照文件，然后子线程退出，内存释放。（shutdown save slave命令都会触发）
​      AOF是把写命令写到一个日志文件里。每一个写命令都通过 write 函数追加到 appendonly.aof 中。

### 缓存和数据库一致性问题

采取合适的策略来降低缓存和数据库间数据不一致的概率，而无法保证两者间的强一致性。合适的策略包括合适的缓存更新策略，更新数据库后及时更新缓存、缓存失败时增加重试机制。给缓存设置过期时间，是保证最终一致性的解决方案

更新数据时，怎么最好保证双写数据一致性？

1. 先更新数据库，再更新缓存

   两个请求同时请求一条数据更新操作，会导致脏数据

2. 先删除缓存，再更新数据库

   也会产生脏数据

3. 先更新数据库，再删除缓存

   - 更新：应用程序先从cache取数据，没有得到，则从数据库中取数据，成功后，放到缓存中。
   - 命中：应用程序从cache中取数据，取到后返回。
   - 失效：先把数据存到数据库中，成功后，再让缓存失效。

   另外，知名社交网站facebook也在论文《Scaling Memcache at Facebook》中提出，**他们用的也是先更新数据库，再删缓存的策略。**



### 缓存雪崩、缓存穿透、缓存预热、缓存更新、缓存降级等问题

- 缓存雪崩：
  - 原因：缓存设置的过期时间都是一样的、缓存挂掉。
  - 同一时间大面积key失效，导致本来访问缓存的量一下子要请求到数据库。导致数据库cpu和内存占有率变大，然后带来了一系列的连锁反应，导致系统崩溃。（可以给key的过期时间加个随机有效值，指定一个分钟，然后后面的尾数随机等）
  - 缓存挂掉：事发前：redis集群，尽量避免挂掉；事发中：限流降级；事发后：redis持久化，恢复数据
  
- 缓存穿透：
  - 原因：查询一个一定不存在的数据，缓存中没有数据，数据库也没有对应的数据，导致既查询了缓存又查询了数据库。
  - 高级用法布隆过滤器（Bloom Filter）过滤器限制请求到数据库
  - 可以把空对象放到缓存里，不过设置一个过短的有效期
  
- 缓存预热：有些数据需要经常查询，可以先手动或自动把数据库数据同步到缓存中	

  - mysql倒入百万数据到redis

    按照上述redis协议，我们使下sql来构造协议数据。然后编写脚本使用pipe模式导入redis。

- 缓存更新：缓存失效

### redis的过期策略以及内存淘汰机制

定期删除+惰性删除策略
	定期删除，每隔100ms随机抽查key，过期了就删除。肯定不会抽查所有的key。
	惰性删除，当获取key时，会判断他的过期时间，如果过期了就删除。
	总有过期的key没有被删除，这时候就要配置内存淘汰机制。（redis.conf中有配置）
	内存淘汰策略：
	redis.conf 配置文件里配置最大内存maxmemory；当现有内存大于maxmemory，触发内存淘汰策略。
	volatile-lru：最近最少使用的数据淘汰
	volatile-random：从已设置过期时间的数据集(server.db[i].expires)中任意选择数据淘汰
	volatile-ttl：挑选将要过期的数据淘汰，ttl值越大越优先被淘汰
	allkeys-lru：从数据集(server.db[i].dict)中挑选最近最少使用的数据淘汰
	allkeys-random：从数据集(server.db[i].dict)中选择任意数据淘汰
	noeviction：不移除任何key，内存满时直接返回一个写错误 ，默认选项，一般不会选用

### 注意点

1. key、value的储存对象的大小最大为512M
2. redis支持的数据结构：String|list|set|sorted set|hash
3. key的命名方式：object-type:idvalue:field，之间用分号隔开

### 主要命令

1. 查询所有键：keys *      (慎重使用，数据量大直接搞挂服务器)

2. 查询当前键的数量：dbsize 
   键值操作相关命令：
   del key；（删除键）
   exists key；（查看键存在）

3. 字符串操作指令：
   set com:user aa;(设置指定值)
   get com:user;(获取指定key的值)
   getrange com:product 0 2;(获取子字符串)
   getset com:user cc;(设置新值，返回旧值)
   mget com:user com:product;(获取所有(一个或多个)给定 key 的值)  
   
   setex com:animal 9 yy;(只有在key存在时设置值，并带上有效期：秒)
setnx com:animal uu;(只有在key不存在时设置key的值)  (返回值0或1)
   incr key；（key对应的值加1）
   decr key；（key对应的值减1）
   
   用到场景：缓存数据、计数器、分布式锁
   
4. list列表操作指令：
    lpush user aa; 
      lpush user bb; (插入到头部)
      llen user; (列表长度)
      lrange user 0 10; (查看范围的数据)
      lpop user; (取出第一个元素)
      blpop user 10;(移出并获取列表的第一个元素， 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止)
      	brpop cn:user 2;(移出并获取列表的最后一个元素， 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止)
      	lindex cn:user 0;(通过索引获取列表中的元素)
      	linsert cn:user before bb zz;
      	linsert cn:user after bb zz;(在列表的元素前或者后插入元素)
      	lpushx cn:user kk;(将一个值插入到已存在的列表头部)
      	lrem cn:user -2 zz;(移除列表元素)
      	lset cn:user 3 aa;(通过索引设置列表元素的值)
      	rpop cn:user;(移除并获取列表最后一个元素)
      	rpoplpush cn:user cn:tempuser;(移除列表的最后一个元素，并将该元素添加到另一个列表并返回)
      	rpush cn:user bb;(将一个或多个值插入到列表的尾部(最右边))
      	rpushx cn:user bb;()

5. set集合操作指令：

    ​		sadd key value；（向集合中添加一个成员）
    ​		spop key；（移除并返回一个随机元素）

6. sorted set集合操作指令：

    ​	zadd key score value；（向有序集合添加一个成员）
    ​	zrangebyscore key score min max; (通过分数取出有序集合中的成员)
    ​	zrem key value；（移除有序队列中成员）
    ​	用到的场景：延时队列

7. hash操作指令：
    hmset com:type usera aa userb bb;(同时将多个field-value(域-值)对设置到哈希表key中)
      	hgetall com:type; (获取key中所有field-value)
      	hget com:type usera; (获取key中field对应的value)
      	hset com:type usera cc;(将哈希表 key 中的字段 field 的值设为 value )
      	hkeys com:type;(获取hash中所有字段field)
      	hvals com:type;(获取field下的所有值)
      	hsetnx com:type userc nn;(只有在字段field不存在时，设置哈希表字段的值)
      	hdel com:type usera;(删除一个或多个哈希表字段)
      	hexists com:type usera;(查看哈希表 key 中，指定的字段是否存在)

### 集群模式

- 主从模式

  主数据库和从数据库（master、slave），

  主数据库提供读写操作，读写操作变化的数据自动同步到从数据库。从数据库提供读操作，并且接收主数据库同步过来的数据一个master可以拥有多个slave，一个slave只有从属于一个master slave挂了不影响其他slave与master之间的同步，重新启动后会自动从master同步数据master挂了，不会影响slave的读操作，但是redis提供不了写操作，只有master重启后才能写操作master挂了后，slave也不会选举其中一个master。

  工作机制：slave启动后会向master发送sync命令（同步数据）。master接收到命令后会保存快照和缓存指令，然后将快照文件和指令发送给slave，slave接收到快照文件和缓存指令就加载快照文件也执行指令。这样保证数据一致性。

- 哨兵模式(sentinel模式)

  监控redis集群的运行状况，sentinel模式建立在主从模式基础上，如果master节点挂了，sentinel会在slave节点中选举一个来做master，其他的slave修改指向新的master。
  		原先的master重启后，充当slave节点，接收新的master同步数据。
  		sentinel也是一个集群，他们之间是会自动监控
  		一个sentinel或sentinel集群可以管理多个redis，也可以监控同一个redis

  工作机制：每个sentinel以每秒向master，slave和sentinel实例发送一次ping命令；如果一个实例距离最后一次有效回复ping命令时间超过设定的有效值，则这个实例被标志为主观下线。当一个master被标记为主观下线，则正在监视这个master的所有sentinel实例每秒一次的频率确认master的确进入主观下线状态。当有足够多的sentinel确认master的确进入了主观下线状态，则标记此master为客观下线状态。

- cluster分布式模式（解决单机redis容量的问题，解决单机性能问题，将存储数据进行分片存储到多个redis实例中）

多个redis节点网络互联（ping-pong机制），数据共享

redis-cluster把所有的物理节点映射到[0-16383]slot 哈希槽上（不一定是平均分配）,cluster 负责维护node<->slot<->value； 集群中放置一个 key-value 时，根据 CRC16(key) mod 16383的值，决定将一个key放到哪个桶中。

​		所有的节点都是一主一从或一主多从，其中的slave不提供服务，备用
​		不支持同时处理多个key（mset、mget），因为redis要把key均匀分布在各个节点上，并发量高的话同时创建key-value导致不可预测的情况
​		支持在线增加、删除节点
​		客户端可以连接任何一个主节点进行读写
​		<img src=".\img\redis-cluster.png" alt="redis-cluster" style="zoom:50%;" />

		具体搭建可参照博客：https://blog.csdn.net/miss1181248983/article/details/90056960

### redis支持lua脚本

（项目中通过这个实现限流）配置每秒的速率和每秒的容量值

项目中的场景有：

1. 缓存数据
2. 分布式锁
3. 延时队列
4. 限流
5. 消息队列

### redis安装步骤及详细属性配置

1. 进入官网找到下载地址 [https://redis.io/download]，鼠标右击选择 **复制链接地址**

   **进入到Xshell控制台，进入usr/，输入wget，命令如下:**

   ~~~shell
   cd usr/
   wget https://download.redis.io/releases/redis-6.0.10.tar.gz
   ~~~

2. 解压

   - **解压后在根目录上输入ls 列出所有目录会发现与下载redis之前多了一个redis-6.0.10.tar.gz文件和 redis-6.0.10的目录。**
   - **一般都会将redis目录放置到 /usr/local/redis目录，所以这里输入下面命令将目前在/root目录下的redis-6.0.10文件夹更改目录，同时更改文件夹名称为redis。**

   ~~~shell
   tar -zvxf redis-6.0.10.tar.gz
   mv /usr/redis-6.0.10 /usr/local/redis
   cd local/
   ~~~

3. 编译

   **cd到redis目录，输入命令make执行编译命令，接下来控制台会输出各种编译过程中输出的内容。**（注意，编译需要C语言编译器gcc的支持，如果没有，需要先安装gcc。可以使用rpm -q gcc查看gcc是否安装，如果编译出错，请使用make clean清除临时文件。之后，找到出错的原因，解决问题后再来重新安装。 ）

   ~~~shell
   cd redis/
   make
   ~~~

   如果输入make命令出现上图所示问题时，可能是gcc需要升级或安装

   ~~~
   1、安装gcc套装：
   yum install cpp
   yum install binutils
   yum install glibc
   yum install glibc-kernheaders
   yum install glibc-common
   yum install glibc-devel
   yum install gcc
   yum install make
   2、升级gcc
   yum -y install centos-release-scl
   yum -y install devtoolset-9-gcc devtoolset-9-gcc-c++ devtoolset-9-binutils
   scl enable devtoolset-9 bash
   3、设置永久升级：
   echo "source /opt/rh/devtoolset-9/enable" >>/etc/profile
   4、重新make：
   ~~~

4. 安装

   ~~~shell
   make PREFIX=/usr/local/redis install
   ~~~

   这里多了一个关键字 PREFIX= 这个关键字的作用是编译的时候用于指定程序存放的路径。比如我们现在就是指定了redis必须存放在/usr/local/redis目录。假设不添加该关键字Linux会将可执行文件存放在/usr/local/bin目录，库文件会存放在/usr/local/lib目录。配置文件会存放在/usr/local/etc目录。其他的资源文件会存放在usr/local/share目录。这里指定号目录也方便后续的卸载

5. 启动

   **根据上面的操作已经将redis安装完成了。在目录/usr/local/redis 输入下面命令启动redis**

   ~~~shell
   ./bin/redis-server ./redis.conf
   ~~~

6. redis.config配置文件

   在目录/usr/local/redis下有一个redis.conf的配置文件。我们上面启动方式就是执行了该配置文件的配置运行的。我么可以通过cat、vim、less等Linux内置的读取命令读取该文件。

   也可以通过redis-cli命令进入redis控制台后通过CONFIG GET * 的方式读取所有配置项。 如下：

   ~~~shell
   redis-cli
   ~~~

   如出现 bash: redis-cli: 未找到命令
   **解决方法：**

   ~~~shell
   make install
   
   CONFIG GET *
   ~~~

   **修改配置文件：**这里我要将daemonize改为yes，同时也将#bind 127.0.0.1注释，将protected-mode设置为no。
   这样启动后我就可以在外网访问了

   ~~~
   vim /usr/local/redis/redis.conf
   ~~~

   使用命令 /requirepass 快速查找到 # requirepass foobared 然后去掉注释，这个foobared改为自己的密码。也可以不加密码

   **开机启动配置**

   ~~~
   echo "/usr/local/bin/redis-server /etc/redis/redis.conf &" >> /etc/rc.local
   ~~~

   **查看Redis是否正在运行，命令如下：**

   ~~~
   ps -aux | grep redis
   ~~~

   

   详细属性配置

   | 配置项                                                       | 说明                                                         |
   | ------------------------------------------------------------ | ------------------------------------------------------------ |
   | daemonize  no                                                | 默认情况下，redis 不是在后台运行的，如果需要在后台运行，把该项的值更改为yes。 |
   | pidfile  /var/run/redis.pid                                  | 当Redis 在后台运行的时候，Redis 默认会把pid 文件放在/var/run/redis.pid，你可以配置到其他地址。当运行多个redis 服务时，需要指定不同的pid 文件和端口 |
   | port                                                         | 监听端口，默认为6379                                         |
   | bind 127.0.0.1                                               | 指定Redis 只接收来自于该IP 地址的请求，如果不进行设置，那么将处理所有请求，在生产环境中为了安全最好设置该项。默认注释掉，不开启 |
   | timeout 0                                                    | 设置客户端连接时的超时时间，单位为秒。当客户端在这段时间内没有发出任何指令，那么关闭该连接 |
   | tcp-keepalive 0                                              | 指定TCP连接是否为长连接,"侦探"信号有server端维护。默认为0.表示禁用 |
   | loglevel notice                                              | log 等级分为4 级，debug,verbose, notice, 和warning。生产环境下一般开启notice |
   | logfile stdout                                               | 配置log 文件地址，默认使用标准输出，即打印在命令行终端的窗口上，修改为日志文件目录 |
   | databases 16                                                 | 设置数据库的个数，可以使用SELECT 命令来切换数据库。默认使用的数据库是0号库。默认16个库 |
   | save 900 1<br/>save 300 10<br>save 60 10000                  | 保存数据快照的频率，即将数据持久化到dump.rdb文件中的频度。用来描述"在多少秒期间至少多少个变更操作"触发snapshot数据保存动作。默认设置，意思是：<br/>if(在60 秒之内有10000 个keys 发生变化时){<br/>进行镜像备份<br/>}else if(在300 秒之内有10 个keys 发生了变化){<br/>进行镜像备份<br/>}else if(在900 秒之内有1 个keys 发生了变化){<br/>进行镜像备份<br/>} |
   | stop-writes-on-bgsave-error yes                              | 当持久化出现错误时，是否依然继续进行工作，是否终止所有的客户端write请求。默认设置"yes"表示终止，一旦snapshot数据保存故障，那么此server为只读服务。如果为"no"，那么此次snapshot将失败，但下一次snapshot不会受到影响，不过如果出现故障,数据只能恢复到"最近一个成功点" |
   | rdbcompression yes                                           | 在进行数据镜像备份时，是否启用rdb文件压缩手段，默认为yes。压缩可能需要额外的cpu开支，不过这能够有效的减小rdb文件的大小，有利于存储/备份/传输/数据恢复 |
   | rdbchecksum yes                                              | 是否进行校验和，是否对rdb文件使用CRC64校验和,默认为"yes"，那么每个rdb文件内容的末尾都会追加CRC校验和，利于第三方校验工具检测文件完整性 |
   | dbfilename dump.rdb                                          | 镜像备份文件的文件名，默认为 dump.rdb                        |
   | dir ./                                                       | 数据库镜像备份的文件rdb/AOF文件放置的路径。这里的路径跟文件名要分开配置是因为Redis 在进行备份时，先会将当前数据库的状态写入到一个临时文件中，等备份完成时，再把该临时文件替换为上面所指定的文件，而这里的临时文件和上面所配置的备份文件都会放在这个指定的路径当中 |
   | slaveof <masterip> <masterport>                              | 设置该数据库为其他数据库的从数据库，并为其指定master信息。   |
   | masterauth                                                   | 当主数据库连接需要密码验证时，在这里指定                     |
   | slave-serve-stale-data yes                                   | 当主master服务器挂机或主从复制在进行时，是否依然可以允许客户访问可能过期的数据。在"yes"情况下,slave继续向客户端提供只读服务,有可能此时的数据已经过期；在"no"情况下，任何向此server发送的数据请求服务(包括客户端和此server的slave)都将被告知"error" |
   | slave-read-only yes                                          | slave是否为"只读"，强烈建议为"yes"                           |
   | repl-ping-slave-period 10                                    | slave向指定的master发送ping消息的时间间隔(秒)，默认为10      |
   | repl-timeout 60                                              | slave与master通讯中,最大空闲时间,默认60秒.超时将导致连接关闭 |
   | repl-disable-tcp-nodelay no                                  | slave与master的连接,是否禁用TCP nodelay选项。"yes"表示禁用,那么socket通讯中数据将会以packet方式发送(packet大小受到socket buffer限制)。<br/>可以提高socket通讯的效率(tcp交互次数),但是小数据将会被buffer,不会被立即发送,对于接受者可能存在延迟。"no"表示开启tcp nodelay选项,任何数据都会被立即发送,及时性较好,但是效率较低，建议设为no |
   | slave-priority 100                                           | 适用Sentinel模块(unstable,M-S集群管理和监控),需要额外的配置文件支持。slave的权重值,默认100.当master失效后,Sentinel将会从slave列表中找到权重值最低(>0)的slave,并提升为master。如果权重值为0,表示此slave为"观察者",不参与master选举 |
   | requirepass foobared                                         | 设置客户端连接后进行任何其他指定前需要使用的密码。警告：因为redis 速度相当快，所以在一台比较好的服务器下，一个外部的用户可以在一秒钟进行150K 次的密码尝试，这意味着你需要指定非常非常强大的密码来防止暴力破解。 |
   | maxclients 10000                                             | 限制同时连接的客户数量。当连接数超过这个值时，redis 将不再接收其他连接请求，客户端尝试连接时将收到error 信息。默认为10000，要考虑系统文件描述符限制，不宜过大，浪费文件描述符，具体多少根据具体情况而定 |
   | maxmemory <bytes>                                            | redis-cache所能使用的最大内存(bytes),默认为0,表示"无限制",最终由OS物理内存大小决定(如果物理内存不足,有可能会使用swap)。此值尽量不要超过机器的物理内存尺寸,从性能和实施的角度考虑,可以为物理内存3/4。此配置需要和"maxmemory-policy"配合使用,当redis中内存数据达到maxmemory时,触发"清除策略"。在"内存不足"时,任何write操作(比如set,lpush等)都会触发"清除策略"的执行。在实际环境中,建议redis的所有物理机器的硬件配置保持一致(内存一致),同时确保master/slave中"maxmemory""policy"配置一致。<br/>当内存满了的时候，如果还接收到set 命令，redis 将先尝试剔除设置过expire 信息的key，而不管该key 的过期时间还没有到达。在删除时，将按照过期时间进行删除，最早将要被过期的key 将最先被删除。如果带有expire 信息的key 都删光了，内存还不够用，那么将返回错误。这样，redis 将不再接收写请求，只接收get 请求。maxmemory 的设置比较适合于把redis 当作于类似memcached的缓存来使用。 |
   | maxmemory-policy volatile-lru                                | 内存不足"时,数据清除策略,默认为"volatile-lru"。<br/>volatile-lru  ->对"过期集合"中的数据采取LRU(近期最少使用)算法.如果对key使用"expire"指令指定了过期时间,那么此key将会被添加到"过期集合"中。将已经过期/LRU的数据优先移除.如果"过期集合"中全部移除仍不能满足内存需求,将OOM.<br/>allkeys-lru ->对所有的数据,采用LRU算法<br/>volatile-random ->对"过期集合"中的数据采取"随即选取"算法,并移除选中的K-V,直到"内存足够"为止. 如果如果"过期集合"中全部移除全部移除仍不能满足,将OOM<br/>allkeys-random ->对所有的数据,采取"随机选取"算法,并移除选中的K-V,直到"内存足够"为止<br/>volatile-ttl ->对"过期集合"中的数据采取TTL算法(最小存活时间),移除即将过期的数据.<br/>noeviction ->不做任何干扰操作,直接返回OOM异常<br/>另外，如果数据的过期不会对"应用系统"带来异常,且系统中write操作比较密集,建议采取"allkeys-lru" |
   | maxmemory-samples 3                                          | 默认值3，上面LRU和最小TTL策略并非严谨的策略，而是大约估算的方式，因此可以选择取样值以便检查 |
   | appendonly no                                                | 默认情况下，redis 会在后台异步的把数据库镜像备份到磁盘，但是该备份是非常耗时的，而且备份也不能很频繁。所以redis 提供了另外一种更加高效的数据库备份及灾难恢复方式。开启append only 模式之后，redis 会把所接收到的每一次写操作请求都追加到appendonly.aof 文件中，当redis 重新启动时，会从该文件恢复出之前的状态。但是这样会造成appendonly.aof 文件过大，所以redis 还支持了BGREWRITEAOF 指令，对appendonly.aof 进行重新整理。如果不经常进行数据迁移操作，推荐生产环境下的做法为关闭镜像，开启appendonly.aof，同时可以选择在访问较少的时间每天对appendonly.aof 进行重写一次。<br/>另外，对master机器,主要负责写，建议使用AOF,对于slave,主要负责读，挑选出1-2台开启AOF，其余的建议关闭 |
   | appendfilename appendonly.aof                                | aof文件名字，默认为appendonly.aof                            |
   | appendfsync always/everysec/no                               | 设置对appendonly.aof 文件进行同步的频率。always 表示每次有写操作都进行同步，everysec 表示对写操作进行累积，每秒同步一次。no不主动fsync，由OS自己来完成。这个需要根据实际业务场景进行配置 |
   | no-appendfsync-on-rewrite no                                 | 在aof rewrite期间,是否对aof新记录的append暂缓使用文件同步策略,主要考虑磁盘IO开支和请求阻塞时间。默认为no,表示"不暂缓",新的aof记录仍然会被立即同步 |
   | auto-aof-rewrite-percentage 100                              | 当Aof log增长超过指定比例时，重写log file， 设置为0表示不自动重写Aof 日志，重写是为了使aof体积保持最小，而确保保存最完整的数据。 |
   | auto-aof-rewrite-min-size 64mb                               | 触发aof rewrite的最小文件尺寸                                |
   | lua-time-limit 5000                                          | lua脚本运行的最大时间                                        |
   | slowlog-log-slower-than 10000                                | "慢操作日志"记录,单位:微秒(百万分之一秒,1000 * 1000),如果操作时间超过此值,将会把command信息"记录"起来.(内存,非文件)。其中"操作时间"不包括网络IO开支,只包括请求达到server后进行"内存实施"的时间."0"表示记录全部操作 |
   | slowlog-max-len 128                                          | "慢操作日志"保留的最大条数,"记录"将会被队列化,如果超过了此长度,旧记录将会被移除。可以通过"SLOWLOG <subcommand> args"查看慢记录的信息(SLOWLOG get 10,SLOWLOG reset) |
   | hash-max-ziplist-entries 512                                 | hash类型的数据结构在编码上可以使用ziplist和hashtable。ziplist的特点就是文件存储(以及内存存储)所需的空间较小,在内容较小时,性能和hashtable几乎一样.因此redis对hash类型默认采取ziplist。如果hash中条目的条目个数或者value长度达到阀值,将会被重构为hashtable。<br/>这个参数指的是ziplist中允许存储的最大条目个数，，默认为512，建议为128 |
   | hash-max-ziplist-value 64                                    | ziplist中允许条目value值最大字节数，默认为64，建议为1024     |
   | list-max-ziplist-entries 512<br/>list-max-ziplist-value 64   | 对于list类型,将会采取ziplist,linkedlist两种编码类型。解释同上。 |
   | set-max-intset-entries 512                                   | intset中允许保存的最大条目个数,如果达到阀值,intset将会被重构为hashtable |
   | zset-max-ziplist-entries 128<br/>zset-max-ziplist-value 64   | zset为有序集合,有2中编码类型:ziplist,skiplist。因为"排序"将会消耗额外的性能,当zset中数据较多时,将会被重构为skiplist。 |
   | activerehashing yes                                          | 是否开启顶层数据结构的rehash功能,如果内存允许,请开启。rehash能够很大程度上提高K-V存取的效率 |
   | client-output-buffer-limit normal 0 0 0<br/>client-output-buffer-limit slave 256mb 64mb 60<br/>client-output-buffer-limit pubsub 32mb 8mb 60 | 客户端buffer控制。在客户端与server进行的交互中,每个连接都会与一个buffer关联,此buffer用来队列化等待被client接受的响应信息。如果client不能及时的消费响应信息,那么buffer将会被不断积压而给server带来内存压力.如果buffer中积压的数据达到阀值,将会导致连接被关闭,buffer被移除。<br/>buffer控制类型包括:normal -> 普通连接；slave ->与slave之间的连接；pubsub ->pub/sub类型连接，此类型的连接，往往会产生此种问题;因为pub端会密集的发布消息,但是sub端可能消费不足.<br/>指令格式:client-output-buffer-limit <class> <hard> <soft> <seconds>",其中hard表示buffer最大值,一旦达到阀值将立即关闭连接;<br/>soft表示"容忍值",它和seconds配合,如果buffer值超过soft且持续时间达到了seconds,也将立即关闭连接,如果超过了soft但是在seconds之后，buffer数据小于了soft,连接将会被保留.<br/>其中hard和soft都设置为0,则表示禁用buffer控制.通常hard值大于soft. |
   | hz 10                                                        | Redis server执行后台任务的频率,默认为10,此值越大表示redis对"间歇性task"的执行次数越频繁(次数/秒)。"间歇性task"包括"过期集合"检测、关闭"空闲超时"的连接等,此值必须大于0且小于500。此值过小就意味着更多的cpu周期消耗,后台task被轮询的次数更频繁。此值过大意味着"内存敏感"性较差。建议采用默认值。 |
   | include /path/to/local.conf<br/> include /path/to/other.conf | 额外载入配置文件。                                           |

   