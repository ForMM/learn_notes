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

常用注解：

~~~java
@SpringCloudApplication
@EnableDiscoveryClient
@EnableApolloConfig
@EnableHystrix
@EnableHystrixDashboard
@EnableFeignClients(basePackages = {"com.fadada", "application"})
@ComponentScan(basePackages = {"com.fadada", "application"})
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

