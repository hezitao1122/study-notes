# spring-cloud 源码分析系列
##  概述
   🚀 存放的为spring的源码阅读以及笔记
## spring-cloud-netflix
1. ### [spring-cloud-netflix](./spring-cloud-netflix/README.md)
    🚀 存放的为spring-cloud-netflix的源码阅读笔记

>####1). [spring-cloud-netflix-eureka](./spring-cloud-netflix/eureka/README.md)  
 * [eureka源码分析地址](https://github.com/hezitao1122/netflix-eureka-source-code-study)
 * 存储的为netflix-eureka的源码分析
 * 地址为 https://github.com/hezitao1122/netflix-eureka-source-code-study
>>#####(1).EurekaServer整体架构图
>>>  ![](./spring-cloud-netflix/eureka/EurekaServer整体架构设计.png)
>>#####(2).EurekaServer的启动的流程图
>>>  ![](./spring-cloud-netflix/eureka/EurekaServer启动.png)
>>####(3).EurekaServer的服务注册
>>>  ![](./spring-cloud-netflix/eureka/EurekaServer服务注册的最基本.png)
>>####(4).Eureka增量抓取注册表
>>> ![](./spring-cloud-netflix/eureka/Eureka增量抓取注册表.png)
>>####(5).EurekaClient主动下线
>>> ![](./spring-cloud-netflix/eureka/EurekaClient主动下线.png)
>>####(6).EurekaServer多级缓存过期机制
>>> ![](./spring-cloud-netflix/eureka/Eureka多级缓存过期机制.png)
>>####(7).EurekaServer的自我保护机制
>>> ![](./spring-cloud-netflix/eureka/EurekaServer自我保护机制.png)
>>####(8).EurekaServer集群间的同步批处理 - 三级打包
>>> ![](./spring-cloud-netflix/eureka/EurekaServer同步批处理机制.png)

>>####    2). [spring-cloud-netflix-ribbon](./spring-cloud-netflix/ribbon/README.md)
>>  * [ribbon源码分析地址](https://github.com/hezitao1122/netflix-ribbon-source-study)
>>    * 存储的为netflix-ribbon的源码分析
>>    * 地址为 https://github.com/hezitao1122/netflix-ribbon-source-study
>>#####(1).ribbon的大体流程图
>>>  ![](./spring-cloud-netflix/ribbon/ribbon的大体流程图.png)
>>#####(2).@LoadBalanced注解的用途
>>>  ![](./spring-cloud-netflix/ribbon/@LoadBalanced注解.png)
>>#####(3).LoadBalancer加载的过程
>>>  ![](./spring-cloud-netflix/ribbon/LoadBalancer获取的过程.png)
>>#####(4).LoadBalancerInterceptor拦截器的拦截过程
>>>  ![](./spring-cloud-netflix/ribbon/LoadBalancerInterceptor拦截原理.png)
>>#####(5).Eureka和Ribbon整合的基本原理
>>>  ![](./spring-cloud-netflix/ribbon/Eureka和Ribbon整合基本原理.png)
>>#####(6).ribbon整合eureka获取注册表
>>>  ![](./spring-cloud-netflix/ribbon/Ribbon整合Eureka获取服务注册表.png)
>>#####(7).ribbon持续从eureka中获取注册表
>>>  ![](./spring-cloud-netflix/ribbon/ribbon持续从eureka中获取注册表.png)
>>#####(8).ribbon真正执行http请求的流程
>>>  ![](./spring-cloud-netflix/ribbon/ribbon正真执行http请求的流程.png)
>>#####(9).springcloud选择ribbon的一个默认负载均衡算法
>>>  ![](./spring-cloud-netflix/ribbon/springcloud选择ribbon的一个默认负载均衡算法.png)
>>#####(10).ribbon使用的时候,使用默认参数存在的一些问题
>>>  ![](./spring-cloud-netflix/ribbon/Ribbon的负载均衡算法存在的问题.png)
>
>####    3). [spring-cloud-netflix-feign](./spring-cloud-netflix/feign/README.md)
>>#####(1).feign调用的核心流程
>>>  ![](./spring-cloud-netflix/feign/Feign调用的核心流程.png)
>>#####(2).FeignClient的动态代理生成机制
>>>  ![](./spring-cloud-netflix/feign/FeignClient的动态代理.png)
>>#####(3).feign的请求处理机制
>>>  ![](./spring-cloud-netflix/feign/feign请求处理机制.png)
>
>>####    4). [spring-cloud-netflix-hystrix](./spring-cloud-netflix/hystrix/README.md)
>> * [hystrix源码分析地址](https://github.com/hezitao1122/netflix-hystrix-source-study)
>> * 存储的为netflix-hystrix的源码分析
>> * 地址为 https://github.com/hezitao1122/netflix-hystrix-source-study
>>#####(1).hystrix整合feign的代理生成逻辑
>>>  ![](./spring-cloud-netflix/hystrix/Hystrix整合Feign代理生成逻辑.png)
>>#####(2).hystrix执行原理图
>>>  ![](./spring-cloud-netflix/hystrix/hystrix执行原理图.jpg)
>
>>####    5). [spring-cloud-netflix-zuul](./spring-cloud-netflix/zuul/README.md)
>>#####(1).zuul调用的原理图
>>>  ![](./spring-cloud-netflix/zuul/Zuul原理图.jpg)