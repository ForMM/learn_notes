

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

内存溢出（out of memory）：程序运行过程中无法申请足够的内存导致的一种错误。

内存泄露（memory leak）：程序中有些对象不会被GC回收，始终占用内存，无法释放已申请的内存空间。即被分配的对象引用链可达但已无用。一次内存泄露似乎没有多大影响，但是内存泄露堆积后的后果就是内存溢出。

内存溢出的原因：

1. 内存中加载的数据量过于庞大，如一次从数据库取出过多数据。（项目中有次接入方不断上传文件，第三方jar一直加载文件流导致内不断占用内存，最后服务器崩掉）
2. 集合类中有对象引用，使用完后未清空，使得jvm不能回收
3. 代码中存在死循环或者循环中产生过多重复的对象实体
4. 使用的第三方软件bug
5. 启动参数内存值设定的太小

内存溢出的解决方案：

1. 修改jvm启动参数，直接增加内存。（-Xms，-Xmx参数不能忘记加）（增加服务器，通过负载均衡到不同地方）
2. 检查错误日志，查看“OutOfMerory”错误前是否有其他异常和错误
3. 对代码进行走查和分析，找出可能发生内存泄露的位置
   1. 检查代码中是否有死循环或递归调用
   2. 检查是否有大循环重复产生新对象实体
   3. 检查对数据库查询中，是否有一次获得全部数据的查询。一般来说，一次性取个十万条记录到内存中，就可能产生内存溢出。这问题比较隐蔽，上线前数据库中数据较少，不容易出现问题。上线后数据库中数据较多，就可能产生了。所以查询一般要带上分页。
   4. 检查list、map等集合中是否使用完后，未清除的问题。导致这些对象不能被GC。
   5. 使用内存查看工具动态查看内存使用情况

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

1. jps

   jps是jdk提供的一个查看当前java进程的小工具，显示当前所有java进程pid的命令

   ~~~
   -q：仅输出VM标识符，不包括classname,jar name,arguments in main method 
   -m：输出main method的参数 
   -l：输出完全的包名，应用主类名，jar的完全路径名 
   -v：输出jvm参数 
   -V：输出通过flag文件传递到JVM中的参数(.hotspotrc文件或-XX:Flags=所指定的文件 
   -Joption：传递参数到vm,例如:-J-Xms512m
   ~~~

2. 

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

| 参数                    | 说明                                                         | 实例                     |
| ----------------------- | ------------------------------------------------------------ | ------------------------ |
| -Xms                    | 初始堆大小，默认物理内存的1/64                               | -Xms512M                 |
| -Xmx                    | 最大堆大小，默认物理内存的1/4                                | -Xms2G                   |
| -Xmn                    | 新生代内存大小，官方推荐为整个堆的3/8                        | -Xmn512M                 |
| -Xss                    | 线程堆栈大小，jdk1.5及之后默认1M，之前默认256k               | -Xss512k                 |
| -XX:NewRatio=n          | 设置新生代和年老代的比值。如:为3，表示年轻代与年老代比值为1：3，年轻代占整个年轻代年老代和的1/4 | -XX:NewRatio=3           |
| -XX:SurvivorRatio=n     | 年轻代中Eden区与两个Survivor区的比值。注意Survivor区有两个。如:8，表示Eden：Survivor=8:1:1，一个Survivor区占整个年轻代的1/8 | -XX:SurvivorRatio=8      |
| -XX:PermSize=n          | 永久代初始值，默认为物理内存的1/64                           | -XX:PermSize=128M        |
| -XX:MaxPermSize=n       | 永久代最大值，默认为物理内存的1/4                            | -XX:MaxPermSize=256M     |
| -verbose:class          | 在控制台打印类加载信息                                       |                          |
| -verbose:gc             | 在控制台打印垃圾回收日志                                     |                          |
| -XX:+PrintGC            | 打印GC日志，内容简单                                         |                          |
| -XX:+PrintGCDetails     | 打印GC日志，内容详细                                         |                          |
| -XX:+PrintGCDateStamps  | 在GC日志中添加时间戳                                         |                          |
| -Xloggc:filename        | 指定gc日志路径                                               | -Xloggc:/data/jvm/gc.log |
| -XX:+UseSerialGC        | 年轻代设置串行收集器Serial                                   |                          |
| -XX:+UseParallelGC      | 年轻代设置并行收集器Parallel Scavenge                        |                          |
| -XX:ParallelGCThreads=n | 设置Parallel Scavenge收集时使用的CPU数。并行收集线程数。     | -XX:ParallelGCThreads=4  |
| -XX:MaxGCPauseMillis=n  | 设置Parallel Scavenge回收的最大时间(毫秒)                    | -XX:MaxGCPauseMillis=100 |
| -XX:GCTimeRatio=n       | 设置Parallel Scavenge垃圾回收时间占程序运行时间的百分比。公式为1/(1+n) | -XX:GCTimeRatio=19       |
| -XX:+UseParallelOldGC   | 设置老年代为并行收集器ParallelOld收集器                      |                          |
| -XX:+UseConcMarkSweepGC | 设置老年代并发收集器CMS                                      |                          |
| -XX:+CMSIncrementalMode | 设置CMS收集器为增量模式，适用于单CPU情况                     |                          |



#### 服务不可用，怎么排查？

线上机器的cpu使用率逐步增高，最终达到100%导致线上服务不可用。如下性能调优讲解。

#### jvm性能调优

 - 目标：使用较小的内存占用来获得较高的吞吐量或者较低的延迟。cpu load过高、请求延迟、tps降低等，甚至出现内存泄漏（每次垃圾收集使用的时间越来越长，垃圾收集频率越来越高，每次垃圾收集清理掉的垃圾数据越来越少）、内存溢出导致系统崩溃。

 - 目的：**减少GC的频率和Full GC的次数**，过多的GC和Full GC是会占用很多的系统资源。

 - 几个比较重要的指标：

    - 内存占用：程序正常运行需要的内存大小
    - 延迟：由于垃圾回收引起的程序停顿时间
    - 吞吐量：用户程序运行时间占用户程序和垃圾回收占用总时间的比值

   同时满足一个程序内存占用小、延迟低、高吞吐量是不可能的，程序的目标不同，调优时所考虑的方向也不同，在调优之前，必须要结合实际场景，有明确的的优化目标，找到性能瓶颈，对瓶颈有针对性的优化，最后进行测试，通过各种监控工具确认调优后的结果是否符合目标。

 - 调优工具

   调优可以依赖、参考的数据有系统运行日志、堆栈错误信息、gc日志、线程快照、堆转储快照等

    - 系统运行日志：系统运行日志就是在程序代码中打印出的日志，描述了代码级别的系统运行轨迹（执行的方法、入参、返回值等），一般系统出现问题，系统运行日志是首先要查看的日志。
    - 堆栈错误信息：当系统出现异常后，可以根据堆栈信息初步定位问题所在，比如根据“java.lang.OutOfMemoryError: Java heap space”可以判断是堆内存溢出；根据“java.lang.StackOverflowError”可以判断是栈溢出；根据“java.lang.OutOfMemoryError: PermGen space”可以判断是方法区溢出等。
    - gc日志：程序启动时用 -XX:+PrintGCDetails 和 -Xloggc:/data/jvm/gc.log 可以在程序运行时把gc的详细过程记录下来，或者直接配置“-verbose:gc”参数把gc日志打印到控制台，通过记录的gc日志可以分析每块内存区域gc的频率、时间等，从而发现问题，进行有针对性的优化。 
    - 线程快照：根据线程快照可以看到线程在某一时刻的状态，当系统中可能存在请求超时、死循环、死锁等情况是，可以根据线程快照来进一步确定问题。通过执行虚拟机自带的“jstack pid”命令，可以dump出当前进程中线程的快照信息，更详细的使用和分析网上有很多例，这篇文章写到这里已经很长了就不过多叙述了，贴一篇博客供参考：http://www.cnblogs.com/kongzhongqijing/articles/3630264.html
    - 堆转储快照：程序启动时可以使用 “-XX:+HeapDumpOnOutOfMemory” 和 “-XX:HeapDumpPath=/data/jvm/dumpfile.hprof”，当程序发生内存溢出时，把当时的内存快照以文件形式进行转储（也可以直接用jmap命令转储程序运行时任意时刻的内存快照），事后对当时的内存使用情况进行分析。

   调优方式有：

   	1. 用 jps（JVM process Status）可以查看虚拟机启动的所有进程、执行主类的全名、JVM启动参数。
    	2. 用jstat（JVM Statistics Monitoring Tool）监视虚拟机信息 。jstat -gc pid 500 10 ：每500毫秒打印一次Java堆状况（各个区的容量、使用容量、gc时间等信息），打印10次。
    	3. 用jmap（Memory Map for Java）查看堆内存信息，执行jmap -histo pid可以打印出当前堆中所有每个类的实例数量和内存占用。执行 jmap -dump 可以转储堆内存快照到指定文件，比如执行 jmap -dump:format=b,file=/data/jvm/dumpfile_jmap.hprof 3361 可以把当前堆内存的快照转储到dumpfile_jmap.hprof文件中，然后可以对内存快照进行分析。
    	4. 利用jconsole、jvisualvm分析内存信息(各个区如Eden、Survivor、Old等内存变化情况)。内存快照的第三方工具，比如eclipse mat，它比jvisualvm功能更专业，出了查看每个类及对应实例占用的空间、数量，还可以查询对象之间的调用链，可以查看某个实例到GC Root之间的链。

   调优经验：

   ​	JVM配置方面，一般情况可以先用默认配置（基本的一些初始参数可以保证一般的应用跑的比较稳定了），在测试中根据系统运行状况（会话并发情况、会话时间等），结合gc日志、内存监控、使用的垃圾收集器等进行合理的调整，当老年代内存过小时可能引起频繁Full GC，当内存过大时Full GC时间会特别长。

   ​	JVM的新生代和老年代配置多大合适？这个是相对系统而言的，调优就是找答案的过程，物理内存一定的情况下，新生代设置越大，老年代就越小，FULL GC的频率就越高，但FULL GC的时间越短。相反新时代设置越小，老年代设置越大，FULL GC的频率越低，但FULL GC的时间越长。建议如下：

   1. -Xms和-Xmx的值设置成相等，堆大小默认为-Xms指定的大小，默认空闲堆内存小于40%时，JVM会扩大堆到-Xmx指定的大小；空闲堆内存大于70%时，JVM会减小堆到-Xms指定的大小。如果在Full GC后满足不了内存需求会动态调整，这个阶段比较耗费资源。
   2. 新生代的内存尽量设置大一些，让对象在新生代多存活一段时间，每次Minor GC 都要尽可能多的收集垃圾对象，防止或延迟对象进入老年代的机会，以减少应用程序发生Full GC的频率。
   3. 老年代如果使用CMS收集器，新生代可以不用太大，因为CMS的并行收集速度也很快，收集过程比较耗时的并发标记和并发清除阶段都可以与用户线程并发执行。
   4. 方法区大小的设置，1.6之前的需要考虑系统运行时动态增加的常量、静态变量等，1.7只要差不多能装下启动时和后期动态加载的类信息就行。

   代码实现方面，性能出现问题比如程序等待、内存泄漏除了JVM配置可能存在问题，代码实现上也有很大关系：

   1. 避免创建过大的对象及数组：过大的对象或数组在新生代没有足够空间容纳时会直接进入老年代，如果是短命的大对象，会提前出发Full GC。
   2. 避免同时加载大量数据，如一次从数据库中取出大量数据，或者一次从Excel中读取大量记录，可以分批读取，用完尽快清空引用。
   3. 当集合中有对象的引用，这些对象使用完之后要尽快把集合中的引用清空，这些无用对象尽快回收避免进入老年代。
   4. 可以在合适的场景（如实现缓存）采用软引用、弱引用，比如用软引用来为ObjectA分配实例：SoftReference objectA=new SoftReference(); 在发生内存溢出前，会将objectA列入回收范围进行二次回收，如果这次回收还没有足够内存，才会抛出内存溢出的异常。 
      避免产生死循环，产生死循环后，循环体内可能重复产生大量实例，导致内存空间被迅速占满。
   5. 尽量避免长时间等待外部资源（数据库、网络、设备资源等）的情况，缩小对象的生命周期，避免进入老年代，如果不能及时返回结果可以适当采用异步处理的方式等。

 - 方法和步骤

   1. 监控gc的状态
   2. 生成堆的dump文件
   3. 分析dump文件
   4. 分析结果，判断是否优化
   5. 调整gc类型和内存分配
   6. 不断分析和调整

 - 排查步骤

   1. 宿主机器问题

      一般排查机器问题，我们主要考虑几方面问题：内存，CPU，磁盘。内存和CPU问题，通过简单的top命令，可以看到具体的情况

      ~~~shell
      top -p ${pid}
      #查看该进程关联线程情况
      top -H -p ${pid}
      ~~~

   2. JVM内存，是否频繁GC

      jmap是与堆有关的工具，建议在JVM启动时开启 -XX:+HeapDumpOnOutOfMemoryError，该命令可以让应用在发生OOM时，自动生成HeapDump的转储文件，方便问题排查。

      ~~~shell
      #查看系统的堆情况,目前系统占用的年轻代、年老代、永久代的占用比率，还有具体的年轻代子分区的占用情况，Eden Space、From Space和To Space，系统的内存占用比率可以一目了解
      jmap -heap ${pid}
      
      #查询哪些实例占用内存情况
      jmap -histo ${pid} 
      #具体到包名
      jmap -histo ${pid} | grep ${package} 
      
      #查询GC情况，我们可以通过jstat命令，可以查询到每个堆分代的内存占用情况和Young GC和Full GC次数和时间等
      jstat -gcutil ${pid} 1000 10
      ~~~

      - **S0：**幸存1区当前使用比例
      - **S1：**幸存2区当前使用比例
      - **E：**伊甸园区使用比例
      - **O：**老年代使用比例
      - **M：**元数据区使用比例
      - **CCS：**压缩使用比例
      - **YGC：**年轻代垃圾回收次数
      - **FGC：**老年代垃圾回收次数
      - **FGCT：**老年代垃圾回收消耗时间
      - **GCT：**垃圾回收消耗总时间

   3. 线程栈，是否线程暴涨，线程死锁

       	jstack 工具可以获取当前线程运行情况，目前为止笔者没有发现一个简单有效的命令方式去判断系统有没有死锁，一般都是使用jconsole连接后使用其检测死锁的工具和jstack dump出具体线程栈信息后，然后人为分析，这两次方式都不是很便捷。
          如果时间和环境允许，最好是使用dump出来的文件的，然后使用MAT（Memory Analyse Tools）分析，MAT强大的分析功能和可视化界面是个不错的选择。

   4. 排查日志，检查程序代码

#### jvm问题排查案例

	1. 接入方不停上传文件，导致内存不断被占用，最会服务器崩掉。当时立即采用增加服务器，用nignx均衡到不同服务器维持业务不受影响。然后采用导出dump文件，分析代码
 	2. 

#### MAT分析工具

https://blog.csdn.net/yxz329130952/article/details/50288145

https://blog.csdn.net/zhanshenzhi2008/article/details/89070049?utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromMachineLearnPai2%7Edefault-1.control&dist_request_id=1330144.23566.16181350724815687&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromMachineLearnPai2%7Edefault-1.control

Eclipse Memory Analyzer是一个快速且功能丰富的Java堆分析器，可帮助您查找内存泄漏并减少内存消耗。使用Memory Analyzer分析具有数亿个对象的高效堆转储，快速计算对象的保留大小，查看谁阻止垃圾收集器收集对象，运行报告以自动提取泄漏嫌疑者。

1. Heap Dump

堆转储文件，是java进程在某个时间内的快照。它在触发快照的时候保存了很多信息：java对象和类信息。通常在写Heap Dump文件前会触发一次Full GC。

2. 怎样获取dump

   - 通过OOM获取，即在OutOfMemoryError后获取一份HPROF二进制Heap Dump文件，可以在jvm里添加参数：

     ```
     -XX:+HeapDumpOnOutOfMemoryError
     ```

     - 主动获取，即在虚拟机添加参数如下，然后在Ctrl+Break组合键即可获取一份Heap Dump

       ~~~
       -XX:+HeapDumpOnCtrlBreak
       ~~~

     - 使用HPROF agent

       使用Agent可以在程序执行结束时或受到SIGOUT信号时生成Dump文件。配置在虚拟机的参数如下：

       ```
       -agentlib:hprof=heap=dump,format=b
       ```

     - jmap 可以在cmd里执行，命令如下：

       ```
       jmap -dump:format=b file=<文件名XX.hprof> <pid>
       ```

     - 使用JConsole

     - 使用Memory Analyzer Tools的File -> Acquire Heap Dump功能

3. MAT用来做什么

   - 找出内存泄漏的原因
   - 找出重复引用的类和jar
   - 分析集合的使用
   - 分析类加载器

4. MAT使用介绍

   1. overview

      用MAT打开一个hprof文件后一般会进入如下的overview界面，或者和这个界面类似的leak suspect界面，overview界面会以饼图的方式显示当前消耗内存最多的几类对象，可以使我们对当前内存消耗有一个直观的印象。

   2. dominator_tree（支配树）

      支配树可以直观地反映一个对象的retained heap，shallow heap和retained heap:

      - shallow heap:指的是某一个对象所占内存大小。
      - retained heap:指的是一个对象的retained set所包含对象所占内存的总大小。

      retained set指的是这个对象本身和他持有引用的对象和这些对象的retained set所占内存大小的总和。

      支配树主要可以用于诊断一个对象所占内存为什么会不断膨胀，一个对象膨胀，就说明它对应到支配树中的子树就越来越庞大，只要分析这个对象对应的子树，确定那些对象是不应该出现在子树中就可以对问题手到病除。

   3. Histogram

      Histogram是我们使用最多的一个，可以列出内存中的对象，对象的个数及其大小。

      Class Name ： 类名称，java类名
      Objects ： 类的对象的数量，这个对象被创建了多少个
      Shallow Heap ：一个对象内存的消耗大小，不包含对其他对象的引用
      Retained Heap ：是shallow Heap的总和，也就是该对象被GC之后所能回收到内存的总和

      在某一项上右键打开菜单选择 list objects ->with incoming refs 将列出该类的实例。

      快速找出某个实例没被释放的原因，可以右健 Path to GC Roots–>exclue all phantom/weak/soft etc. reference 。用这个方法可以快速找到某个对象的 **GC Root**,一个存在 GC Root的对象是不会被 GC回收掉的.

   4. Leak Suspects

      自动分析内存内存泄漏的原因，可以直接定位到Class，且行数。

   5. Top Consumers

      通过图型列出最大的Object

   

