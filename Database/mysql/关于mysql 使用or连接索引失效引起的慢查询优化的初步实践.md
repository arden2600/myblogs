## 关于mysql 使用or连接索引失效引起的慢查询优化的初步实践:<br>
最近在使用mysql开发时候，遇到了稍微多的数据的时候，sql查询中order by语法的查询效率明显的降低了好几个数量级，所以进行了一些sql语句调整尝试或者说是简单的优化尝试。该篇文章仅供参考。<br>
数据库部署在阿里云服务上，mysql版本为5.6，下面sql查询的还仅仅是前15条记录/页。查询需求是: 在商户表(t_merchants)和店铺表(t_shops)中，根据所给的查询条件，查询店铺表，将根据得到的店铺表记录在查询店铺关联的商户表信息记录查询出来。其中，也`包括未添加店铺(也就是说商户表中店铺数量字段为0)的商户`记录，并且根据创建时间排序。商户商户表与店铺表是`一对多`关系。<br>
初始sql查询语句是:<br>
```sql
SELECT DISTINCT
       m.i_id   MERCHANTID,
       m.vc_metchants_name MERCHANTNAME,
       m.vc_address DETAILADDR,
       m.i_shop_num   SHOPNUM
FROM 
      t_merchantsd m, t_shop s
WHERE (
       m.i_id = s.i_merchants_id
    OR m.i_shop_num = 0 )
AND m.i_id IS NOT NULL
AND s.i_state != 4
AND m.i_state != 3
ORDER BY
  m.vc_create_time DESC LIMIT 1,15
```  
<br>
在mysql查询工具中，通过使用explain查询执行过程，几千条数据，执行查询居然花费了7s左右。（硬件和远程数据库也许也有些影响，但是也不会那么慢）:<br>
![image](https://github.com/arden2600/myblogs/blob/master/Database/mysql/images/original_mysql_search_explain_2016-12-26.png) <br>
<br>
从上图的执行计划看出，对商户表和店铺表都进行了全表扫描。其中在数据库设计中，商户表的`i_state,vc_create_time`和店铺表的`i_state,i_merchants_id`都是索引列。但是执行计划中，对sql语句中AND后面的条件查询，都没有使用到索引，也就是说索引失效了。那么问题在哪的?那应该只有WHERE语句后面的条件`(m.i_id = s.i_merchants_id or  OR m.i_shop_num = 0)`。其中id相关的列都是索引相关的，只有OR条件后的m.i_shop_num不是索引列，会不会是这种写法导致了用不上索引，导致查询结果慢了很多。<br>

---
所以，尝试了将`OR m.i_shop_num=0`条件去掉，进行了查询。结果平均查询花费时间也就是30毫秒左右而已，可见不通过索引查询会花费很多时间，也就证明了，上面的sql语句因为加上了**m.i_shop_num=0**导致了索引失效了。那么问题来了，怎么可以使得在保持查询需求情况下，修改sql使得查询走索引呢?<br>
<br>
在不断尝试过程中，发现可以使用子查询来解决问题：不能同时关联查询两个表，最后的查询结果是商户表记录信息。那么，可以先查询满足条件的商户id集合，在根据商户id集合，查询商户表中商户id在这id集合中的商户记录即可。sql如下:<br>
```sql
SELECT 
    m.i_id              MERCHANTID,
    m.vc_merchants_name MERCHANTNAME,
    m.vc_address        DETAILADDR,
    m.i_shop_num        SHOPNUM
FROM 
    t_merchants m 
WHERE m.id in (
    SELECT DISTINCT m1.i_id FROM t_merchants m1, t_shop s
                            WHERE (m1.i_id = s.i_merchants_id OR m1.i_shop_num = 0)
                            AND s.i_state != 4
                            AND m1.i_state != 3
                            AND m1.i_id IS NOT NULL)
ORDER BY m.vc_create_time DESC LIMIT 1,15          
```
<br>
查询仅仅花费了60ms左右，比刚开始的6/7s而言，快了太多了。使用explain查询执行计划信息如下图:<br>
![image](https://github.com/arden2600/myblogs/blob/master/Database/mysql/images/op_mysql_search_explain_2016-12-26.png) <br>
在商户表和店铺表之间，内部嵌套使用一个eq_ref类型扫描查询。大部分性能的提升跟他就有关系了。两次查询计划仅仅就是使用了eq_ref类型查询。那么eq_ref是什么东东呢。关于eq_ref的介绍，这里有篇文章说得挺好：[http://www.cnblogs.com/heat-man/p/4945708.html](http://www.cnblogs.com/heat-man/p/4945708.html) <br>
<br>
sql就是博大精深，所以在写sql的时候，有空闲或者有精力的情况下，可以使用explain来看看执行计划，说不定可以发现可以优化的空间，何乐不为赛~
