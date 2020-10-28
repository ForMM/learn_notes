# Mysql

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

