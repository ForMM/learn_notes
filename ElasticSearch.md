### ElasticSearch学习

#### 倒排索引

- 正向索引

  文档1的id----》单词1：出现次数，出现的位置列表；单词2：出现次数，出现的位置列表...

  文档2的id----》关键词的列表

  通过key，去找value

  **正向索引（forward index），那么就需要扫描索引库中的所有文档，找出所有包含关键词的文档，再根据打分模型进行打分，排出名次后呈现给用户。**这样的索引结构根本无法满足实时返回排名结果的要求。

- 反向索引

  **搜索引擎会将正向索引重新构建为倒排索引**，即把文件ID对应到关键词的映射转换为**关键词到文件ID的映射**，每个关键词都对应着一系列的文件，这些文件中都出现这个关键词。

  “关键词1”：“文档1”的ID，“文档2”的ID，…………。

  “关键词2”：带有此关键词的文档ID列表。

  从词的关键字，去找文档。

- 单词-文档矩阵（数据结构），“倒排索引”是实现单词到文档映射关系的最佳实现方式。

#### 查询语言

- AND

  文档匹配当前从句当且仅当AND操作符左右两边的词项都在文档中出现。

- OR

  包含当前从句中任意词项的文档都会被视为与该从句匹配。

- NOT

  与当前从句匹配的文档必须不包含NOT操作符后面的词项。

- +

  只有包含+操作符后面的词项的文档才会被认为是从句匹配。

- -

  与从句匹配的文档不能出现-操作符后的词项。


#### linux安装教程

​	下载地址：https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.11.1-linux-x86_64.tar.gz

~~~shell
#下载到当前目录
curl -L -O https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.11.1-linux-x86_64.tar.gz
#解压到当前目录
tar -zxvf elasticsearch-7.11.1-linux-x86_64.tar.gz
#运行es
./elasticsearch-7.11.1/bin/elasticsearch

~~~

遇见的问题：

1. 由于内存分配不够造成的，修改适合本机的内存，修改文件config/jvm.options， 修改内存使用量为  -Xms512m  -Xmx512m

~~~shell
error:
Java HotSpot(TM) 64-Bit Server VM warning: INFO: os::commit_memory(0x00000000c5330000, 986513408, 0) failed; error='Cannot allocate memory' (errno=12)
	at org.elasticsearch.tools.launchers.JvmOption.flagsFinal(JvmOption.java:119)
	at org.elasticsearch.tools.launchers.JvmOption.findFinalOptions(JvmOption.java:81)
	at org.elasticsearch.tools.launchers.JvmErgonomics.choose(JvmErgonomics.java:38)
	at org.elasticsearch.tools.launchers.JvmOptionsParser.jvmOptions(JvmOptionsParser.java:135)
	at org.elasticsearch.tools.launchers.JvmOptionsParser.main(JvmOptionsParser.java:86)
~~~

2. 不允许使用root用户启动，那么我们新建一个es用户，并赋予权限：添加es用户

~~~shell
[2021-02-24T17:46:26,352][ERROR][o.e.b.ElasticsearchUncaughtExceptionHandler] [iZwz95ydf6212wz3xgltwfZ] uncaught exception in thread [main]
org.elasticsearch.bootstrap.StartupException: java.lang.RuntimeException: can not run elasticsearch as root
	at org.elasticsearch.bootstrap.Elasticsearch.init(Elasticsearch.java:163) ~[elasticsearch-7.11.1.jar:7.11.1]
	at org.elasticsearch.bootstrap.Elasticsearch.execute(Elasticsearch.java:150) ~[elasticsearch-7.11.1.jar:7.11.1]
	at org.elasticsearch.cli.EnvironmentAwareCommand.execute(EnvironmentAwareCommand.java:75) ~[elasticsearch-7.11.1.jar:7.11.1]
	at org.elasticsearch.cli.Command.mainWithoutErrorHandling(Command.java:116) ~[elasticsearch-cli-7.11.1.jar:7.11.1]
	at org.elasticsearch.cli.Command.main(Command.java:79) ~[elasticsearch-cli-7.11.1.jar:7.11.1]
	at org.elasticsearch.bootstrap.Elasticsearch.main(Elasticsearch.java:115) ~[elasticsearch-7.11.1.jar:7.11.1]
	at org.elasticsearch.bootstrap.Elasticsearch.main(Elasticsearch.java:81) ~[elasticsearch-7.11.1.jar:7.11.1]
Caused by: java.lang.RuntimeException: can not run elasticsearch as root
	at org.elasticsearch.bootstrap.Bootstrap.initializeNatives(Bootstrap.java:100) ~[elasticsearch-7.11.1.jar:7.11.1]
	at org.elasticsearch.bootstrap.Bootstrap.setup(Bootstrap.java:167) ~[elasticsearch-7.11.1.jar:7.11.1]
	at org.elasticsearch.bootstrap.Bootstrap.init(Bootstrap.java:387) ~[elasticsearch-7.11.1.jar:7.11.1]
	at org.elasticsearch.bootstrap.Elasticsearch.init(Elasticsearch.java:159) ~[elasticsearch-7.11.1.jar:7.11.1]
	... 6 more

~~~

~~~shell
#添加用户
useradd es
#添加es用户密码
passwd es
#将文件夹elasticsearch-7.11.1赋予es权限
chown -R es:es /app/elastic/elasticsearch-7.11.1
#切换为es用户
su es
#启动es
../bin/elasticsearch &
~~~

3. elasticsearch最大线程数目太低

   ~~~
   修改 /etc/security/limits.conf
   
   root用户来操作在文件末尾增加以下两行（注意：es是我自己的用户名，如果不填，可以用*代替）
   
   es  soft nproc  4096
   es  hard nproc  4096
   ~~~

   

4. 浏览器访问http://118.24.242.170:9200/拒绝访问（118.24.242.170为服务器ip）

   ~~~
   使用root用户，打开elasticsearch.yml文件，如下：
   vi /usr/local/tool/elasticsearch/elasticsearch-5.4.2/config/elasticsearch.yml
   文件内增加如下代码
   network.host: 0.0.0.0
   ~~~

5. elasticsearch用户拥有的内存权限太小，至少需要262144

   ~~~
   在   /etc/sysctl.conf文件最后添加一行
   vm.max_map_count=262144
   
   或者
   
   sysctl -w vm.max_map_count=262144
   ~~~

   

6. 的node-1是上面一个默认的记得打开就可以了

   ~~~
   the default discovery settings are unsuitable for production use; at least one of [discovery.seed_hosts, discovery.seed_providers, cluster.initial_master_nodes] must be configured
   
   修改
   elasticsearch.yml
   取消注释保留一个节点
   cluster.initial_master_nodes: ["node-1"]
   ~~~

7. 详细配置

   ~~~
   cluster.name: elasticsearch
   # 配置的集群名称，默认是elasticsearch，es服务会通过广播方式自动连接在同一网段下的es服务，通过多播方式进行通信，同一网段下可以有多个集群，通过集群名称这个属性来区分不同的集群。
   
   node.name: "Franz Kafka"
   # 当前配置所在机器的节点名，你不设置就默认随机指定一个name列表中名字，该name列表在es的jar包中config文件夹里name.txt文件中，其中有很多作者添加的有趣名字。
   
   node.master: true
   指定该节点是否有资格被选举成为node（注意这里只是设置成有资格， 不代表该node一定就是master），默认是true，es是默认集群中的第一台机器为master，如果这台机挂了就会重新选举master。
   
   node.data: true
   # 指定该节点是否存储索引数据，默认为true。
   
   index.number_of_shards: 5
   # 设置默认索引分片个数，默认为5片。
   
   index.number_of_replicas: 1
   # 设置默认索引副本个数，默认为1个副本。如果采用默认设置，而你集群只配置了一台机器，那么集群的健康度为yellow，也就是所有的数据都是可用的，但是某些复制没有被分配
   # （健康度可用 curl 'localhost:9200/_cat/health?v' 查看， 分为绿色、黄色或红色。绿色代表一切正常，集群功能齐全，黄色意味着所有的数据都是可用的，但是某些复制没有被分配，红色则代表因为某些原因，某些数据不可用）。
   
   path.conf: /path/to/conf
   # 设置配置文件的存储路径，默认是es根目录下的config文件夹。
   
   path.data: /path/to/data
   # 设置索引数据的存储路径，默认是es根目录下的data文件夹，可以设置多个存储路径，用逗号隔开，例：
   # path.data: /path/to/data1,/path/to/data2
   
   path.work: /path/to/work
   # 设置临时文件的存储路径，默认是es根目录下的work文件夹。
   
   path.logs: /path/to/logs
   # 设置日志文件的存储路径，默认是es根目录下的logs文件夹 
   
   path.plugins: /path/to/plugins
   # 设置插件的存放路径，默认是es根目录下的plugins文件夹, 插件在es里面普遍使用，用来增强原系统核心功能。
   
   bootstrap.mlockall: true
   # 设置为true来锁住内存不进行swapping。因为当jvm开始swapping时es的效率 会降低，所以要保证它不swap，可以把ES_MIN_MEM和ES_MAX_MEM两个环境变量设置成同一个值，并且保证机器有足够的内存分配给es。 同时也要允许elasticsearch的进程可以锁住内# # 存，linux下启动es之前可以通过`ulimit -l unlimited`命令设置。
   
   network.bind_host: 192.168.0.1
   # 设置绑定的ip地址，可以是ipv4或ipv6的，默认为0.0.0.0，绑定这台机器的任何一个ip。
   
   network.publish_host: 192.168.0.1
   # 设置其它节点和该节点交互的ip地址，如果不设置它会自动判断，值必须是个真实的ip地址。
   
   network.host: 192.168.0.1
   # 这个参数是用来同时设置bind_host和publish_host上面两个参数。
   
   transport.tcp.port: 9300
   # 设置节点之间交互的tcp端口，默认是9300。
   
   transport.tcp.compress: true
   # 设置是否压缩tcp传输时的数据，默认为false，不压缩。
   
   http.port: 9200
   # 设置对外服务的http端口，默认为9200。
   
   http.max_content_length: 100mb
   # 设置内容的最大容量，默认100mb
   
   http.enabled: false
   # 是否使用http协议对外提供服务，默认为true，开启。
   
   gateway.type: local
   # gateway的类型，默认为local即为本地文件系统，可以设置为本地文件系统，分布式文件系统，hadoop的HDFS，和amazon的s3服务器等。
   
   gateway.recover_after_nodes: 1
   # 设置集群中N个节点启动时进行数据恢复，默认为1。
   
   gateway.recover_after_time: 5m
   # 设置初始化数据恢复进程的超时时间，默认是5分钟。
   
   gateway.expected_nodes: 2
   # 设置这个集群中节点的数量，默认为2，一旦这N个节点启动，就会立即进行数据恢复。
   
   cluster.routing.allocation.node_initial_primaries_recoveries: 4
   # 初始化数据恢复时，并发恢复线程的个数，默认为4。
   
   cluster.routing.allocation.node_concurrent_recoveries: 2
   # 添加删除节点或负载均衡时并发恢复线程的个数，默认为4。
   
   indices.recovery.max_size_per_sec: 0
   # 设置数据恢复时限制的带宽，如入100mb，默认为0，即无限制。
   
   indices.recovery.concurrent_streams: 5
   # 设置这个参数来限制从其它分片恢复数据时最大同时打开并发流的个数，默认为5。
   
   discovery.zen.minimum_master_nodes: 1
   # 设置这个参数来保证集群中的节点可以知道其它N个有master资格的节点。默认为1，对于大的集群来说，可以设置大一点的值（2-4）
   
   discovery.zen.ping.timeout: 3s
   # 设置集群中自动发现其它节点时ping连接超时时间，默认为3秒，对于比较差的网络环境可以高点的值来防止自动发现时出错。
   
   discovery.zen.ping.multicast.enabled: false
   # 设置是否打开多播发现节点，默认是true。。、
   
   discovery.zen.ping.unicast.hosts: ["host1", "host2:port", "host3[portX-portY]"]
   # 设置集群中master节点的初始列表，可以通过这些节点来自动发现新加入集群的节点。
   ~~~

   8. elasticsearch目录

      - bin 运行Elasticsearch实例和管理插件的一些脚本
      - config 配置文件， elasticsearch.yml
      - data 在节点上每个索引/碎片的数据文件的位置
      - lib Elasticsearch自身使用的.jar文件
      - logs 日志文件
      - modules
      - plugins 已安装的插件的存放位置

   9. 安装ik分词器

      分词是全文索引中非常重要的部分，Elasticsearch是不支持中文分词的，ik分词器支持中文。

      ~~~
      ./bin/elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v6.3.0/elasticsearch-analysis-ik-6.3.0.zip
      ~~~

   10. 

#### elasticsearch的数据结构

- 索引

  索引是文档的容器，是一类相似文档的集合

  - Index体现了逻辑空间的概念，每个索引都有自己的mapping定义，用于定义包含的文档的字段名和字段类型
  - Shard 体现物理空间的概念，索引中的数据分散到各个Shard上

  索引的Mapping和Setting

  - Mapping是定义索引中的所有文档中的字段类型
  - Setting是定义数据的不同分布

- 节点

- 文档

- 分片

- 