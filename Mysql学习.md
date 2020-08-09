# Mysql

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



### 表更新操作

~~~mysql
#更新表名
ALTER TABLE t_course RENAME t_zq_course;
#原有表新增一列
alter table t_account add column operate_type tinyint(2) DEFAULT '0' COMMENT '操作类型，1代表更新，0新增'; 
#原有表新增多列
ALTER TABLE t_account ADD COLUMN company_name VARCHAR (100) DEFAULT NULL COMMENT '企业名称',
 ADD COLUMN license_no VARCHAR (100) DEFAULT NULL COMMENT '企业组织代码';

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



~~~

