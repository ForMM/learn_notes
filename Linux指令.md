# Linux指令

### 系统相关命令

~~~shell
uname -a;(查看系统信息)
head -n 1 /etc/issue  # 查看操作系统版本
df -h;(查看磁盘情况)
free -m;(查看内存使用量和交换区使用量)
~~~

### 查看端口占用

~~~shell
netstat -tunlp | grep 8080;
##或者
lsof -i 8080;

##-R 是指级联应用到目录里的所有子目录和文件,777 是所有用户都拥有最高权限
sudo chmod -R 777 某一目录;
~~~

### 文件相关命令

~~~shell
whereis svn;(查看文件安装路径)
which svn;(查看运行文件路径)

yum install -y unzip zip;(安装zip软件)

find /app/image/temp/ -name upload_7445edec.tmp;(查找文件，第一个目录，第二个文件名)

tar -czvf target_name.tar.gz  dir_or_file; (将文件压缩成tar包)
zip target.zip dir_or_file;(将文件压缩成zip)

cp -r api /app/tomcat6/webapps/api2;(第一个为源文件，第二个为目标文件)
mv api api2;(修改api为api2)

~~~



### 查找日志相关命令

~~~shell
less /app/tomcat/log/aa.log; (查询日志)
grep pool-26-thread-1 aa.log.2019-07-05 |grep 18:07:05,642 -C 7 --color;
zgrep -a 1aaa55a574494903ad0ee6b273df59f7  /app/full/logs/aa.log.2019-05-17.tar.gz 


~~~

### linux安装配置java

~~~shell
mkdir /usr/java
cd /usr/java
(jdk tar包放到此目录)
tar -zxvf jdk-8u151-linux-x64.tar.gz
vi /etc/profile

#添加如下内容：
export JAVA_HOME=/usr/java/jdk1.8.0_162
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib:$CLASSPATH
export JAVA_PATH=${JAVA_HOME}/bin:${JRE_HOME}/bin
export PATH=$PATH:${JAVA_PATH}

source /etc/profile   （重新加载环境变量）
~~~

### 安装查看字体库

~~~shell
 #安装字体
 yum -y install fontconfig kde-filesystem
 yum -y install kde-i18n-Chinese
 wget http://139.219.12.193:18080/pkg/default/TrueType.tar
 mkdir -p /usr/share/fonts/chinese/
 tar -xvf TrueType.tar -C /usr/share/fonts/chinese/
 fc-cache -v
 
 #查看系统字体
 fc-list
~~~

### mac连接远程服务器

~~~shell
### mac操作远程服务器
#连接服务器
ssh root@192.168.100.100

#上传文件
scp /path/filename root@192.168.100.100:/path

#从服务器上下载文件
scp username@servername:/path/filename /Users/mac/Desktop（本地目录）

#上传目录到服务器
scp -r local_dir username@servername:remote_dir

#从服务器下载整个目录
scp -r username@servername:/root/（远程目录） /Users/mac/Desktop（本地目录）
~~~

### 查看线程的命令

~~~shell
top; 查看系统内存情况
top -p pid;查看指定进程id的内存情况
ps -ef|grep pid;查看线程具体情况

~~~

### 穿透工具

~~~
路径：/usr/proxyy
nohup ./GoProxyServer &
~~~

### vim相关操作指令

~~~
vim rabbitmq.config
按键i进行插入编辑操作
#搜索 the写法：/the     +回车

直接退出：ctrl+Z
~~~

### linux进程通信的方式

- 管道

  内核管理的一个缓冲区，管道的一端连接一个进程的输入，一端连接另一个进程的输出。一个进程输入信息，另一个取出信息。

- 消息队列

  消息队列是消息的链表，存放在内核中并由消息队列标志符标识。

- 共享内存

  一个进程映射一段能够被其他进程访问的内存，允许其他进程进行通信。

- 信号量

  信号量是一个计数器，可以用来控制多个进程对共享资源的访问。

- 信号

  信号是比较复杂的通信方式，用于通知接收进程有某种事情发生

- 套接字

  它是更为通用的进程间通信机制，可用于不同机器之间的进程间通信。



### curl命令

~~~shell
不带有任何参数时，curl 就是发出 GET 请求
curl https://www.baidu.com

curl -d 'uuid=20ad806ac1a0408d9923695c6988c2ba_0' -X POST https://jcloud.gyuncai.cn/signature/get_contract_image

https://www.ruanyifeng.com/blog/2019/09/curl-reference.html
~~~

