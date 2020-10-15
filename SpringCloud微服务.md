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
	服务网关：Zuul
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



#### 注册中心Eureka

~~~
分布式系统的CAP理论：
  一致性（C）：所有节点上的数据时刻保持同步
  可用性（A）：每个请求都能收到一个结果，不管是成功或者失败
  分区容错性（P）：系统应该能持续提供服务，即使内部有消息丢失
  
  
  
~~~

