##Java高并发

#### java中定义的一些锁

<img src="/Users/hayashika/program-kk/学习笔记/learn_notes/img/1604322358084.jpg" alt="1604322358084" style="zoom:50%;" />

- 悲观锁 VS 乐观锁	

  悲观锁：对于同一个数据的并发操作，悲观锁认为自己在使用数据的时候一定有别的线程来修改数据，因此在获取数据的时候会先加锁，确保数据不会被别的线程修改。java中，synchronized关键字和Lock的实现类都是悲观锁。

  乐观锁：乐观锁认为自己在使用数据时不会有别的线程修改数据，所以不会添加锁，只是在更新数据的时候去判断之前有没有别的线程更新了这个数据。乐观锁在Java中是通过使用无锁编程来实现，最常采用的是CAS算法。

- 自旋锁 VS 适应性自旋锁

  

- 无锁 VS 偏向锁 VS 轻量级锁 VS 重量级锁

- 公平锁 VS 非公平锁

- 可重入锁 VS 非可重入锁

  synchronized、ReentrantLock可重入锁，NonReentrantLock非可重入锁

  首先ReentrantLock和NonReentrantLock都继承父类AQS，其父类AQS中维护了一个同步状态status来计数重入次数，status初始值为0。

  当线程尝试获取锁时，可重入锁先尝试获取并更新status值，如果status == 0表示没有其他线程在执行同步代码，则把status置为1，当前线程开始执行。如果status != 0，则判断当前线程是否是获取到这个锁的线程，如果是的话执行status+1，且当前线程可以再次获取锁。而非可重入锁是直接去获取并尝试更新当前status的值，如果status != 0的话会导致其获取锁失败，当前线程阻塞。

  释放锁时，可重入锁同样先获取当前status的值，在当前线程是持有锁的线程的前提下。如果status-1 == 0，则表示当前线程所有重复获取锁的操作都已经执行完毕，然后该线程才会真正释放锁。而非可重入锁则是在确定当前线程是持有锁的线程之后，直接将status置为0，将锁释放。

-  独享锁 VS 共享锁

  ReentrantLock独享锁，ReentrantReadWriteLock共享锁。

  独享锁也叫排他锁，是指该锁一次只能被一个线程所持有。如果线程T对数据A加上排它锁后，则其他线程不能再对A加任何类型的锁。获得排它锁的线程即能读数据又能修改数据。JDK中的synchronized和JUC中Lock的实现类就是互斥锁。

  共享锁是指该锁可被多个线程所持有。如果线程T对数据A加上共享锁后，则其他线程只能对A再加共享锁，不能加排它锁。获得共享锁的线程只能读数据，不能修改数据。



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

​	CountDownLatch可以把它看作一个计数器，这个计数器的操作是原子操作，同时只能有一个线程去操作这个计数器，也就是同时只能有一个线程去减这个计数器里面的值。底层跟ReentrantLock一样的使用内部类Sync继承AQS。

​	主要场景：有一个任务想要往下执行，但必须要等到其他的任务执行完毕后才可以继续往下执行。

~~~java
//创建一个值为3的计数器
CountDownLatch latch = new CountDownLatch(3); 

//任何调用这个对象上的await()方法都会阻塞，直到这个计数器的计数值被其他的线程减为0为止
latch.await(); 

//计数器里的值减1
latch.countDown();  

~~~

#### Semaphore（计数信号量）

常常被用来控制访问速率。对于某一种资源，我们希望其最多被N个线程同时访问。内部是基于AQS的共享模式。

~~~java
Semaphore semaphore=new Semaphore(3);

//获取令牌锁
semaphore.acquire();

//释放令牌锁
semaphore.release();

//获取当前使用许可数
semaphore.availablePermits()
~~~



#### CyclicBarries(可循环使用屏障)

让一组线程到达一个屏障（也可以叫屏障点）时被阻塞，直到最后一个线程到达屏障时，屏障才会开门，所有被拦屏障拦截的线程才会继续干活，线程进入屏障通过CylicBarrier的await()方法。

~~~java
//创建屏障点为6的锁
CyclicBarrier cyclicBarrier = new CyclicBarrier(6 ,new Runnable(){
  public void run() {
    System.out.println("开始工作");
  }
});

//进入屏障，进行等待最后一个线程到来，才去执行任务
this.cyclicBarrier.await();
~~~



1. 现在有一个方法task，希望只能被10个线程调用，利用Java相关类，应该如何来实现？
2. 公平锁与非公平锁的区别，在java中是怎么体现出来的，重入锁的底层实现原理



