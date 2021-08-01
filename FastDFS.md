## FastDFS

FastDFS是一款轻量级的开源分布式文件系统，功能包括：文件存储、文件同步、文件上传、文件下载等，解决了文件大容量存储和高性能访问问题。特别适合以文件为载体的在线服务，如图片、视频、文档服务等等。

libfastcommon从开源项目FastDFS中提取的C公共函数库，这个库非常简单和稳定。提供的函数功能包括：字符串、日志、链表、哈希表、网络通信、ini配置文件读取、base64编码/解码、url编码/解码、时间轮计时器（timer）、跳表（skiplist）、对象池和内存池等等。详细信息请参阅C头文件。

### Linux系统FastDFS安装

通过git下载对应包来进行安装，如果linux系统没有git工具，先安装git，有直接下载FastDFS相关包。

~~~shell
yum install git

#指定一个fastdfs目录， 比如 /app/fastdfs，在这个目录下存放所有的相关包
git clone https://gitee.com/fastdfs100/libfastcommon.git

#进入libfastcommon公共包目录
cd libfastcommon

#依次执行脚本
./make.sh
./make.sh install

#如果系统不支持.sh编译，需要安装perl，gcc;安装好后再执行上面的两个相关脚本
yum -y install perl
yum install gcc-c++

#设置环境变量和软链接
#32位ubuntu中，libfastcommon会安装在/usr/lib 中，64位系统则安装在 /usr/lib64 中。依次执行以下命令：（根据自己的操作系统选择路径）
export LD_LIBRARY_PATH=/usr/lib64/
ln -s /usr/lib64/libfastcommon.so /usr/local/lib64/libfastcommon.so

#指定一个fastdfs目录， 比如 /app/fastdfs，在这个目录下存放所有的相关包
git clone https://gitee.com/fastdfs100/fastdfs.git

#进入fastdfs目录
cd libfastcommon

#依次执行脚本
./make.sh
./make.sh install

#修改配置文件
#有四个配置文件
#client.conf  storage.conf  storage.conf tracker.conf
cd /etc/fdfs/

#修改tracker.conf中日志存放路径，HTTP端口号
#base_path=/app/fastdfs/log
#http.server_port=8090 （我默认没有改）
#注意21200端口需要能够请求通，部署到云服务器需要开通端口访问
vim tracker.conf

#storage.conf中分组名，存储地址，日志地址，HTTP端口号
#group_name=group1
#store_path0=/app/fastdfs/storage0 （这些路径先自己创建好）
#base_path=/app/fastdfs/log
#http.server_port=8090 （我默认没有改）
#tracker_server=192.168.209.121:22122改为：tracker_server=192.168.1.207:22122，这个ip改成自己的
#注意23000端口需要能够请求通，部署到云服务器需要开通端口访问
vim storage.conf

#启动服务
/usr/bin/fdfs_trackerd /etc/fdfs/tracker.conf
/usr/bin/fdfs_storaged /etc/fdfs/storage.conf

#可以查看日志：/app/fastdfs/log/trackerd.log，来判断trackerd是否正常启动起来
#可以查看日志：/app/fastdfs/log/storaged.log，来判断storaged是否正常启动起来

#查看进程
ps -ef|grep fdfs
netstat -unltp|grep fdfs （可以查看监听端口）

#测试上传文件
#/app/image.jpg这个文件存在
fdfs_test /etc/fdfs/client.conf upload /app/image.jpg

#进入fastdfs目录
#fastdfs-nginx-module安装
git clone https://gitee.com/fastdfs100/fastdfs-nginx-module.git
#进入src目录，复制mod_fastdfs.conf 到/etc/fdfs/
cd fastdfs-nginx-module/src
cp mod_fastdfs.conf /etc/fdfs/
#进入/etc/fdfs/
#编辑修改配置文件
#base_path=/tmp改成：base_path=/app/fastdfs/log
#tracker_server=tracker:22122改成：tracker_server=192.168.1.207:22122
#url_have_group_name = false改成：url_have_group_name = true；#url中包含group名称
#store_path0=/home/yuqing/fastdfs改成：store_path0=/app/fastdfs/storage0
vim mod_fastdfs.conf

#nginx安装，下载nginx包，解压
wget http://nginx.org/download/nginx-1.20.1.tar.gz
tar -zxvf nginx-1.20.1.tar.gz
#nignx依赖包安装
yum -y install zlib zlib-devel openssl openssl--devel pcre pcre-devel
#安装nginx并添加fastdfs模块
./configure --prefix=/usr/local/nginx --add-module=/usr/local/fastdfs-nginx-module-master/src
make
make install
#检查nginx模块,是否把fastdfs模块添加进去
cd /usr/local/nginx/sbin/
./nginx -V
#配置nginx配置文件
cd /usr/local/nginx/conf
vim nginx.conf

server {
        listen       80;
        server_name  192.168.1.207;

        location /group1/M00/{
                root /app/fastdfs/storage0/data;
                ngx_fastdfs_module;
        }
    }

#启动nginx
cd /usr/local/nginx/sbin/
./nginx -c /usr/local/nginx/conf/nginx.conf

#启动nginx报错
# ERROR - file: ini_file_reader.c, line: 631, include file “http.conf” not exists, line: “#include http.conf”
#找fastdfs的源码安装目录, conf子目录
#将该子目录中的http.conf,拷贝到/etc/fdfs
#找Nginx的 源码安装目录, conf子目录
#将mime.tyes的文件, 拷贝到/etc/fdfs

#测试上传，浏览器查看
#在当前目录下存在image.jpg图片
fdfs_upload_file /etc/fdfs/client.conf image.jpg
#返回group1/M00/00/00/L3FXAGDxU7CAJWtsAADAbuF0Qjo239.jpg
#浏览器上查看如下地址
#http://192.168.1.207/group1/M00/00/00/L3FXAGDxU7CAJWtsAADAbuF0Qjo239.jpg

~~~





