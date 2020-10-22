1. springcloud相关组件？答：配置中心使用appllo，注册中心Eureka，服务调用feign，日志链路跟踪使用的是Slueth等等
2. 有使用过zookeeper作为分布式锁嘛？ 答：没有，我使用的是redisson作为分布式锁
3. 配置中心appllo的优点？怎么实现的动态配置更新？
4. 熔断和降级的区别？

~~~
springcloud相关组件：
	快速开发单体的微服务：Springboot
	注册中心：Eureka
	服务调用：Feign
	日志链路跟踪：Sleuth
	负载均衡：Ribbon
	服务熔断器：Hystrix
	服务网关：Zuul gateway
	配置中心：appllo
	


~~~


#### IDEA创建springcloud多模块项目

~~~
1.构建主工程
首先创建一个Maven项目作为主工程，类型无所谓，这里建议使用maven-archetype-quickstart骨架，创建过程如下：
    File-->New-->Project
    -->Maven-->Create from archetype-->maven-archetype-quickstart-Next
    -->GroupId={你的GroupId}-->AritifactId={你的ArtifactId}
    -->Next-->Next-->Finish-->New Whindow
    
2.构建子模块
模块项目创建于主工程之内，创建过程如下：
    右键点击项目名称-->New-->Module
    选中Spring Initializr-->Next
    -->Group={主工程的GroupId}-->Aritifact={当前模块的ArtifactId}、
	-->Next-->Next-->Finish
	
优化结构
	删除主工程多余目录并不需要在主工程进行任何代码开发，所以删除其src目录。
	编辑主工程pom.xml作为主工程，其pom.xml可以作为其他子模块工程的基准依赖，方便进行统一的版本管理。
    主工程pom.xml：
    <!--子模块工厂配置-->
    <modules>
        <module>module-a</module>
        <module>module-b</module>
    </modules>
    
    子工程pom.xml：
     <!--父工程的依赖-->
    <parent>
        <groupId>pers.hanchao</groupId>
        <artifactId>main-project</artifactId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    



~~~

#### springboot重点知识

~~~
1、springboot的核心注解
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
	
2、springboot的自动配置原理
3、springboot如何使用xml配置
4、springboot核心配置文件
5、springboot的启动原理
	






~~~



#### 常用注解

~~~java
常用注解：

@SpringCloudApplication
@EnableDiscoveryClient
@EnableApolloConfig
@EnableHystrix
@EnableHystrixDashboard
@EnableFeignClients(basePackages = {"com.kk", "application"})
@ComponentScan(basePackages = {"com.kk", "application"})
@EnableTransactionManagement(
        order = Integer.MAX_VALUE - 1
)
public class VertApplication {
    public static void main(String[] args) {
        SpringApplication.run(VertApplication.class, args);
    }
}


/**
@EnableDiscoveryClient
服务注册到注册中心
**/
~~~

~~~

  
  
  
~~~

### 服务调用Feign

~~~

~~~

### Eureka

~~~
分布式系统的CAP理论：
  一致性（C）：所有节点上的数据时刻保持同步
  可用性（A）：每个请求都能收到一个结果，不管是成功或者失败
  分区容错性（P）：系统应该能持续提供服务，即使内部有消息丢失

注册中心Eureka
	Eureka包括两个组件：Eureka server和Eureka client

    Eureka server提供服务注册服务，各个节点启动后，会在Eureka server中进行注册，Eureka server就会存	储所有可用的服务节点。Eureka server本身也是一个服务，搭建单机版的Eureka server注册中心，需要配置取消Eureka server的自动注册逻辑。
    Eureka server通过Register、Get、Renew等接口提供服务的注册、发现、心跳检测等服务。
    
    Eureka client是一个java客户端，同时也是一个内置的、使用轮询负载算法的负载均衡器。向Eureka server发送心跳，默认周期30秒。如果Eureka server在多个心跳周期内没有接收到某个服务的心跳，将会从中心移除掉这个节点，默认周期90秒。
    

	搭建单机版Eureka server
	
	搭建集群版Eureka server
	
	Eureka server安全认证
	
	
	

~~~

#### 熔断与降级

~~~
熔断：分布式系统中，某个A服务由于自身原因或网络引起的不能服务，然后系统自动断开A服务的请求。防止雪崩。
降级：解决资源不足和访问增加，整个系统负荷增加可能触发降级

触发条件不一样：熔断是某个服务引起的，降级是系统整体负荷考虑
处理目标不一样：熔断是处理某个服务，降级是对一个业务层的系统处理（一般先从最外层开始降级）

~~~


