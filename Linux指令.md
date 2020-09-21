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

#### linux安装配置java

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



#### 安装查看字体库

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

