# Mysql

### 隔离级别

1. 读未提交（READ UNCOMMITTED）
2. 读已提交（READ COMMITTED）
3. 可重复读（REPEATABLE READ）
4. 串行化（SERIALIZABLE）

从上往下，隔离强度逐渐增强，性能逐渐变差。采用哪种隔离级别要根据系统需求权衡决定，其中，**可重复读**是 MySQL 的默认级别。

### linux安装mysql1.8

~~~shell
#查询系统是否安装了mysql
rpm -qa |grep -i mysql

#查找mysql对应的文件夹（二选一）
whereis mysql
find / -name mysql

#卸载并删除MySQL安装的组键服务
rpm -ev mysql80-community-release-el8-1.noarch

#删除系统中MySQL的所有文件夹
rm -rf /usr/share/mysql
rm -rf /usr/lib64/mysql
rm -rf /usr/bin/mysql
rm -rf /etc/selinux/targeted/active/modules/100/mysql

rpm -qa|grep mysql;
cd /usr/local/src/;
wget https://dev.mysql.com/get/mysql80-community-release-el7-1.noarch.rpm;
rpm -ivh mysql80-community-release-el7-1.noarch.rpm;
yum install -y mysql-server;
systemctl start mysqld;
yum clean all;
rpm --rebuilddb;
yun update;
systemctl start mysqld;
cat /var/log/mysqlld.log;
systemctl enable mysqld;
mysql -u root -p;

查看密码强度
show variables like 'validate_password%';

create user 'keke'@'%' identified by 'asdf1234';
grant all on *.* to 'keke'@'%' with grant option;
ALTER USER 'keke'@'%' IDENTIFIED WITH mysql_native_password BY 'asdf1234';
select user,host from mysql.user;

##
GRANT语法：   
   GRANT 权限 ON 数据库.* TO 用户名@'登录主机' IDENTIFIED BY '密码'
权限：
   ALL,ALTER,CREATE,DROP,SELECT,UPDATE,DELETE
   新增用户：权限为USAGE,即为："无权限",想要创建一个没有权限的用户时,可以指定USAGE
数据库：
     *.*              表示所有库的所有表
     mylove.*         表示mylove库的所有表
     mylove.loves     表示mylove库的loves表 
用户名：
     MySQL的账户名
登陆主机：
     允许登陆到MySQL Server的客户端ip
     '%'表示所有ip
     'localhost' 表示本机
     '10.155.123.55' 特定IP
密码：
      MySQL的账户名对应的登陆密码
~~~



初次安装mysql，进入终端，进入/usr/local/mysql/bin目录，输入指令进入mysql指令操作界面：

~~~shell
./mysql -u root -p
~~~

依次输入安装时的密码就行。之后安装navicat客户端，当navicat版本不支持最新版mysql（8.0）加密方式，需要执行指令支持传统的密码加密。

~~~shell
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';
FLUSH PRIVILEGES;
exit; ##退出编辑窗口
~~~

### 忘记数据库密码修改密码

~~~
顺序执行命令（第一个cmd窗口）：
cd C:\Program Files\MySQL\MySQL Server 8.0\bin  （window可以在目录栏输入cmd，按下enter键）
net stop mysql
mysqld --console --skip-grant-tables --shared-memory

顺序执行命令（第二个窗口）
mysql -u root -p
//不输入密码直接回车
use mysql
update user set authentication_string='' where user='root';
quit

关闭窗口1；打开第三个窗口：
net start mysql
cd C:\Program Files\MySQL\MySQL Server 8.0\bin
mysql -u root -p   
ALTER USER 'root'@'localhost' IDENTIFIED BY 'root2019';

~~~



### 创建库

~~~mysql
CREATE DATABASE demo;
~~~

### 创建表

~~~mysql
#创建表，每个字段需要添加注释，表也要添加注释，索引也要创建好
CREATE TABLE `d_account` (
  `id` varchar(32) NOT NULL COMMENT 'uuid主键值',
  `account` varchar(40) NOT NULL COMMENT '账号（唯一的标志）',
  `email` varchar(100) DEFAULT NULL COMMENT '邮箱',
  `status` tinyint(1) DEFAULT '0' COMMENT '审核状态，0未认证，1待审核，2审核通过，3审核不通过',
  `create_time` datetime DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  PRIMARY KEY (`id`),
  KEY `idx_account` (`account`) USING BTREE,
  KEY `idx_email` (`email`) USING BTREE,
  KEY `idx_create_time` (`create_time`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='账号表'
~~~



### 表添加更新操作

~~~mysql
#更新表名
ALTER TABLE t_course RENAME t_zq_course;
#插入数据
INSERT INTO t_account(account,email,status,create_time) VALUES('65333@qq.com','65333@qq.com','1','');
#原有表新增一列
alter table t_account add column operate_type tinyint(2) DEFAULT '0' COMMENT '操作类型，1代表更新，0新增'; 
#原有表新增多列
ALTER TABLE t_account ADD COLUMN company_name VARCHAR (100) DEFAULT NULL COMMENT '企业名称',
 ADD COLUMN license_no VARCHAR (100) DEFAULT NULL COMMENT '企业组织代码';

#更新表字段长度
ALTER TABLE t_account MODIFY COLUMN company_name varchar(40);

#更新数据
update  t_account set operate_type=1 where account='aaaaa';

#在现有表添加联合索引
ALTER TABLE t_account ADD INDEX idx_account_status (`account`,`status`); 

~~~



### 查询操作

~~~mysql
 #每日，每周，每月分组统计数据
 select DATE_FORMAT(create_time,'%Y%m%d') days,count(id) count from t_account group by days; 
 select DATE_FORMAT(create_time,'%Y%u') weeks,count(id) count from t_account group by weeks;  
 select DATE_FORMAT(create_time,'%Y%m') months,count(id) count from t_account group by months; 

#查找重名的数据有哪些？按创建时间降序
SELECT title, price, type, create_time FROM t_zq_course WHERE title IN ( SELECT title FROM t_zq_course GROUP BY title HAVING (COUNT(*) > 1)) ORDER BY create_time DESC;



~~~



### mysql的逻辑结构

![mysql-1](\img\mysql-1.png)

mysql架构总共分为三层：

1. 连接线程处理：连接处理、授权认证、安全等。
2. mysql的核心部分：查询解析、分析、优化、缓存以及所有的内置函数；存储过程、触发器、视图等。
3. 存储引擎，存储引擎负责MySQL中数据的存储和提取。服务器通过API和存储引擎进行通信。这些接口屏蔽了不同存储引擎之间的差异，使得这些差异对上层的查询过程透明化。

### 数据库工作流程

![mysql-2](\img\mysql-2.jpg)

- 客户端连接
  1. 连接处理：客户端同数据库服务层建立TCP连接，连接管理模块会建立连接，并请求一个连接线程。如果连接池中有空闲的连接线程，则分配给这个连接，如果没有，在没有超过最大连接数的情况下，创建新的连接线程负责这个客户端。 
  2. 授权认证：在真正的操作之前，还需要调用用户模块进行授权检查，来验证用户是否有权限。通过后，方才提供服务，连接线程开始接收并处理来自客户端的SQL语句。
- 核心服务
  1. 连接线程接收到SQL语句之后，将语句交给SQL语句解析模块进行语法分析和语义分析。
  2. 如果是一个查询语句，则可以先看查询缓存中是否有结果，如果有结果可以直接返回给客户端。
  3. 如果查询缓存中没有结果，就需要真的查询数据库引擎层了，于是发给SQL优化器，进行查询的优化。如果是表变更，则分别交给insert、update、delete、create、alter处理模块进行处理。
- 数据库引擎
  1. 打开表，如果需要的话获取相应的锁。 
  2. 先查询缓存页中有没有相应的数据，如果有则可以直接返回，如果没有就要从磁盘上去读取。
  3. 当在磁盘中找到相应的数据之后，则会加载到缓存中来，从而使得后面的查询更加高效，由于内存有限，多采用变通的LRU表来管理缓存页，保证缓存的都是经常访问的数据。
  4. 最后，获取数据后返回给客户端，关闭连接，释放连接线程。

### 优化点

	1. 尽量保持查询简单且只返回必需的数据，减小通信间数据包的大小和数量是一个非常好的习惯。所以尽量避免使用SELECT *以及加上LIMIT限制。
 	2. 查询缓存，这个慎重使用，缓存的不当操作也会带来很大的系统消耗。query_cache_type设置为DEMAND，这时只有加入SQL_CACHE的查询才会走缓存，其他查询则不会，这样可以非常自由地控制哪些查询需要被缓存。
 	3. 数据库设计方面的优化：
      - 用多个小表代替一个大表，注意不要过度设计
     - 批量插入代替循环单条插入
     - 合理控制缓存空间大小，一般来说其大小设置为几十兆比较合适
     - 可以通过SQL_CACHE和SQL_NO_CACHE来控制某个查询语句是否需要进行缓存

### 索引的数据结构和算法

二叉树：

平衡二叉树

B+tree（多路搜索树），通过左旋形成平衡二叉树

索引分类：

1. 普通索引index

2. 唯一索引

   主键索引：primary key ：加速查找+约束（不为空且唯一）

    唯一索引：unique：加速查找+约束 （唯一）

3. 联合索引

   -primary key(id,name):联合主键索引
   -unique(id,name):联合唯一索引
   -index(id,name):联合普通索

4. 全文索引fulltext

   用于搜索很长一篇文章的时候，效果最好。

5. 空间索引：几乎不用

索引注意点：

1. 冗余和重复索引
2. 删除长期未使用的索引

**优化COUNT()查询**

统计行数，直接使用COUNT(*)，意义清晰，且性能更好

**优化关联查询**

**优化UNION**

MySQL处理UNION的策略是先创建临时表，然后再把各个查询结果插入到临时表中，最后再来做查询。因此很多优化策略在UNION查询中都没有办法很好的时候。经常需要手动将WHERE、LIMIT、ORDER BY等字句“下推”到各个子查询中，以便优化器可以充分利用这些条件先优化。

除非确实需要服务器去重，否则就一定要使用UNION ALL，如果没有ALL关键字，MySQL会给临时表加上DISTINCT选项，这会导致整个临时表的数据做唯一性检查，这样做的代价非常高。当然即使使用ALL关键字，MySQL总是将结果放入临时表，然后再读出，再返回给客户端。虽然很多时候没有这个必要，比如有时候可以直接把每个子查询的结果返回给客户端。