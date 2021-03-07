### springboot重点知识

- springboot核心注解

  ~~~
  @SpringBootApplication 启动类的注解；这个注解组合了@SpringBootConfiguration
  @EnableAutoConfiguration @ComponentScan的注解。
  @SpringBootConfiguration：@Configuration ;相当于传统的xml配置文件
  @EnableAutoConfiguration：开启自动配置，根据依赖的jar包自动配置项目。比如：配置tomcat、加载web.xml文件、mvc插件等。
  @ComponentScan：自动发现扫描组件，扫描到@Service、@Controller、@Component等这些注解，并注册为bean。
  @ResponseBody:返回结果写入HTTP response body，一般把json数据写入
  @Controller:控制器负责将用户发来的URL请求转发到对应的服务接口（service层）
  @RestController:@ResponseBody和@Controller的合集
  @RequestMapping:提供路由信息，负责URL到Controller中的具体函数的映射
  @Import:用来导入其他配置类
  @ImportResource:用来加载xml配置文件
  @Autowired:自动导入依赖的bean
  @Service:修饰service层的组件
  @Repository:确保DAO或者repositories提供异常转译；被ComponetScan发现并配置
  @Bean:用@Bean标注方法等价于XML中配置的bean
  @Value：注入Spring boot application.properties配置的属性的值
  @Component：泛指组件，当组件不好归类的时候，我们可以使用这个注解进行标注
  @Configuration：标明为配置类
  ~~~

- springboot自动配置原理

  springboot有一个配置文件application.properties，配置文件中有server.port这些配置。他们是如何生效的？

  1. springboot项目中启动类必须设置@SpringBootApplication注解，这个包含@SpringBootConfiguration
     @EnableAutoConfiguration @ComponentScan的注解。自动配置的实现靠@EnableAutoConfiguration注解来实现。

  2. @EnableAutoConfiguration注解使用了@Import注解，@Import导入AutoConfigurationImportSelector.class类。这个类就来处理需要自动配置的类，他里面SpringFactoriesLoader.loadFactoryNames会扫描到spring-boot-autoconfigure-x.x.x.jar包里的META-INF/spring.factories文件中的配置类信息。

  3. spring-boot-autoconfigure-x.x.x.jar包里很多配置类（aop、amqp、elasticserch等），每一个XxxxAutoConfiguration自动配置类都是在某些条件之下才会生效的，不会全部进行加载。这些条件的限制在Spring Boot中以注解的形式体现，常见的**条件注解**有如下几项：

     ~~~java
     @ConditionalOnBean：当容器里有指定的bean的条件下。
     
     @ConditionalOnMissingBean：当容器里不存在指定bean的条件下。
     
     @ConditionalOnClass：当类路径下有指定类的条件下。
     
     @ConditionalOnMissingClass：当类路径下不存在指定类的条件下。
     
     @ConditionalOnProperty：指定的属性是否有指定的值，比如@ConditionalOnProperties(prefix=”xxx.xxx”, value=”enable”, matchIfMissing=true)，代表当xxx.xxx为enable时条件的布尔值为true，如果没有设置的情况下也为true。
     ~~~

     **@ConfigurationProperties**，它的作用就是从配置文件中绑定属性到对应的bean上，而**@EnableConfigurationProperties**负责导入这个已经绑定了属性的bean到spring容器中。

     一定要记得XxxxProperties类的含义是：封装配置文件中相关属性；XxxxAutoConfiguration类的含义是：自动配置类，目的是给容器中添加组件

- springboot启动原理

  启动类：

  ~~~java
  @SpringBootApplication
  public class CommonConfigApplication {
    public static void main(String[] args) {
      SpringApplication.run(CommonConfigApplication.class, args);
    }
  }
  ~~~

  - SpringApplication.run方法首先会new SpringApplication()进行初始化的操作

    1. 根据classpath下是否存在（ConfigurableWebApplicationContext）判断是否要启动一个WebApplicationContext
    2. 设置属性List<ApplicationContextInitializer<?>> initializers和List<ApplicationListener<?>> listeners中途读取了类路径下所有META-INF/spring.factories的属性，并缓存到了SpringFactoriesLoader的cache缓存中，而这个cache会在本文中用到
    3. 推断主类，并赋值到属性mainApplicationClass

  - 创建监听器SpringApplicationRunListeners,然后starting()，监听SpringApplication的启动

  - 加载springboot的配置环境（ConfigurableEnvironment），如果是web容器，加载StandardEnvironment。并把配置环境（environment）放入监听器中。

  - Banner属性设置

  - 用配置上下文（ConfigurableApplicationContext）创建

  - perpareContext方法将监听器、配置环境、banner等与配置上下文相关联

  - refreshContext(context)方法将bean实例化并注入ioc容器

    refresh()方法做了很多核心工作比如BeanFactory的设置，BeanFactoryPostProcessor接口的执行、BeanPostProcessor接口的执行、自动化配置类的解析、spring.factories的加载、bean的实例化、条件注解的解析、国际化的初始化等等

  - 最后springboot的收尾工作

- springboot内嵌tomcat启动原理

  通过注解@SpringBootApplication和SpringApplication.run方法配置属性、获取监听器，发布应用开始启动事件初、始化输入参数、配置环境，输出banner、创建上下文、预处理上下文、刷新上下文、再刷新上下文、发布应用已经启动事件、发布应用启动完成事件。在SpringBoot中启动tomcat的工作在刷新上下文这一步。而tomcat的启动主要是实例化两个组件：Connector、Container，一个tomcat实例就是一个Server，一个Server包含多个Service，也就是多个应用程序，每个Service包含多个Connector和一个Container，而一个Container下又包含多个子容器。

- springboot手动和自动注入bean

  ​	手动注入：实现ApplicationContextAware接口，利用ApplicationContext获取bean
  ​    自动注入：
  ​    a、@ComponentScan
  ​    b、@Configuration+@Bean
  ​    c、@Import({ImportDemo.class})

- 如何禁用指定的自动配置类

  ~~~java
   @EnableAutoConfiguration(exclude={DataSourceAutoConfiguration.class})
  ~~~

- springboot监听器

  运行状态监控的功能；使用时倒入spring-boot-starter-actuator的依赖即可

- 核心配置文件方式

  application 和 bootstrap 文件；
  	bootstrap 配置文件是系统级别的，用来加载外部配置，如配置中心的配置信息，也可以用来定义系统不会变化的属	性.bootstatp 文件的加载先于application文件；
  	application 配置文件是应用级别的，是当前应用的配置文件；

- springboot如何使用xml配置

  @Configuration和@ImportResource 配置即可,location传入的是一个字符串数组,所以可以传入多个xml配	置.
  	如：@ImportResource(locations = {"classpath:beans.xml"})

  ​	

#### springboot给静态变量注入值

~~~java
@Autowired
priavte static BeanClass beanname;
~~~

无法实例化这个变量，就会出现NullPointerException。原因：

1. 静态变量不是对象的属性，是类的属性。
2. 类加载的时候已经初始化了此变量。
3. 初始化此配置时还未通过spring容器实例化。

解决方法：

~~~java
priavte static BeanClass beanname;

public static BeanClass getBeanname(){
	return beanname;
}

@Autowired
public void setBeanname(BeanClass bean){
	Utils.beanname = bean;
}
~~~

为变量添加get set方法，一定要注意，这里的set方法不是静态的。

同样@value注解实现一些配置项的值的注入，同样的处理方式。

