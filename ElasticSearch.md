### ElasticSearch学习

#### 倒排索引



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

~~~

~~~

