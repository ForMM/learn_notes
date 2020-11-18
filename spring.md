# Spring框架学习

### Spring注解

~~~java
@Controller
@ResponseBody
@RestController （指定返回类型为json）（相当于：@controller+@ResponseBody）

@Configuration  (配置、启动spring容器，相对于之前spring配置文件中beans标签)
@Configuration+@Bean (启动容器+注册Bean)
@Configuration+@Componet  (启动容器+注册Bean)

@Aspect  (声明一个切面类)
@Pointcut("execution(public * com.example.demo.controller..*.*(..))")  （声明切入点）
@Before("webLog()")   （方法执行前处理）
@Around("webLog()")		（增强环绕）
@After("webLog()")   （方法执行后处理）
@AfterReturning(pointcut = "webLog()", returning = "ret")  （方法返回结果后执行）
@AfterThrowing(pointcut = "webLog()")		(方法执行异常)
备注：日志打印请求路径、类型、参数可以在@Before注解方法中实现，也可以在@Around注解方法中实现。
也可以用@Around实现@Before、@Around注解功能

~~~

### Spring IOC

- spring ioc的实现原理
  	控制反转：有一个依赖关系，从最上层往最下层找出依赖链，从最底层往上一步一步new对象。这个过程交给第三方容器来实现。Ioc/Di是一种设计理念，利用容器管理Bean的注入，解决Bean之间的依赖关系。
    	ioc指spring ioc container，包括beans、core、context、spel。
    	功能是bean的创建、注册、存储、销毁等
    	重点接口和类：BeanFactory、ApplicationContext、WebApplicationContext
    	bean生命周期：实例化、设置属性值、初始化、销毁；
- 容器启动过程：
   	1. web容器（tomcat）提供一个上下文环境，就是ServletContext
      	2. web.xml文件中提供contextLoaderListener，容器启动时触发初始化事件，这个类监听到了此事件就会调用contextInitialized，在这个方法中会初始化一个启动上下文（WebApplicationContext）。然后读取xml文件中bean的配置保存到ServletContext中。
         	3. 初始化servlet，也将其存到ServletContext中。

```xml
	<listener>  
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>  
</listener>
<context-param>  
    <param-name>contextConfigLocation</param-name>  
    <param-value>classpath:spring/applicationContext.xml</param-value>  
</context-param>  


<servlet>  
    <servlet-name>DispatcherServlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>  
    <init-param>  
        <param-name>contextConfigLocation</param-name>  
        <param-value>classpath:spring/dispatcher-servlet.xml</param-value>  
    </init-param>  
    <load-on-startup>1</load-on-startup>
</servlet>  
<servlet-mapping>  
    <servlet-name>DispatcherServlet</servlet-name>  
    <url-pattern>/</url-pattern> 
</servlet-mapping>  
```
bean加载过程：
	1、加载存储介质中的xml文件到Resource中
	2、BeanDefinitionReader读取Resource所指向的配置文件资源，然后解析配置文件。配置文件中每一个<bean>解析成一个BeanDefinition对象，并保存到BeanDefinitionRegistry中
	3、容器扫描BeanDefinitionRegistry中的BeanDefinition，使用Java的反射机制自动识别出Bean工厂后处理后器（实现BeanFactoryPostProcessor接口）的Bean，然后调用这些Bean工厂后处理器对BeanDefinitionRegistry中的BeanDefinition进行加工处理。主要完成以下两项工作：

1）对使用到占位符的<bean>元素标签进行解析，得到最终的配置值，这意味对一些半成品式的BeanDefinition对象进行加工处理并得到成品的BeanDefinition对象；
2）对BeanDefinitionRegistry中的BeanDefinition进行扫描，通过Java反射机制找出所有属性编辑器的Bean（实现java.beans.PropertyEditor接口的Bean），并自动将它们注册到Spring容器的属性编辑器注册表中（PropertyEditorRegistry）；
		4、Spring容器从BeanDefinitionRegistry中取出加工后的BeanDefinition，并调用InstantiationStrategy着手进行Bean实例化的工作；
		5、在实例化Bean时，Spring容器使用BeanWrapper对Bean进行封装，BeanWrapper提供了很多以Java反射机制操作Bean的方法，它将结合该Bean的BeanDefinition以及容器中属性编辑器，完成Bean属性的设置工作
		6、利用容器中注册的Bean后处理器（实现BeanPostProcessor接口的Bean）对已经完成属性设置工作的Bean进行后续加工，直接装配出一个准备就绪的Bean。



> Resource -> BeanDefinition -> BeanWrapper -> Object

​	在Spring 进行 依赖注入的时候，首先把这种资源转化成Resource抽象，通过里面的IO流读取定义的bean。然后再转化成BeanDefinitioin，里面定义了包括构造器注入，以及setter注入的定义。最后通过BeanWrapper这个接口，首先获取定义的构造器注入属性，通过反射中的Constructor来创建对象。基于这个对象，通过java里面的内省机制获取到定义属性的属性描述器(PropertyDescriptor)，调用属性的写入方法完成依赖注入，最后再调用Spring的自定义初始化逻辑，主要包括以下三个扩展点：

- BeanPostProcess，Spring aop就是基于此扩展。
- Init-method，可以在 bean 标签通过 init-method定义，也可以实现InitializingBean
- XXXAware，Spring容器感知类，可以在bean里面获取到 Spring容器的内部属性。



参考地址：https://zhuanlan.zhihu.com/p/29344811
				https://www.jianshu.com/p/1dec08d290c1

### Spring AOP

1. 什么是spring aop？

   面向切面编程，一个横切关注点是一个可以影响到整个应用的关注点，而且应该被尽量地集中到代码的一个地方，例如事务管理、权限、日志、安全等。

2. Spring AOP的关注点和横切关注点有什么区别？

   **关注点是我们想在应用的模块中实现的行为**，不同的关注点（或者模块）可能是库存管理、航运管理、用户管理等

   **横切关注点是贯穿整个应用程序的关注点**，日志、安全和数据转换，它们在应用的每一个模块都是必须的

3. Spring有哪些不同的通知类型

   （1）**前置通知(Before Advice)**，在连接点之前执行的Advice，不过除非它抛出异常，否则没有能力中断执行流。

   （2）**后置通知(After Advice)**，无论连接点是通过什么方式退出的(正常返回或者抛出异常)都会执行在结束后执行这些Advice。

   （3）**围绕通知(Around Advice)**，围绕连接点执行的Advice，就你一个方法调用。这是最强大的Advice。

   （4）**返回之后通知(After Retuning Advice)**，在连接点正常结束之后执行的Advice。

   （5）**抛出（异常）后执行通知(After Throwing Advice)**，如果一个方法通过抛出异常来退出的话，这个Advice就会被执行。

4. Spring aop的代理是什么

   **AOP 代理是一个由 AOP 框架创建的用于在运行时实现切面协议的对象**。Spring AOP默认为 AOP 代理使用标准的 JDK 动态代理。这使得任何接口（或者接口的集合）可以被代理。Spring AOP 也可以使用 CGLIB 代理。这对代理类而不是接口是必须的。

5. 引介、连接点、切入点、织入是什么

   **连接点**表示应用执行过程中能够插入切面的一个点，这个点可以是方法的调用、异常的抛出。

   **切入点**就是连接点的集合；对应Spring中的@Pointcut注解；

   **切面**是通知和切点的结合；对应Spring中的注解@Aspect修饰的一个类；

   **织入**是把切面应用到目标对象来创建新的代理对象的过程；

6. 源码分析

   spring用代理类包裹切面，把他们织入到Spring管理的bean中。也就是说代理类伪装成目标类，它会截取对目标类中方法的调用，让调用者对目标类的调用都先变成调用伪装类，伪装类中就先执行了切面，再把调用转发给真正的目标bean。

https://blog.csdn.net/qukaiwei/article/details/50367761

### Spring事务

事务逻辑上的一组操作，这个操作里的各个逻辑单元，要么一起成功，要么一起失败。

- ##### 事务特性（4种）：

  原子性：强调事务的不可分割

  一致性：事务的执行前后数据完整性保存一致

  隔离性：一个事务执行过程中，不受其他事务干扰

  持久性：事务一旦结束，数据就持久到数据库

- ##### 如果不考虑隔离性引发安全性问题：

  脏读：一个事务读到了另一个事务的未提交的数据

  不可重复读：一个事务读到了另一个事务已经提交的update数据导致多次查询结果不一致

  虚读：一个事务读到了另一个事务已经提交的insert的数据导致多次查询结果不一致

- ##### 解决读问题：设置事务隔离级别

  

### Spring 用到的设计模式





