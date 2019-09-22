# Spring框架学习

通过写接口方式，学习平时用到的工具类。

1. freemarker制作DOCX模版填充自定义内容输出PDF
2. oss|cos等混合云（阿里云、腾讯云、微软云、华为云）上传工具类
3. 微信、支付宝刷脸人脸识别接口
4. 微信、支付宝支付接口
5. 对称加密（sha1 sha256 AES等）加密工具类，设计简单的摘要计算方法
6. http请求工具类



## Spring注解

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



### 返回结果类设计

为了更好兼容各个客户端理解后端的返回结果，需要制定一个返回规范结果类Result。一般会设计成restful格式。也有很多平台会维护自己的一套返回码体系。



### Spring AOP

~~~
ProceedingJoinPoint接口
MethodSignature接口


~~~

### service层需要自行处理异常吗

不需要涉及事务回滚的就处理异常，需要事务回滚的就抛出异常交给controller层处理