# Kafka框架学习

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



### Springboot集成logback、mdc日志打印追踪

两种实现方式mdc日志追踪：拦截器、AOP

理解类：

1. HandlerInterceptorAdapter类

   SpringWebMVC框架拦截器类，用于请求前预处理、处理后的后处理。

   应用场景：1. 日志处理 2. 权限检查，登陆检测  3. 性能监控

2. WebMvcConfigurer接口

### Spring AOP

~~~
ProceedingJoinPoint接口
MethodSignature接口


~~~



### Spring事务

事务逻辑上的一组操作，这个操作里的各个逻辑单元，要么一起成功，要么一起失败。

##### 事务特性（4种）：

原子性：强调事务的不可分割

一致性：事务的执行前后数据完整性保存一致

隔离性：一个事务执行过程中，不受其他事务干扰

持久性：事务一旦结束，数据就持久到数据库

##### 如果不考虑隔离性引发安全性问题：

脏读：一个事务读到了另一个事务的未提交的数据

不可重复读：一个事务读到了另一个事务已经提交的update数据导致多次查询结果不一致

虚读：一个事务读到了另一个事务已经提交的insert的数据导致多次查询结果不一致

##### 解决读问题：设置事务隔离级别



### 深入了解Spring 

spring结构主要有：

spring core container: spring-beans、spring-core、spring-context、spring-expression

spring aop：

spring jdbc:

spring web:

spring test:

#### spring Ioc/Di（控制反转/依赖注入）

Ioc/Di是一种设计理念，利用容器管理Bean的注入，解决Bean之间的依赖关系。

核心类BeanFactory工厂类来注入bean。指定bean采用注解或XML配置文件。



#### spring aop

通过代理模式为目标对象生成代理对象，并将横切逻辑插入到目标方法执行的前后。



#### spring mvc

mvc的核心类是dispatchservlet



