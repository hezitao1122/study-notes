# Dubbo

1. ## 简介

   1. ### Dubbo

      1. 分布式的服务框架

   2. #### 名词

      1. 注册中心 Register 
   2. #### 服务提供者 provider
      
   3. 服务消费者 consumer
      
   3. ##### 底层原理
   
   ![](..\imgs\dubbo结构.png)
      1. 第一层：service 层，接口层，给服务提供者和消费者来实现的
      2. 第二层：config 层，配置层，主要是对 dubbo 进行各种配置的
      3. 第三层：proxy 层，服务代理层，无论是 consumer 还是 provider，dubbo 都会给你生成代理，代理之间进行网络通信
      4. 第四层：registry 层，服务注册层，负责服务的注册与发现
      5. 第五层：cluster 层，集群层，封装多个服务提供者的路由以及负载均衡，将多个实例组合成一个服务
      6. 第六层：monitor 层，监控层，对 rpc 接口的调用次数和调用时间进行监控
      7. 第七层：protocal 层，远程调用层，封装 rpc 调用
      8. 第八层：exchange 层，信息交换层，封装请求响应模式，同步转异步
      9. 第九层：transport 层，网络传输层，抽象 mina 和 netty 为统一接口
      10. 第十层：serialize 层，数据序列化层
      
   4. #### 工作流程
   ![](..\imgs\dubbo消费.png)
      1. provider向注册中心注册
      2. consumer向注册中心订阅,注册中心会通知consumer注册好的服务
      3. consumer调用provider
      4. consumer和provider都异步通知注册中心
   
2. ### dubbo支持的通信协议和序列化协议

   1. #### 支持的协议:

      1. ##### dubbo协议: 

         1. 默认的协议，

            - 采用单一长连接：建立一个链接来发送多个请求

            - 采用NIO异步通信、

            - 基于hessian作为序列化协议： 把Java对象转为byte的数组；具有跨语言

         2. ##### 使用的场景：

            - 传输数据量小（每次请求数据量在100kb以内）
            - 并发高

      2. #### rmi 协议

         走Java二进制序列化，多个短连接，适合消费者和提供者数量差不多的情况下，适用于一文件传输，一般较少用

      3. #### hessian 协议

         走hessian序列化协议，多个短连接，适用于提供者比消费者数量还多的情况，适用于文件传输，较少用

      4. #### http协议：

         走json序列化

      5. #### webservice

         走SOAP文本序列化协议

   2. ### hessian序列化协议

      1. #### 数据结构
         
         1. 原始的二进制数据
         2. boolean
         3. 64-bit date （64位毫秒值的日期）
         4. 64-bit double
         5. 32-bit long
         6. null
         7. UTF-8 编码的String
      2. #### 三种递归类型
         
         1. list for lists and arrays
         2. map for maps and dictionaries
         3. object for objects
      3. #### 特殊类型
         
         1. ref 用来表示对共享对象的引用

   3. ### 支持的负载均衡策略

      1. #### random loadbalenc: 

         默认的负载均衡策略，即随机调用实现负载均衡，可以对不同provider实例设置不同的权重，会按照权重来负载均衡，权重越大分配流量越高。

      2. #### roundrobin loadbalance

         默认均匀地将流量打到各个机器上去，但是如果各个机器的性能都不一样，容易导致性能差的负载过高。所以此时需要调整权重，让性能差的机器承载权重小一点，流量少一些。

      3. #### leastactive loadbalance

         自动感知,性能差的机器接收到的请求越少

      4. #### consistanthash loadbalance

         一致性hash算法，相同参数的请求一定会发送到一个provider中去。provider挂掉的时候，会基于虚拟节点均匀分配剩余的流量，抖动不会太大。如果需要的不是随机负载均衡，就走这个策略。

   4. ### dubbo的集群容错策略

      1. #### failover cluster模式
         
         1. 失败自动切换，自动重试其他机器。default
      2. #### failfast cluster模式
         
         1. 一次调用失败就立即失败
         2. 常见于非幂等性的写操作
      3. #### failsafe cluster模式
         
         1. 遇到异常就忽略掉,常见不重要的接口调用,如日志
      4. #### failback cluster模式
         
         1. 失败了后台允许自动记录请求,然后定时重发,比较适合消息队列场景
      5. #### forking cluster模式
         
         1. 并行调用多个provider,只要一个成功就立即返回，常见于实时性较高的读操作，但会浪费更多的资源
      6. #### broadcast cluster模式
         
         1. 逐个调用所有的provider.任何一个出错则报错。

   5. ### 动态代理策略

      1. 默认使用javassist动态代理字节码生成，创建代理类。
      2. 可以通过spi扩展机制配置自己的动态代理策略

3. ## SPI机制

   1. ### 是什么？
      
      - 简单来说，就是service provider interface
      - 定义一个接口，但是在运行的时候，选择哪个实现，需要根据指定的配置信息，去找对应的实现类加载进来
   2. ### dubbo的SPI思想
      
      1. dubbo也有spi思想，不过没有用jdk的spi机制，而是自己实现了一套spi机制
      2. #### 如何实现？
         
         1. 定义一个接口，加@SPI("dubbo")注解， 接口名为Protocol
         2. 在resources下定义一个文件/META_INF/dubbo/internal/com.alibaba.dubbo.rpc.Protocol  文件内容为  dubbo = xxx.MyProtocol
         3. 通过SPI机制来提供实现类,是通过dubbo作为默认key 去配置文件里面找的，配置文件名与接口全限定类名一样的，通过dubbo作为key可以找到默认的实现
         4. 如果想要动态的去掉默认的实现类，需要使用@Adaptive,在运行时会针对Protocol生成代理类,这个代理类的两方法里面会有代理代码,代理方法会根据url中的protocol来获取那个key,默认是dubbo，你也可以自己制定，如果指定了别的key，那么就会获取别的实现类的实例了

4. ### 服务治理、降级、失败重试和2超时等

   1. ### 服务治理
      
      1. 调用链路自动生成
      2. 服务访问压力以及时长统计
      3. 服务分层（避免循环依赖）
      4. 调用链路失败监控和报警
      5. 服务鉴权
      6. 每个服务的可用性控制
   2. ### 服务降级
      
      1. 调用失败的时候，可以通过mock返回null
   3. ### 失败重试和超时
      
      1. consumer调用provider要是失败了，如抛出异常，此时应该是可以重试的，或者调用超时也可以重试