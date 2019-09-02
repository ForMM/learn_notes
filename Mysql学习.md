# Mysql

### 创建表

~~~mysql
#创建表，每个字段需要添加注释，表也要添加注释，索引也要创建好
CREATE TABLE `t_account` (
  `id` varchar(32) NOT NULL  COMMENT '主键',
  `account` varchar(40) DEFAULT NULL COMMENT '账号',
  `create_time` datetime DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `status` TINYINT(1) DEFAULT 0 COMMENT '状态，0待审核，1已通过'
  PRIMARY KEY (`id`),
  KEY `idx_account` (`account`),
  KEY `idx_create_time` (`create_time`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT '账号表',;
~~~



### 表更新操作

~~~mysql
#原有表新增一列
alter table t_account add column operate_type tinyint(2) DEFAULT '0' COMMENT '操作类型，1代表更新，0新增'; 

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

