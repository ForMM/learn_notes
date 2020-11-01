##Java高并发

#### 公平锁和非公平锁在jdk里的体现

​		公平锁：线程获取锁的顺序和调用lock的顺序是一样的，FIFO

​		非公平锁：线程获取锁的顺序和调用lock的顺序无关，全凭运气

#### 重入锁的底层实现

- 重入锁：可以对同一临界资源重复加锁；ReentrantLock、synchronized都是重入锁。ReentrantLock灵活些，可以设置公平锁和非公平锁；synchronized非公平锁。

- ReentrantLock类中的内部静态类Sync继承了AQS（AbstractQueuedSynchronizer）；AQS是一种提供了原子管理同步状态、阻塞唤醒线程以及队列模型的简单框架。

  AQS原理图：

![aqs](/Users/hayashika/program-kk/学习笔记/learn_notes/img/aqs.png)

​	AQS使用一个Volatile的int类型的成员变量来表示同步状态，通过内置的FIFO队列来完成资源获取的排队工作，通过CAS完成对State值	的修改。

​	https://tech.meituan.com/2019/12/05/aqs-theory-and-apply.html

- ReentrantLock的具体用法：

  ~~~java
    ReentrantLock lock = new ReentrantLock();
     //获取锁
     lock.lock();
     //释放锁
     lock.unlock();  
     
  	//加锁一定要对应释放锁
     try {
       lock.lock();
     }catch (Exception e){
  
     }finally {
       lock.unlock();
     }
     
  		//指定线程唤醒
     Condition condition1 = lock.newCondition();
  	 Condition condition2 = lock.newCondition();
  	 condition1.await();//阻塞
  	 condition1.signal();//唤醒
  
  ~~~

#### coutndownlatch的底层实现

​	底层跟ReentrantLock一样的使用内部类Sync继承AQS。

