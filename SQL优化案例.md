# 案例一

1. ## 背景
   - 电商平台亿级数据量商品系统的SQL优化

   - 由于存在多个索引,MySQL选择了一个不太适合的索引,导致了慢查询

   - 语句

      select * from products where category = 'xxx' and sub_category = 'xx' order by id desc limit 1000,10

   - 有一个索引 my_index(category , sub_category)

2. ## 排查

   - 使用explain关键字观察，并未使用我们所指定的索引my_index，而是使用的index方式，即走id聚簇索引

   - extra里包含了using where

   - 使用语句强制其使用索引

      select * from products force my_index(category , sub_category) where category = 'xxx' and sub_category = 'xx' order by id limit 1000,10
   
3. 依旧存在的问题

   - 为什么MySQL会选择聚簇索引去走全表扫描？
   - 为什么不使用 my_index(category , sub_category)索引？
   - 即使使用了聚簇索引，为什么之前的查询没有问题，而现在的查询就有问题？

4. 为什么会有上述问题？

   1. 针对上述第一个问题，这是一个亿级大表，所以my_index这个索引也很大
      - MySQL会判断根据二级索引查询的时候是否会进行回表查询，如select * 会进行回表
      - order by id desc limit xx，xx 这个操作，MySQL会判断使用my_index(category , sub_category)这个索引是否会有很多操作，然后会基于临时表进行join操作
      - 而使用id的聚簇索引，且进行limit操作的情况下，只需要扫描聚簇索引，扫描到满足条件的足够数据即进行返回，故MySQL选择了聚簇索引
   2. 针对第三个问题
      - 一开始根据where条件，是能很快找到满足条件的返回值的数据的
      - 后来商品系统增加了商品系统的子项目，但该子类下并无真实的商品数据存在。
      - 故where category = 'xxx' and sub_category = 'xx' 并不存在，会按照聚簇索引进行全表扫描

# 案例二

1. ## 背景

   - 商品评论系统的SQL优化

   - 做了分库分表，单表数据量在百万

   - 每一个商品的评论都在同一张表中

   - 故产生了几十万评论的深度分页问题

     select * from comments where product_id = 'xx' and is_goods = '1' limit 100000,10

   - 有my_index(prpduct_id) 这个索引

2. ### 问题

   - 由于my_index(product_id)，故需要回表去查数据，并进行判断
   - 如果有几十万条数据，则需要根据my_index然后进行几十万次回表查询
   - 优化思路是跟案例一反过来，让这条SQL走聚簇索引进行判断，而不走索引

3. ### 优化

   - 因为做了分库，每张表的数据也才百万级

   - 走二级索引，在由于is_good条件未建立索引，下次还会进行几十万数据的回表判断，所以慢

   - SQL优化为：

     select * from commonts a , (select id from commonts where product_id = 'xx' and is_good = '1' order by id desc limit 10000,10) b where a.id = b.id

   - 上述SQL主动让MySQL走根据聚簇索引的扫描，执行查询

# 案例三

1. ### 背景

   1. 当时有人在代码中，开启了一个事务删除了千万级的数据，导致了频繁的慢查询
   2. 慢查询不一定是由SQL引起的，也可以是MySQL服务器的资源占满了，如cpu、网络、磁盘等超负载导致的。
   3. 上述硬件资源都正常

2. ### 排查

   - 使用MySQL profiling工具去查看执行和耗时

   - 要使用，得先进行MySQL设置，set profilling=1 ，接着MySQL会自动记录查询语句的Profilling信息

   - 如若此时执行show profiles，就会列出各种查询语句的Profilling信息

   - 针对单个查询语句，看到Profilling具体信息

     show profile cpu ， block io for query id

3. ### 问题

   - sending data耗时高
   - show engine Innodb status 看一下存储引擎状态，即history list lenga 这个指标异常高，和undo log日志版本链相关。故这个事务运行时很长，多版本快照不能被purge清理掉
   - 长事务在运行删除的时候，实际上是加了一个删除标识，实际上并没有删除操作。其他事务对其进行查询的时候，可能把这个上千万数据扫描一遍