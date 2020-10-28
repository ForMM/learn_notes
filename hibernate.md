# Hibernate理解

#### Hibernate工作原理

​	三个核心接口：Configuration、SessionFactory、Session 

#### Hibernate的get和load的区别

#### Hibernate的数据三种状态

#### Hibernate的缓存机制

#### Hibernate的getCurrentsession和openSession的区别

获取数据库的会话

1. getCurrentSr]ession会绑定当前线程，我们在配置hibernate时会让spring来管理事务，这个有事务的线程会绑定当前线程的session。而openSession会创建一个新的session。
2. 

#### Hibernate的乐观锁和悲观锁

#### Hibernate的懒加载机制

#### Hibernate的事务机制	       