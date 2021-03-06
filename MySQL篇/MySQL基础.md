
---
 MySQL基础
---
1. ## MySQL基本概念
   1. #### MySQL驱动是什么东西?

      1. 访问数据库则进行网络连接

      2. 这个网络连接则需要由MySQL驱动创建

      3. 基于网络连接建立各种CRUD操作

   2. #### 数据库连接池解决了什么问题?

      1. 当并发过高时候，如果此时只有一个数据库连接，多个线程抢占一个数据库连接，肯定会导致效率低下

      2. 如果每个请求去访问数据库都创建一个连接，使用后进行销毁，那么创建和销毁线程的开销过大，会导致程序效率低下

      3. 故引进连接池，需要的时候从池里取，不需要的时候放回池里

   3. #### 数据库连接池是用来干什么的？

      1. 维护了server与DB之间的多个连接
      2. 进行权限校验和验证

2. ### MySQL架构设计

![](../imgs/MySQL基础架构.png)

1. ##### 用户发送请求到Tomcat过程
   
1. Tomcat收到请求后，获取连接池，将SQL通过连接池发送给MySQL
   
   2. 把MySQL当黑盒来执行
   
      MySQL的网络连接必须由线程来处理
   
2. ##### 有线程监听网络请求

3. ##### 取出网络请求的参数，交由SQL接口

4. ##### SQL接口

   负责处理接口收到的SQL

5. ##### 查询解析器

   让MySQL的储存引擎能够看得懂SQL

6. ##### 查询优化器：

   1. 查询一个最优的查询路径

   2. 针对编写的SQL生成查询路径树

   3. 从里面选择一条最优查询路径

7. ##### 调用存储引擎接口，执行真正的SQL语句：

   1. 数据库服务并不知道数据库是存放在哪里，数据管理都是由存储引擎进行控制的。

   2. 故需要引入存储引擎：
      
      按照要求、步骤去查询内存和缓存的数据，更新磁盘数据等。

8. ##### 执行器

   1. 根据执行计划调用存储引擎接口

   2. 根据优化器生成的一套执行计划，然后不停地调用存储引擎的各种接口，去执行SQL语句



### InnoDB存储引擎的架构

#### InnoDB的存储结构

![](../imgs/InnoDB写入流程.png)

1. ##### InnoDB会先查询数据是否存在，如果存在，直接加独占锁。如果不存在，则将磁盘里的数据读取到Buffer Pool进行缓存,并加独占锁

2. ###### undo日志文件，万一事务执行失败，还原数据：
   
   保存了原始的数据，可以让事务执行失败的时候进行事务混滚

3. ##### 更新Buffer Pool中的缓存数据
   
   1. 完成1、2两步之后，则开始更新缓冲池中的数据
   
   2. 此时由于只更新了缓冲池中的数据，并未更新数据库文件中的数据，所以为脏数据

4. ##### Redo Log Buffer 万一系统宕机，避免数据丢失
   
   1. 此时内存中的数据修改了，但文件中的未进行修改，宕机了怎么办？
   
   2. 把内存中的修改写入一个Redo Log Buffer中去，也是一个内存缓冲区

5. ##### 事务未提交就宕机了怎么办？
   
   此时如若宕机，由于数据仅写入了内存中，未写入文件磁盘中，所以此时宕机是无光紧要的。

6. ##### 提交事务的时候，将redo日志写入磁盘中
   
   1. 此时会根据一定的策略把redo日志从redo log buffer中刷入文件系统里去，是通过innodb_flush_log_tx_commit这个参数要控制的.
   2. 提交日志成功,redo日志肯定会出现在磁盘上,所以如果此时宕机,buffer pool的数据刷入失败,InnoDB也可以从redo日志中导入恢复数据
   3. 如果此值是2，提交的时候会把redo日志先写入磁盘文件对应的系统缓存（OS Cache）中去，可能隔一秒或数秒才刷入磁盘中去。
   4. 如果此值是1，则代表事务提交得立即刷入磁盘中
   5. 值是0，则代表不写入磁盘
   
7. #### binlog日志是什么东西?

   1. ##### 日志分类

      - 重做日志: 对哪个数据也的什么记录，做了个什么修改，如redo日志。
      - 归档日志：记录的是偏向逻辑型的日志，如对name做了更新操作，值是xx。

   2. MySQL属于自己的日志文件，不是属于存储引擎

   3. 存储引擎在提交日志的时候，同时会写入binlog日志

8. #### 执行器的作用以及顺序

   1. 将磁盘数据加载数据到Buffer Pool，并加独占锁
   2. 将数据写入undo log日志，更新Buffer Poo
   3. 将更新写入redo log buffer中，并根据配置刷入磁盘中
   4. 写入binlog日志

9. #### binlog日志的刷盘策略

   1. 有个sync_binlog参数，默认值是0
   2. 为0的情况下，为先写入os cache，再刷入磁盘中
   3. 为1的情况下，会在踢脚事务的时候，强制写入磁盘中去

10. #### 基于bin log日志和redo log日志完成事务的提交

    1. 完成binlog日志写入后，就会完成最终的事务提交，此时会把本次更新对应的binlog文件名和更新binlog日志的位置，都写入到redo log文件里去。
    2. 然后给redo log文件加上commit标识符
    3. commit标识符是用来表示redo log日志和binlog日志的一致性的，仅有redo log和binlog都写入成功了，且数据都为一致性的，才代表成功了。

11. #### 后台随机讲内存的脏数据刷入到磁盘中

    1. MySQL后台有一定的线程，会随机讲修改后的数据刷入到磁盘中
    2. 在刷入前宕机： 重启后会根据redo日志恢复之前提交的事务，将做过的修改加载到内存中去

### MySQL生产环境部署

1. #### 一般需要什么环境的机器？

   1. 如果只有几百上千人的系统，那么什么机器影响不大。
   2. Java服务器，如果4C8G，每秒大概能承受500左右并发量。只是个大概，与每个请求的请求时间有关。
   3. MySQL通常使用8C16G，甚至使用16C32G机器
   4. Java系统的主要压力和负担都是集中在MySQL数据库上，所以数据库往往是负载最高的。
   5. 8C16G通常扛一两千并发是没问题
   6. 16C32G通常扛三四千并发是没问题
   7. 磁盘最好为SSD

2. #### 互联网公司的生产需求？

   1. ##### 应该首先要进行数据库规划，清楚大概有多少压力

   2. ##### MySQL应该让DBA去安装，DBA会增加一些MySQL调优参数、Linux内核调优参数等。

   3. ##### 应该进行压测

      需要进行基准测试，如模拟1000请求，观察机器cpu负载、磁盘IO、网络负载、内存负载情况等。

   4. ##### QPS和TPS

      1. QPS： 

         Query Per Second 即数据库每秒可以处理多少个请求，大致理解为数据库每秒能处理多少个SQL

         对于一个由多个服务组成的系统，每秒能接收的并发数为QPS

      2. TPS

         Transaction Per Second 即每秒处理的事务量，即提交+回滚的事务，一般一个事务包含多个SQL

         每秒能完成多少笔交易为TPS的概念，一个请求多个子系统的请求调用

   5. ##### IO压测性能的指标：

      1. IOPS：

         机器随机IO并发能力，即随机读写能力（太慢影响随机输啊盘速度）

      2. 吞吐量

         磁盘存储每秒可读多少字节的数据量。（影响事务提交的时候redo日志的顺序读写）

      3. Latency往磁盘写入一条数据的延时

         提交时需顺序写入redo log，故写入时间延迟与事务时间正相关

   6. ##### 其他性能指标

      1. CPU负载
      2. 网络负载
      3. 内存负载

3. ### 数据库压测

   1. #### 压测工具： sysbench功能

      1. 帮助构造大量数据
      2. 模拟几千个线程并发访问数据库
      3. 包括模拟出各种事务提交到数据库的场景
      4. 可以模拟出几十万TPS的压测场景

   2. #### 压测模式

      1. oltp_read_only模式: 只读性能测试
      2. oltp_read_writer模式: 综合性能测试
      3. oltp_delete模式: 删除性能测试
      4. oltp_update_index模式: 更新使用索引性能测试
      5. oltp_update_none_index模式: 更新不使用索引性能测试
      6. oltp_insert: 插入性能测试
      7. oltp_write_only: 写入性能测试

   3. #### 线程:

      1. 初始的时候可用10，然后逐渐地增加线程数，让其承载更高的QPS，一直到极限

   4. #### 机器的其他性能，

      在逐渐的加大线程时，查看硬件性能指标

   5. #### 如何观察硬件性能指标？

      1. ##### CPU性能： TOP命令的第一行

         top - 15:00:00  up 42:35 , 1 user , load average : 0.15 , 0.05 , 0.01

         - 42:35 -> 机器的运行时间
         - 1 user -> 一个用户在使用
         - load average 0.15 , 0.05 , 0.01  第一个数值代表CPU在一分钟的负载情况 , 第二个是五分钟,第三个是15分钟 。此值是最大值为 cpu的核心数

      2. ##### 内存负载情况： 也为TOP命令

         Men:    3G total   , 2G userd , 1.2G free  , 0.3G buffers

         - total -> 总内存空间
         - userd -> 已使用内存空间
         - free -> 空闲空间
         - buffers -> os内核缓冲区空间

      3. ##### 磁盘IO情况: dstat -d 命令或 dstat -r 命令

         -ask/total 

         | read | writer |
         | ---- | ------ |
         | 103k | 211k   |
         | 0    | 11k    |

         上述数据代表每秒读103K ,写211k

         -io/total

         | read | write |
         | ---- | ----- |
         | 0.25 | 31.9  |
         | 0    | 253   |

         上述代表读的IOPS和写的IOPS

      4. ##### 网卡流量使用情况： dstat -n命令

         -net/total

         | recv | send |
         | ---- | ---- |
         | 16k  | 17k  |

         每秒网卡接收到流量有多少k，发送多少k。

         千兆网卡一般传输速度在100M左右

   6. #### 生产环境数据库部署方案：

      1. ### 不能让数据库裸奔

         需对数据库进行监控，如cpu、内存、网络、磁盘IO、慢查询、QPS等

      2. ### 可视化监控平台

         - Promethus： 监控数据采集系统
         - Grafana： 将采集后的数据组成一些报表，供可视化监控展示

      





