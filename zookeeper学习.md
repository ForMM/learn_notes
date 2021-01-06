# zookeeper学习

### zookeeper是什么？

zookeeper是一个分布式协调服务，可以实现统一命名服务、状态同步服务、集群管理、分布式应用配置项的管理等。简单点说zookeeper=文件系统+监听通知服务。

- 文件系统
- 监听通知机制

### zookeeper单机模式安装

~~~shell
wget http://mirror.bit.edu.cn/apache/zookeeper/stable/apache-zookeeper-3.5.8.tar.gz
tar -zxvf apache-zookeeper-3.5.8.tar.gz 
cp ./conf/zoo_sample.cfg ./conf/zoo.cfg
bin/zkServer.sh start
./zkServer.sh status


~~~

### zookeeper协调技术

### 分布式锁的实现

### zookeeper的服务机制