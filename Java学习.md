### Java基础知识

#### 基础概念

#### 基本语法

##### 	自增自减

​		需要某个整数类型变量增加 1 或减少 1，Java 提供了一种特殊的运算符，用于这种表达式，叫做自增运算符（++)和自减运算符（--）。++和--运算符可以放在变量之前，也可以放在变量之后，当运算符放在变量之前时(前缀)，先自增/减，再赋值；当运算符放在变量之后时(后缀)，先赋值，再自增/减。

##### 	continue、break、和 return 的区别是什么？

  		1. continue ：指跳出当前的这一次循环，继续下一次循环。
  		2. break ：指跳出整个循环体，继续执行循环下面的语句。

return 用于跳出所在方法，结束该方法的运行。return 一般有两种用法：

1. `return;` ：直接使用 return 结束方法执行，用于没有返回值函数的方法

2. `return value;` ：return 一个特定值，用于有返回值函数的方法

   ##### 泛型

   Java 在编译期间，所有的泛型信息都会被擦掉，这也就是通常所说类型擦除 。

   泛型一般有三种使用方式:泛型类、泛型接口、泛型方法。

   **常用的通配符为： T，E，K，V，？**

   - ？ 表示不确定的 java 类型
   - T (type) 表示具体的一个 java 类型
   - K V (key value) 分别代表 java 键值中的 Key Value
   - E (element) 代表 Element

#### 基本数据类型

	1. bit就是位，也叫比特位，是计算机表示数据最小的单位
 	2. byte就是字节
 	3. 1byte=8bit ，0001 1100，一般用两个16进制来显示，所以我们经常看到1个字节显示为 1c
 	4. 1byte就是1B

<img src="/Users/hayashika/program-kk/学习笔记/learn_notes/img/java-datatype-1.jpeg" alt="java-datatype-1" style="zoom:80%;" />

由于数据在计算机中的表示，最终以**二进制**的形式存在，所以有时候使用二进制，可以更直观地解决问题。 
但，二进制数太长了。所以就产生了其他进制数据。

十六进制整型常量：以十六进制表示时，需以0x或0X开头，如0xff,0X9A。

八进制整型常量：八进制必须以0开头，如0123，034。

长整型：长整型必须以L作结尾，如9L,342L。

浮点数常量：由于小数常量的默认类型是double型，所以float类型的后面一定要加f(F)。同样带小数的变量默认为double类型。

如：float f;f=1.3f;//必须声明f。

字符常量：字符型常量需用两个单引号括起来（注意字符串常量是用两个双引号括起来）。Java中的字符占两个字节。

**Java运算符**

<<   :  左移运算符，num <<1,相当于num乘以2

\>>   :  右移运算符，num >>1,相当于num除以2

\>>>  :  无符号右移，忽略符号位，空位都以0补齐，（计算机中数字以补码存储，首位为符号位）。

#### 面向对象

#### 	

​		

#### 反射

#### 异常

#### IO流





### HashMap原理

​	Hashmap允许键值为null，不能保证插入元素的顺序，线程不安全。

- Hashmap的数据结构
  	链表的数组（数组+（单向）链表+红黑树）jdk1.8
  数据结构图要画出来
  	Node<k,v>节点值，里面属性值有hash、key、value、next（node）。
  	![hashmap-1](.\img\hashmap-1.png)
- 数据插入原理
  	1、key的哈希与数组长度取模后的值即是插入位置，如果此位置数组为空，则直接插入元素；如果两个元素相等，则覆盖，不等则在原元素下创建链表的结构存储该元素。
  	2、当链表元素太多了就会转成红黑树结构存储，链表转红黑树的阀值8，红黑树转链表的阀值是6
- 初始化方式
  	两个重要参数：初始化容量大小和加载因子；
  	当数组元素大小大于容量值乘以加载因子的值时数组出现扩容机制。
- 扩容机制
  	新生成一个新数组，原来的老数据需要重新计算哈希值重新分配到新数组中，老数据数据清空。这样很消耗性能。
- 哈希函数怎么设计？这样设计的好处？
  1. 先获取key的hashcode，再与hashcode的高16位与低16位异或运算
  2. 这个叫扰动函数，第一能尽可能降低hash碰撞，第二高效采用位运算
- hashmap线程安全吗？怎么解决这个线程安全问题？
  	线程不安全，1.7并发时扩容会产生环形链和数据丢失的现象，1.8并发时会出现数据覆盖问题。
  	Collections.synchronizedMap（new HashMap）
- concurrenthashmap的分段锁是怎么实现的？
  	数据结构：分段数组加链表，线程安全
  	将数据分成一段一段存储，然后给每一段数据进行加锁，当一个线程占用其中一段数据，其他线程可以访问其他段数据。
- Hashmap里的元素是无序的，有序的Map有LinkedHashMap和TreeMap；他们是怎么实现有序的？
  	linkedHashMap数据结构：数组+单向链表+双向链表（hashmap+双向链表）
  	有序方式有：插入排序和访问排序（acessorder值来控制，false为插入排序）
  	构造方法中初始化了一个只有头节点的双向链表。put数据时将数据插入数组中，还要插入到双向链表中。
  	实现有序的方法addBefore（Entry<k,v> entry）
- TreeMap的数据结构：红黑树	

### 注解实现原理

- 注解声明
- 使用注解的元素
- 操作注解使其起作用(注解处理器)

注解：可以理解为标签，可以在类、方法、属性都可以加上标签。

元注解：

~~~java
@Documented
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@Inherited
public @interface LogAnnotation {
	String value() default "default";
}

@Documented //此注解包含在javadoc中
@Target  //用在什么地方，ElementType.METHOD
/**
CONSTRUCTOR:构造器声明
FIELD:域声明
LOCAL_VARIABLE:局部变量声明
METHOD:方法声明
PACKAGE:包声明
PARAMETER:参数说明
TYPE:类、接口声明
**/
@Retention //需要在什么级别保存该注解信息
/**
SOURCE:注解将被编译器丢弃
CLASS:注解在class文件中可用，但会被JVM丢弃
RUNTIME:运行期也会保存该注解，通过反射机制可以读取注解信息
**/
@Inherited  //允许子类继承父类的注解
~~~

注解处理器：

​	为了运行时能准确获取到注解的相关信息，Java在java.lang.reflect 反射包下新增了AnnotatedElement接口，它主要用于表示目前正在 JVM 中运行的程序中已使用注解的元素，通过该接口提供的方法可以利用反射技术地读取注解的信息，如反射包的Constructor类、Field类、Method类、Package类和Class类都实现了AnnotatedElement接口。

| 返回值                   | 方法名称                                                     | 说明                                                         |
| ------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `<A extends Annotation>` | `getAnnotation(Class<A> annotationClass)`                    | 该元素如果存在指定类型的注解，则返回这些注解，否则返回 null。 |
| `Annotation[]`           | `getAnnotations()`                                           | 返回此元素上存在的所有注解，包括从父类继承的                 |
| `boolean`                | `isAnnotationPresent(Class<? extends Annotation> annotationClass)` | 如果指定类型的注解存在于此元素上，则返回 true，否则返回 false。 |
| `Annotation[]`           | `getDeclaredAnnotations()`                                   | 返回直接存在于此元素上的所有注解，注意，不包括父类的注解，调用者可以随意修改返回的数组；这不会对其他调用者返回的数组产生任何影响，没有则返回长度为0的数组 |

- jdk1.8中注解增强

  新增了个元注解@Repeatable，表示在同一个位置重复相同的注解，需要在定义@FilterPath注解时定义一个数组元素接收多个值。

- 利用注解方式可以实现的场景：

1. 校验类
2. 数据库字段处理
3. 日志追踪
4. 接口权限
5. 安全控制

### try{}catch()finally{}带有return执行顺序

1、finally中的代码总会被执行。

2、当try、catch中有return时，也会执行finally。return的时候，要注意返回值的类型，是否受到finally中代码的影响。

3、finally中有return时，会直接在finally中退出，导致try、catch中的return失效。

### java反射机制

- 什么是反射

  Java反射机制的核心是在程序运行时动态加载类并获取类的详细信息，从而操作类或对象的属性和方法。本质是JVM得到class对象之后，再通过class对象进行反编译，从而获取对象的各种信息。

- 反射的原理

  当执行new对象实例时会触发jvm加载class文件，jvm从本地找到class文件并加载到内存，jvm会自动创建一个class对象，一个类只会产生一个class对象。得到了class对象可以反向获取对象的各种消息。

- 反射的基本使用

  1. 反编译
  2. 通过反射机制访问java对象的属性，方法，构造方法等
  3. 反射最重要的用途就是开发各种通用框架。spring根据xml配置文件配置bean等。

- 获取字节码文件对象的三种方式

  1. Class clazz1 = Class.forName("全限定类名");　　//通过Class类中的静态方法forName，直接获取到一个类的字节码文件对象，此时该类还是源文件阶段，并没有变为字节码文件。

  2. Class clazz2 = Person.class;　　　　//当类被加载成.class文件时，此时Person类变成了.class，在获取该字节码文件对象，也就是获取自己， 该类处于字节码阶段。

  3. Class clazz3 = p.getClass();　　　　//通过类的实例获取该类的字节码文件对象，该类处于创建对象阶段　

- 反射机制能够获取信息

  - 判断是否是某个类的实例

    instanceof 关键字来判断是否为某个类的实例。

  - 创建实例的方式

    1. 使用Class对象的newInstance()方法来创建Class对象对应类的实例。
    2. 先通过Class对象获取指定的Constructor对象，再调用Constructor对象的newInstance()方法来创建对象，这种方法可以用指定的构造器构造类的实例。

### java动态代理

- 静态代理：

  **代理模式可以在不修改被代理对象的基础上，通过扩展代理类，进行一些功能的附加与增强。值得注意的是，代理类和被代理类应该共同实现一个接口，或者是共同继承某个类。**

- 动态代理：

  1. 基于jdk的动态代理

     实现InvocationHandler接口和Proxy类创建代理类实现动态代理

  2. 基于CGLIB的动态代理

     基于POJO类的动态代理，那么CGLIB就是一个很好的选择，在Hibernate框架中PO的字节码生产工作就是靠CGLIB来完成的。通过继承的方式实现动态代理。

### java运行外部依赖的jar

~~~shell
java -Djava.ext.dirs=$JAVA_HOME/jre/lib/ext;d:\tmp\lib -jar d:\tmp\contract-base.jar
~~~

### java浅拷贝和深拷贝

浅拷贝：被复制对象的所有变量都含有与原来的对象相同的值，而所有的对其他对象的引用仍然指向原来的对象。即对象的浅拷贝会对“主”对象进行拷贝，但不会复制主对象里面的对象。”里面的对象“会在原来的对象和它的副本之间共享。简而言之，**浅拷贝仅仅复制所考虑的对象，而不复制它所引用的对象**。

深拷贝：深拷贝是一个整个独立的对象拷贝，深拷贝会拷贝所有的属性,并拷贝属性指向的动态分配的内存。当对象和它所引用的对象一起拷贝时即发生深拷贝。深拷贝相比于浅拷贝速度较慢并且花销较大。简而言之，**深拷贝把要复制的对象所引用的对象都复制了一遍。**

实现深拷贝方式：

1. 必须实现Cloneable

2. 要深拷贝，得注意对象的内部对象，也需要clone

### java NIO

1. 同步与异步

   任务A和任务B都要执行，顺序执行AB任务就是同步，AB可以独立执行就是异步；

2. 阻塞与非阻塞

   任务A调用任务B，必须等到返回结果才能继续执行，任务A等待过程就是阻塞状态；任务A调用任务B，不用等待任务B的执行结果就可以执行接下来的任务，这种事非阻塞。

   同步阻塞IO，数据的读写必须阻塞在一个线程内等待其完成。

   同步非阻塞IO，单线程写数据到buffer，同时可以去做其他的事情，当数据读取到buffer后，线程再继续处理数据。
   
   NIO是一种同步非阻塞的IO模型，java.nio包提供了`Channel` 、`Selector`、`Buffer` 等抽象。
   
   - Buffer
   
     IO面向流，NIO面向缓冲区。Buffer是一个对象，包含要写入或读出的数据。所有的数据都是用缓冲区处理。最常用的缓冲区是ByteBuffer，每一种java基本类型都对应有一种缓冲区。
   
   - Channel
   
     NIO通过通道进行读写，通道是双向的，可读也可写。而流是单向的。无论读写只能和Buffer交互。
   
   - Selector
   
     选择器用于使用单个线程处理多个通道。因此，它需要较少的线程来处理这些通道。线程之间的切换对于操作系统来说是昂贵的。 因此，为了提高系统效率选择器是有用的。


	buffer（IO面向流，NIO面向缓冲区）
	channel（通过通道进行读写）
	selector（单线程处理多个通道）



​	**selector：**多路复用器，主要功能是用于检查一个或者多个NIO Channel（通道）的状态是否处于可读、可写。如此可以实现单线程管理多个Channel（通道），当然也可以管理多个网络连接。

​	每个线程通过一个Selector可以管理多个SocketChannel，为了实现Selector管理多个SocketChannel，必须将具体的SocketChannel对象注册到Selector，并声明需要监听的事件（这样Selector才知道需要记录什么数据），一共有4种事件：
1、connect：客户端连接服务端事件，对应值为SelectionKey.OP_CONNECT(8)
2、accept：服务端接收客户端连接事件，对应值为SelectionKey.OP_ACCEPT(16)
3、read：读事件，对应值为SelectionKey.OP_READ(1)
4、write：写事件，对应值为SelectionKey.OP_WRITE(4)

这个很好理解，每次请求到达服务器，都是从connect开始，connect成功后，服务端开始准备accept，准备就绪，开始读数据，并处理，最后写回数据返回。
所以，当SocketChannel有对应的事件发生时，Selector都可以观察到，并进行相应的处理。

一个SelectionKey键表示了一个特定的通道对象和一个特定的选择器对象之间的注册关系。

​	select()方法返回的int值表示有多少通道已经就绪，是自上次调用select()方法后有多少通道变成就绪状态。之前在调用select()时进入就绪的通道不会在本次调用中被计入，而在前一次select()调用进入就绪但现在已经不在于就绪状态的通道也不会被计入。例如：首次调用select()方法，如果有一个通道变成了就绪状态，返回了1，若再次调用select()方法，如果一个另一个通道就绪了，它会再次返回1.如果对第一个就绪的Channel没有做任何操作，现在就有两个就绪的通道，但在每次select()方法调用之间，只有一个通道就绪了。





使用场景：

netty
dubbo等rpc框架

### Timer

1. schedule（task，time）

   task-所安排的任务  time-执行任务的时间

2. schedule（task，time，period）

   task-所要安排执行的任务 time-首次执行任务的时间 period-执行一次task的时间间隔，单位毫秒

3. schedule(task,delay)

   task-所要安排的任务  delay-执行任务前的延迟时间，单位毫秒

4. schedule(task, delay,period)

   在等待delay毫秒后首次执行task，每隔period毫秒重复执行task


### java事件

JDK为用户实现自定义事件监听提供了两个基础的类。一个是代表所有可被监听事件的事件基类java.util.EventObject,所有自定义事件类型都必须继承该类,类结构如下所示:

~~~java
public class EventObject implements java.io.Serializable {

    private static final long serialVersionUID = 5516075349620653480L;

    /**
     * The object on which the Event initially occurred.
     */
    protected transient Object  source;

    /**
     * Constructs a prototypical Event.
     *
     * @param    source    The object on which the Event initially occurred.
     * @exception  IllegalArgumentException  if source is null.
     */
    public EventObject(Object source) {
        if (source == null)
            throw new IllegalArgumentException("null source");

        this.source = source;
    }

    /**
     * The object on which the Event initially occurred.
     *
     * @return   The object on which the Event initially occurred.
     */
    public Object getSource() {
        return source;
    }

    /**
     * Returns a String representation of this EventObject.
     *
     * @return  A a String representation of this EventObject.
     */
    public String toString() {
        return getClass().getName() + "[source=" + source + "]";
    }
}
~~~

该类内部有一个Object类型的source变量,逻辑上表示发生该事件的事件源,实际中可以用来存储包含该事件的一些相关信息。
另一个则是对所有事件监听器进行抽象的接口java.util.EventListener,这是一个标记接口,内部没有任何抽象方法,所有自定义事件监听器都必须实现该标记接口:

~~~java
/**
 * A tagging interface that all event listener interfaces must extend.
 * @since JDK1.1
 */
public interface EventListener {
}
~~~

针对具体业务场景,我们通过扩展java.util.EventObject来自定义事件类型,同时通过扩展java.util.EventListener来定义在特定事件发生时被触发的事件监听器。