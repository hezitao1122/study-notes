# 案例一

1. ## 背景
   1. 电商平台亿级数据量商品系统的SQL优化

   2. 由于存在多个索引,MySQL选择了一个不太适合的索引,导致了慢查询

   3. 语句

      select * from products where category = 'xxx' and sub_category = 'xx' order by id desc limit 1000,10

   4. 有一个索引 my_index(category , sub_category)

2. ## 排查

   1. 使用explain关键字观察，并未使用我们所指定的索引my_index，而是使用的index方式，即走id聚簇索引

   2. extra里包含了using where

   3. 使用语句强制其使用索引

      select * from products force my_index(category , sub_category) where category = 'xxx' and sub_category = 'xx' order by id limit 1000,10