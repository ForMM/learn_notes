# JVM学习

### 主流虚拟机

hotspot

### 自动内存管理机制

#### 1.java内存区域

方法区、虚拟机栈、本地方法栈、堆、程序计数器

方法区：存放类信息（类名、访问修饰符、常量池、字段描述等）、常量、静态变量、及时编译器编译后的代码等

虚拟机栈：存放局部变量表（基本数据类型、对象引用）

~~~java
StackOverflowError
OutOfMemoryError
~~~

本地方法栈：hotspot虚拟机把虚拟机栈和本地方法栈是弄在一起的

堆：存放对象实例、数组，堆内存由年轻代和老年代组成，年轻代分为一个Eden区和两个Survivor区（使用复制收集算法） 

 ![img](https://images2017.cnblogs.com/blog/483385/201801/483385-20180109093559488-1322952384.png) 

程序计数器：

#### 2.异常

内存溢出：程序运行过程中无法申请足够的内存导致的一种错误。

内存泄露：程序中有些对象不会被GC回收，始终占用内存。即被分配的对象引用链可达但已无用。

#### 3.什么样代码产生内存溢出

~~~

~~~

#### 3.垃圾收集算法

造成线程停顿的原因：等待外部资源（数据库连接、网络资源、设备资源等）、死循环、锁等待（死锁和活锁）



### Class文件结构与执行引擎



无符号数和表
=======
无符号数和表组成

类加载过程：加载、连接（校验、准备、解析）、初始化



### Java类加载器







### 常用指令

~~~shell
top     #查系统消耗高的进程
top –H –p  #pid#   根据pid查出进程中子进程
jstack #进程pid# | grep #子线程pid(16)位#    查看占用cpu高的子线程
jmap –heap #pid#     查看堆信息，可以看到堆使用情况和jvm配置参数
jstat –gc #pid#    查看内存使用情况 JVM的GC情况
jmap –dump:format=b,file=/tmp/20190708.hprof #pid#     生成当前jvm内存快照  用于后续分析
jmap –histo:live #pid#     打印每个类的实例数量  内存占用信息  加上live可以手工触发一次fullGC
jstat –gcutil #pid#   查看内存使用百分比，jvm GC情况
jinfo –flags #pid#    可以查看当前进程JVM的所有配置信息
~~~

jmap -heap id

~~~
Attaching to process ID 3764, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.171-b11

using thread-local object allocation.
Parallel GC with 8 thread(s) //采用Parallel GC 

Heap Configuration:
   MinHeapFreeRatio         = 0    //JVM最小空闲比率 可由-XX:MinHeapFreeRatio=<n>参数设置， jvm heap 在使用率小于 n 时 ,heap 进行收缩
   MaxHeapFreeRatio         = 100  //JVM最大空闲比率 可由-XX:MaxHeapFreeRatio=<n>参数设置， jvm heap 在使用率大于 n 时 ,heap 进行扩张 
   MaxHeapSize              = 2095054848 (1998.0MB) //JVM堆的最大大小 可由-XX:MaxHeapSize=<n>参数设置
   NewSize                  = 44040192 (42.0MB) //JVM新生代的默认大小 可由-XX:NewSize=<n>参数设置
   MaxNewSize               = 698351616 (666.0MB) //JVM新生代的最大大小 可由-XX:MaxNewSize=<n>参数设置
   OldSize                  = 88080384 (84.0MB) //JVM老生代的默认大小 可由-XX:OldSize=<n>参数设置 
   NewRatio                 = 2 //新生代：老生代（的大小）=1:2 可由-XX:NewRatio=<n>参数指定New Generation与Old Generation heap size的比例。
   SurvivorRatio            = 8 //survivor:eden = 1:8,即survivor space是新生代大小的1/(8+2)[因为有两个survivor区域] 可由-XX:SurvivorRatio=<n>参数设置
   MetaspaceSize            = 21807104 (20.796875MB) //元空间的默认大小，超过此值就会触发Full GC 可由-XX:MetaspaceSize=<n>参数设置
   CompressedClassSpaceSize = 1073741824 (1024.0MB) //类指针压缩空间的默认大小 可由-XX:CompressedClassSpaceSize=<n>参数设置
   MaxMetaspaceSize         = 17592186044415 MB //元空间的最大大小 可由-XX:MaxMetaspaceSize=<n>参数设置
   G1HeapRegionSize         = 0 (0.0MB) //使用G1垃圾收集器的时候，堆被分割的大小 可由-XX:G1HeapRegionSize=<n>参数设置

Heap Usage:
PS Young Generation //新生代区域分配情况
Eden Space: //Eden区域分配情况
   capacity = 89653248 (85.5MB)
   used     = 8946488 (8.532035827636719MB)
   free     = 80706760 (76.96796417236328MB)
   9.978989272089729% used
From Space: //其中一个Survivor区域分配情况
   capacity = 42467328 (40.5MB)
   used     = 15497496 (14.779563903808594MB)
   free     = 26969832 (25.720436096191406MB)
   36.49275037977431% used
To Space:  //另一个Survivor区域分配情况
   capacity = 42991616 (41.0MB)
   used     = 0 (0.0MB)
   free     = 42991616 (41.0MB)
   0.0% used
PS Old Generation //老生代区域分配情况
   capacity = 154664960 (147.5MB)
   used     = 98556712 (93.99100494384766MB)
   free     = 56108248 (53.508995056152344MB)
   63.722715216167906% used

1819 interned Strings occupying 163384 bytes.
~~~



### jvm参数含义

~~~
-Xms:设置最小堆值（初始堆内存大小，空间不足时再向系统申请扩容）
-Xmx:设置最大堆值
-Xmn:设置年轻代值(设置它等于最小值和最大值相同)
-XX:NewSize:设置年轻代最小值
-XX:MaxNewSize:设置年轻代最大值
-Xss:设置线程栈值大小
-XX:PermSize:设置永久代最小值
-XX:MaxPermSize:设置永久代最大值
-XX:SuriviorRatio:设置年轻代中Eden与s0的比例
-XX:NewRatio:设置老年代与年轻代的比例。
-XX:MinHeapFreeRatio：设置堆空间最小空闲比例。当堆空间的空闲比例小于这个数值时，JVM变主动申请内存空间。
-XX:MaxHeapFreeRation：设置堆空间最大空闲比例。当堆空间的空闲比例大于这个数值时，JVM会压缩堆空间，得到一个较小的堆空间。
-XX:TargetSuriviorRatio：设置surivior空间使用率，当surivior空间使用率达到这个数值时，会将对应的对象送入老年代。
~~~

