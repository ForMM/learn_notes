# Redis

### redis为什么效率这么高,可以支持10万QPS

1. 纯内存访问：数据存放在内存中，内存的响应时间大约是100纳秒，这是Redis每秒万亿级别访问的重要基础。
2. 非阻塞I/O：Redis采用epoll做为I/O多路复用技术的实现，再加上Redis自身的事件处理模型将epoll中的连接，读写，关闭都转换为了时间，不在I/O上浪费过多的时间。
3. 单线程避免了线程切换和竞态产生的消耗。
4. Redis采用单线程模型，每条命令执行如果占用大量时间，会造成其他线程阻塞，对于Redis这种高性能服务是致命的，所以Redis是面向高速执行的数据库

### redis线程模型

I/O多路复用技术：

reactor反应队设计模式

### 数据持久化

​	将内存中的数据保存到硬盘文件保证数据持久化。启动时从硬盘文件加载到内存恢复数据。
​	方式：RDB、AOF；
​      RDB文件是按一定周期将内存数据快照保存到硬盘的二进制文件。对应的文件名dump.rdb
​      实现方式：fork（）一个子线程，主线程复制数据到子线程的内存中，然后由子线程写入到临时文件中，用这个临时文件替换上次的快照文件，然后子线程退出，内存释放。（shutdown save slave命令都会触发）
​      AOF是把写命令写到一个日志文件里。每一个写命令都通过 write 函数追加到 appendonly.aof 中。

### 缓存和数据库一致性问题

采取合适的策略来降低缓存和数据库间数据不一致的概率，而无法保证两者间的强一致性。合适的策略包括合适的缓存更新策略，更新数据库后及时更新缓存、缓存失败时增加重试机制。

### 缓存雪崩、缓存穿透、缓存预热、缓存更新、缓存降级等问题

缓存雪崩：同一时间大面积key失效，导致本来访问缓存的量一下子要请求到数据库。导致数据库cpu和内存占有率变大，然后带来了一系列的连锁反应，导致系统崩溃。（可以给key的过期时间加个随机有效值）
	缓存穿透：缓存中没有数据，数据库也可以对应的数据，导致既查询了缓存又查询了数据库。（1.（高级用法布隆过滤器（Bloom Filter））过滤器限制请求到数据库2.也可以把空对象放到缓存里，不过设置一个过短的有效期）
	缓存预热：有些数据需要经常查询，可以先手动或自动把数据库数据同步到缓存中
	缓存更新：缓存失效

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

- cluster分布式模式（解决单机redis容量的问题，将存储数据进行分片存储到多个redis实例中）

多个redis节点网络互联，数据共享
		所有的节点都是一主一从或一主多从，其中的slave不提供服务，备用
		不支持同时处理多个key（mset、mget），因为redis要把key均匀分布在各个节点上，并发量高的话同时创建key-value导致不可预测的情况
		支持在线增加、删除节点
		客户端可以连接任何一个主节点进行读写
		

		具体搭建可参照博客：https://blog.csdn.net/miss1181248983/article/details/90056960

### redis支持lua脚本

（项目中通过这个实现限流）配置每秒的速率和每秒的容量值

项目中的场景有：
1.缓存数据
2.分布式锁
3.延时队列
4.限流