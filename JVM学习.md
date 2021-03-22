# JVM学习

#### 主流虚拟机

hotspot

#### jvm内存运行时分为五个数据区域(**Java Memory Model ,JMM**)

![jvm-3](.\img\jvm-3.png)

其中方法区和堆是所有线程共享的，栈，本地方法栈和程序虚拟机则为线程私有的。

- 虚拟机栈

  方法运行中内存模型，每次方法运行时会创建栈帧（局部变量表、操作数栈、动态链接、出口）。每一个方法被调用的过程对应一个栈帧在虚拟机栈中从入栈到出栈的过程。

  局部变量表：一片连续的内存空间，用来存放方法参数，以及方法内定义的局部变量，存放着编译期间已知的数据类型。局部变量表所需要的内存空间再编译期完成分配，当进入一个方法时，这个方法再栈中需要分配多大的局部变量空间是确定的，在方法运行期间不会改变局部变量表大小。

  Java虚拟机栈可能出现两种类型的异常：

  1. 线程请求的栈深度大于虚拟机允许的栈深度，将抛出StackOverflowError。
  2. 虚拟机栈空间可以动态扩展，当动态扩展是无法申请到足够的空间时，抛出OutOfMemory异常。

- 本地方法栈

  hotspot虚拟机把虚拟机栈和本地方法栈是弄在一起的

- 程序计数器 （记录线程执行到哪一步）

- 堆 （共享，存放new出来的实例）

- 方法区（保存常量）

  存放类信息（类名、访问修饰符、常量池、字段描述等）、常量、静态变量、及时编译器编译后的代码等

堆：存放对象实例、数组，堆内存由年轻代和老年代组成，年轻代分为一个Eden区和两个Survivor区（使用复制收集算法） 

 ![img](https://images2017.cnblogs.com/blog/483385/201801/483385-20180109093559488-1322952384.png) 

#### 异常

内存溢出：程序运行过程中无法申请足够的内存导致的一种错误。

内存泄露：程序中有些对象不会被GC回收，始终占用内存。即被分配的对象引用链可达但已无用。

#### 垃圾收集算法

JVM堆主要分为新时代、老年代、元空间（jdk1.7称为永久代）。新时代又分为eden区、survivous0区（s0）、survicous1区（s1）。新生代和老年代的大小比例（1:2）新生代中区域比例（8：1：1）
eden区存放new或newinstance的对象。s0和s1一样大。
第一次GC（yong GC 或 minor GC）: 

1. 第一次GC时s0\s1区是空的，此时将其中一个s0作为存放eden区GC后不能回收的对象。
2. 当eden区GC不能回收的对象沾满了s0区，不能回收的对象就转到老年代区。
3. 清空eden区，此时s1为空。就把s1作为eden区GC后不能回收的对象存放点。
第二次GC
1. 当eden区第二次占满，eden区GC不能回收的对象放到S1区
2. 清空eden区和S0区空间，把S0作为eden区GC后不能回收对象的存放点
第三次第四次以此类推，始终保证s0或s1一个是空的存放点，存储临时对象。GC后没有回收的对象，15次后自动进入老年代。

gc root对象有：
1. 虚拟机栈中引用的对象
2. 方法区中的静态属性引用的对象
3. 方法区中常量引用的对象
4. 本地方法栈中引用的对象（native对象）

什么时候触发young gc、full gc？
当egen区内存占满了，触发young gc。当移到的对象内存大于老年代的剩余内存空间时就会触发full gc。

gc算法：
1.标记-清除算法 （标记存活对象，从gcroot对象出发，遍历所有依赖对象，进行标记，清除没有标记的对象）
2.复制算法
3.标记-整理算法
4.分代收集算法（根据各个年代的特点选择不同的垃圾收集算法）

新生代垃圾回收器：
  Serial （-XX:+UseSerialGC）
  ParNew（-XX:+UseParNewGC）
  ParallelScavenge（-XX:+UseParallelGC）
	G1 收集器
老年代垃圾回收器：
  SerialOld（-XX:+UseSerialOldGC）
  ParallelOld（-XX:+UseParallelOldGC）
  CMS（-XX:+UseConcMarkSweepGC）
  G1 收集器

https://www.jianshu.com/p/76959115d486

#### Class文件结构与执行引擎

#### JVM类加载机制

- 什么是类加载

  在代码编译后，就会生成JVM（Java虚拟机）能够识别的二进制字节流文件（*.class）。而JVM把Class文件中的类描述数据从文件加载到内存，并对数据进行校验、转换解析、初始化，使这些数据最终成为可以被JVM直接使用的Java类型，这个说来简单但实际复杂的过程叫做**JVM的类加载机制**。

  加载、验证、准备、初始化、卸载的**开始顺序**是确定的。

- 类加载

  1、通过一个类的全限定名（包名与类名）来获取定义此类的二进制字节流（Class文件）。而获取的方式，可以通过jar包、war包、网络中获取、JSP文件生成等方式。

  2、将这个字节流所代表的静态存储结构转化为方法区的运行时数据结构。这里只是转化了数据结构，并未合并数据。（方法区就是用来存放已被加载的类信息，常量，静态变量，编译后的代码的运行时内存区域）

  3、在内存中生成一个代表这个类的java.lang.Class对象，作为方法区这个类的各种数据的访问入口。这个Class对象并没有规定是在Java堆内存中，它比较特殊，虽为对象，但存放在方法区中。

- 类的连接

  - 验证

    验证被加载后的类是否有正确的结构，类数据是否会符合虚拟机的要求，确保不会危害虚拟机安全。

  - 准备

    为类的静态变量（static filed）在方法区分配内存，并赋默认初值（0值或null值）。如static int a = 100;

    静态变量a就会在准备阶段被赋默认值0。

  - 解析

    将类的二进制数据中的符号引用换为直接引用。

- 类的初始化

  为静态变量赋程序设定的初值。

  

  

  

  





#### 常用指令

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



#### jvm参数含义

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

#### 服务不可用，怎么排查？

线上机器的cpu使用率逐步增高，最终达到100%导致线上服务不可用。

#### jvm内存调优

 - 目的：**减少GC的频率和Full GC的次数**
 - 方法和步骤
   1. 监控gc的状态
   2. 生成堆的dump文件
   3. 分析dump文件
   4. 分析结果，判断是否优化
   5. 调整gc类型和内存分配
   6. 不断分析和调整
 - 

