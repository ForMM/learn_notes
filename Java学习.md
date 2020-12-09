# Java知识总结

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

使用场景：

netty
dubbo等rpc框架