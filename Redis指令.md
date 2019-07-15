# Redis指令

### 要点

1. key、value的储存对象的大小最大为512M
2. redis支持的数据结构：String|list|set|sorted set|hash
3. key的命名方式：object-type:idvalue:field，之间用分号隔开

### 命令：

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
   setnx com:animal uu;(只有在key不存在时设置key的值)

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

   

6. sorted set集合操作指令：

   

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