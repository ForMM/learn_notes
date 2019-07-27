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
@controller
@restcontroller

@Configuration  (配置、启动spring容器，相对于之前spring配置文件中beans标签)
@Configuration+@Bean (@Configuration+@Componet)  (启动容器+注册Bean)

@Aspect  (声明一个切面类)
@Pointcut("execution(public * com.example.demo.controller..*.*(..))")  （声明切入点）
@Before("webLog()")   （调用切入点方法前处预处理）
@AfterReturning(pointcut = "webLog()", returning = "ret")  （执行后的后处理）


~~~



### Springboot集成logback、mdc日志打印追踪

两种实现方式mdc日志追踪：拦截器、AOP

理解类：

1. HandlerInterceptorAdapter类

   SpringWebMVC框架拦截器类，用于请求前预处理、处理后的后处理。

   应用场景：1. 日志处理 2. 权限检查，登陆检测  3. 性能监控

2. WebMvcConfigurer接口

