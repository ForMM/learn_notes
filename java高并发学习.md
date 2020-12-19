#### java线程创建方式

- Runnable

  它只有一个run()函数，用于将耗时操作写在其中，**该函数没有返回值**。

- Callable

  有一个call()函数，**但是call()函数有返回值**。

- Future

  **Future就是对于具体的Runnable或者Callable任务的执行结果进行**

  **取消、查询是否完成、获取结果、设置结果操作**。get方法会产生阻塞。

  ~~~java
  public interface Future<V> {
      boolean cancel(boolean mayInterruptIfRunning);
      boolean isCancelled();
      boolean isDone();
      V get() throws InterruptedException, ExecutionException;
      V get(long timeout, TimeUnit unit)
          throws InterruptedException, ExecutionException, TimeoutException;
  }
  ~~~

- FutureTask

  FutureTask类实现了RunnableFuture接口，RunnableFuture继承了Runnable接口和Future接口，所以FutureTask作为Runnable被线程执行，又可以作为Future得到Callable的返回值。

  线程池中的submit方法中用到了它。

- CompletableFuture

  

  
#### 线程池原理

~~~java
public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              RejectedExecutionHandler handler)
~~~

- 主要参数
  - 核心线程数：提交一个任务，线程池创建一个新线程执行任务，直到当前线程数等于corePoolSize；如果继续提交的任务被保存到阻塞队列，等待被执行。
  - 最大线程数:
  - 线程空闲时的存活时间
  - 线程空闲时的存活时间单位
  - 任务队列
  - 拒绝策略
  
- 线程池状态

  - RUNNING:运行状态，值也是最小的，刚创建的线程池就是此状态

  - SHUTDOWN:停工状态，不再接收新任务，已经接收的会继续执行

  - STOP:停止状态，不再接收新任务，已经接收正在执行的，也会中断

  - TIDYING:清空状态， 所有任务都停止了，工作的线程也全部结束了

  - TERMINATED:终止状态，线程池已销毁

    <img src="./img/thradpool.png" alt="thradpool" style="zoom:50%;" />

- 关闭线程池的方式

- 底层实现原理
#### 什么是死锁？怎么避免死锁？

死锁是两个或两个以上线程互相持有对方需要的资源，导致这些线程处于等待状态，不能继续执行。当线程进入synchronized代码块时，便占有了该资源，直到执行完该代码块才会释放资源，在这个期间，其他线程无法占用资源。当多个线程互相持有对方资源，会互相等待对方释放资源，如果线程不主动释放资源，就会产生死锁。

死锁产生的一些特定条件：

- 互斥条件：进程对于所分配到的资源具有排它性，即一个资源只能被一个进程占用，直到被该进程释放
- 请求和保持条件：一个进程请求获取资源时阻塞，对方已获得资源的不释放
- 不剥夺条件：任何一个资源被线程占用，其他线程不可剥夺占用
- 循环等待条件：所有等待线程形成一个环路，造成永久阻塞

避免死锁：

- 加锁顺序

  当多个线程需要相同的一些锁，但是按照不同的顺序加锁，死锁就很容易发生。如果能确保所有的线程都是按照相同的顺序获得锁，那么死锁就不会发生。但有时总是不尽人意。

- 加锁时限

  尝试获取锁的时候加一个超时时间，这也就意味着在尝试获取锁的过程中若超过了这个时限该线程则放弃对该锁请求。

- 死锁检测

  可通过系统所设置的检测机构，及时地检测出死锁的发生，并精确地确定与死锁有关的进程和资源。然后解除死锁：采取适当措施，从系统中将已发生的死锁清除掉。

**银行家算法**：

在避免死锁方法中允许进程动态地申请资源，但系统在进行资源分配之前，应先计算此次分配资源的安全性，若分配不会导致系统进入不安全状态，则分配，否则等待。

基本数据结构：

- 可利用资源向量 Available：
- 最大需求矩阵Max：
- 分配矩阵 Allocation：
- 需求矩阵Need：

饥饿：

一个或者多个线程因为种种原因无法获得所需要的资源，导致一直无法执行的状态。一直有线程级别高的暂用资源，线程低的一直处在饥饿状态。

#### java中定义的一些锁

<img src="./img/1604322358084.jpg" alt="1604322358084" style="zoom:50%;" />

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

#### 锁体系synchronized

- 使用方法及原理

  锁住对象：使用 javap -verbose TestClass 查看使用的字节码指令为：monitorenter、monitorexit

  锁住方法：使用 ACC_SYNCHRONIZED 标识，该标识指明了该方法是一个同步方法，JVM 通过该 ACC_SYNCHRONIZED 访问标志来辨别一个方法是否声明为同步方法，从而执行相应的同步调用。

- java对象结构

  Java对象存储在堆（Heap）内存，Hotspot的对象头主要包括两部分数据：Mark Word（标记字段）、Klass Pointer（类型指针）。

  **Mark Word**：默认存储对象的HashCode，分代年龄和锁标志位信息。这些信息都是与对象自身定义无关的数据，所以Mark Word被设计成一个非固定的数据结构以便在极小的空间内存存储尽量多的数据。它会根据对象的状态复用自己的存储空间，也就是说在运行期间Mark Word里存储的数据会随着锁标志位的变化而变化。

  **Klass Point**：对象指向它的类元数据的指针，虚拟机通过这个指针来确定这个对象是哪个类的实例。

  #### Monitor

  Monitor可以理解为一个同步工具或一种同步机制，通常被描述为一个对象。每一个Java对象就有一把看不见的锁，称为内部锁或者Monitor锁。

- Jdk1.6后锁优化

  锁的状态共有四种：**无锁态、偏向锁、轻量级锁和重量级锁**，其中偏向锁和轻量级锁是 JDK1.6 开始为了减少获得锁和释放锁带来的性能消耗而引入的。 四种锁的状态会随着竞争情况逐渐升级，锁可以升级但是不能降级，意味着偏向锁可以升级为轻量级锁但是轻量级锁不能降级为偏向锁，目的是为了提高获得锁和释放锁的效率。

#### 公平锁和非公平锁在jdk里的体现

​		公平锁：线程获取锁的顺序和调用lock的顺序是一样的，FIFO

​		非公平锁：线程获取锁的顺序和调用lock的顺序无关，全凭运气

#### 重入锁的底层实现

- 重入锁：可以对同一临界资源重复加锁；ReentrantLock、synchronized都是重入锁。ReentrantLock灵活些，可以设置公平锁和非公平锁；synchronized非公平锁。

- ReentrantLock类中的内部静态类Sync继承了AQS（AbstractQueuedSynchronizer）；AQS是一种提供了原子管理同步状态、阻塞唤醒线程以及队列模型的简单框架。

  AQS原理图：

![aqs](./img/aqs.png)

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

#### ReadWriteLock（读写锁）

读锁和写锁，多个读锁不互斥，读锁和写锁互斥，写锁与写锁互斥。

~~~java
//创建读写锁
ReentrantReadWriteLock reentrantReadWriteLock = new ReentrantReadWriteLock();

//获取写锁 释放写锁
reentrantReadWriteLock.writeLock().lock();
reentrantReadWriteLock.writeLock().unlock();

//获取读锁 释放读锁
reentrantReadWriteLock.readLock().lock();
reentrantReadWriteLock.readLock().unlock();
~~~

#### LockSupport

基于Unsafe类中的park和unpark方法

1. 不能被实例化(构造函数是私有的)

2. 方法都是静态方法

```
//阻塞当前线程
LockSupport.park();

//唤醒线程
LockSupport.unpark(t1)
```



#### 分布式锁的实现方式

- 基于数据库实现分布式锁

  1. 悲观锁

  2. 乐观锁

  可以关注下mybatis、herbinate的实现加锁的方式。

- 基于缓存（redis）实现分布式锁

  - 使用命令：

    1. setnx key val:当且仅当key不存在时，set一个key为val的字符串，返回1；若key存在，则什么都不做，返回0。

    2. expire key:为key设置一个超时时间，单位为second，超过这个时间锁会自动释放，避免死锁。

    3. del key:删除key，删除成功返回1

  - 实现思想：

    1. 获取锁时，使用setnx加锁，并使用expire设置为锁添加一个有效期，超过有效期则自动释放锁，锁的value值为一个随机的uuid，通过此再释放锁的时候进行判断。

       	2. 获取锁时还需要设置一个获取超时时间，若超过这个时间则放弃获取锁。
          	3. 释放锁时，通过uuid判断是不是该锁，若是该锁，则执行del进行锁释放。

  - 具体实现：

    jedis(不可重入锁)、redission（可重入锁）

  - redis分布式环境获取和释放锁（Redlock）

    1. 获取当前Unix时间，以毫秒为单位。
    2. 依次尝试从N个实例，使用相同的key和随机值获取锁。在步骤2，当向Redis设置锁时,客户端应该设置一个网络连接和响应超时时间，这个超时时间应该小于锁的失效时间。例如你的锁自动失效时间为10秒，则超时时间应该在5-50毫秒之间。这样可以避免服务器端Redis已经挂掉的情况下，客户端还在死死地等待响应结果。如果服务器端没有在规定时间内响应，客户端应该尽快尝试另外一个Redis实例。
    3. 客户端使用当前时间减去开始获取锁时间（步骤1记录的时间）就得到获取锁使用的时间。当且仅当从大多数（这里是3个节点）的Redis节点都取到锁，并且使用的时间小于锁失效时间时，锁才算获取成功。
    4. 如果取到了锁，key的真正有效时间等于有效时间减去获取锁所使用的时间（步骤3计算的结果）。
    5. 如果因为某些原因，获取锁失败（*没有*在至少N/2+1个Redis实例取到锁或者取锁时间已经超过了有效时间），客户端应该在所有的Redis实例上进行解锁（即便某些Redis实例根本就没有加锁成功）。

- 基于zookeeper实现分布式锁

- 

1. 现在有一个方法task，希望只能被10个线程调用，利用Java相关类，应该如何来实现？
2. 公平锁与非公平锁的区别，在java中是怎么体现出来的，重入锁的底层实现原理



