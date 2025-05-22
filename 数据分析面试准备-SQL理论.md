# SQL理论

## （一）、 sql如何进行优化

sql优化看运⾏环境，可以分为mysql和Hive，前者是数据库查询优化，后者基于MapReduce。互联⽹分析师更多是基于Hive查询数据，所以下⽂针对Hive如何优化进⾏分析。

 (1). 理解数据仓库的分层和数据粒度是⾸要的。

​		因为相⽐于与数据库是为了数据的储存，更新⽽设计的，数据仓库则是更多为了数据的查询。针对具体的业务需求，选择合适的数据粒度，是sql优化的基础。例如选择⽤户粒度的Hive表，比起访问pv粒度的Hive表，数据量要⼩很多，sql查询也更快。 

(2) 针对典型的问题，例如数据倾斜。 

+ 产⽣原因 

  + group by维度过小,某值的数量过多(后果:处理某值的reduce⾮常耗时) 

  + 去重 distinct count(distinct xx) 某特殊值过多(后果：处理此特殊值的reduce耗时) 

  + 3.连接 join,count(distinct),group by,join等操作，这些都会触发Shuffle动作，⼀旦触发，所有相同key的值就会拉到⼀个或⼏个节点上，就容易发⽣单点问题。

+  (2)解决方案 

  + 业务逻辑:例如我们从业务上就知道在做group by时某些key对应数据量很⼤,我们可以单独对这些key做计算,再与其他key进行join 
  + Hive参数设置: 设置hive.map.aggr = true 在map中会做部分聚集操作，效率更高但需要更多的内存设置hive.groupby.skewindata=true 数据倾斜时负载均衡，当选项设定为true，⽣成的查询计划会有两个MRJob。第⼀个MRJob中，Map的输出结果集合会随机分布到Reduce中，每个Reduce做部分聚合操作，并输出结果，这样处理的结果是相同的GroupBy Key有可能被分发到不同的Reduce中，从⽽达到负载均衡的⽬的；第⼆个MRJob再根据预处理的数据结果按照GroupBy Key分布到Reduce中（这个过程可以保证相同的GroupBy Key被分布到同⼀个Reduce中），最后完成最终的聚合操作。 
  + 查询语句优化: 
    + a. 在count(distinct) 操作前先进⾏⼀次group by,把key先进⾏⼀次reduce,去重 
    + b. map join:使⽤map join让⼩的维度表（1000 条以下的记录条数）先进内存,在map端完成reduce.

## （二） sql优化——count(1)、count(*)与count(列名)的区别

一、从执行效果来看
1. count(1) and count(*)：

​		基本没差别，都是求表的总行数

​		count(*)包括了所有的列，相当于求记录总行数，在统计结果的时候，不会忽略NULL


2. count(1) and count(列名)：

​	（1） count(1) 会统计表中的所有的记录数，不会忽略NULL，包含字段为null 的记录。

​	（2） count(列名) 会统计该列字段在表中出现的次数，会忽略字段为null 的情况，即不统计字段为null 的记录。 

二、从执行效率来看

+ 若列名为主键，count(列名)会比count(1)快  
+ 若列名不为主键，count(1)会比count(列名)快  
+ 若表多个列并且没有主键，则 count（1） 的执行效率优于 count（*）  
+ 若表有主键，则 select count（主键）的执行效率是最优的  
+ 若表只有一个字段，则 select count（*）最优。

## （三）窗口函数

+ 基本结构

  ```sql
  [你的操作] over(
  				partition by 用于分组的列名
  				order by 按序叠加的列名
  				rows / range 窗口滑动的数据范围
  				)
  ```

+ 窗口滑动的数据范围

  ```SQL
  # 当前行 - current row
  # 之前的行 - preceding
  # 之后的行 - following 
  # 无界限 - unbounded
  # 表示从前面的起点 - unbounded preceding
  # 表示到后面的终点 - unbounded following
  
  #举例
  # 取当前行和前五行
  rows between 5 preceding and current row
  # 取当前行和后五行
  rows between current row and 5 following
  # 取前五行和后五行
  rows between 5 preceding 5 following 
  ```


## （四） with as 用法

WITH AS短语，也叫做子查询部分，定义一个SQL片断后，该SQL片断可以被整个SQL语句所用到。有的时候，with as是为了提高SQL语句的可读性，减少嵌套冗余。

比如sql：

```sql
with A as (
  select  * 
  from user
) 
select * 
from A, customer 
where 
  customer.userid = user.id**
```

这个sql语句的意思是：先执行select * from user把结果放到一个临时表A中，作为全局使用。

with as的用法可以通俗点讲是，讲需要频繁执行的slq片段加个别名放到全局中，后面直接调用就可以，这样减少调用次数，优化执行效率。

使用with as有如下好处

+ 可以轻松构建一个临时表，通过对这个表数据进行再处理。但是他比临时表更强大，临时表在会话结束才会自动被P清除，但with as临时表查询完成后就被清除了
+ 复杂的查询会产生很大的sql，with as语法可以把一些公共查询提出来，也可以作为一个中间结果，可以使整个sql语句显得有条理些，提高可读性

## （五）with recursive递归用法

1. 递归得到依次递增的序列:

   ```sql
   with recursive cte (n) as
   (
   	select 1
       union all 
       select n + 1 from cte where n < 5   
   )
   select * from cte;
   
   \\运行结果
   +------+
   | n    |
   +------+
   |    1 |
   |    2 |
   |    3 |
   |    4 |
   |    5 |
   +------+
   ```

2. 递归得到不断复制的字符串

   ```sql 
   with recursive cte as
   (
       select 1 as n, 'abc' as str
       union all
       select n + 1, concat(str, str) from cte where n < 3
   )
   select * from cte;
   
   \\运行结果
   +------+------+
   | n    | str  |
   +------+------+
   |    1 | abc  |
   |    2 | abc  |
   |    3 | abc  |
   +------+------+
   
   \\不限制字符串长度
   with recursive cte as:
   (
       select 1 as n, cast('abc', CHAR(20)) as str
       union all
       select n + 1, concat(str, str) from cte where n < 3
   )
   select * from cte;
   
   \\运行结果
   +------+--------------+
   | n    | str          |
   +------+--------------+
   |    1 | abc          |
   |    2 | abcabc       |
   |    3 | abcabcabcabc |
   +------+--------------+
   
   ```

3.  生成斐波那契数列

```sql
with recursive f(n, f_n, next_f_n) as
(
    select 1, 0, 1
    union all 
    select n + 1, next_f_n, f_n + next_f_n from f where n < 10
)
select * from f;
WITH RECURSIVE fibonacci (n, fib_n, next_fib_n) AS

\\运行结果
+------+-------+------------+
| n    | fib_n | next_fib_n |
+------+-------+------------+
|    1 |     0 |          1 |
|    2 |     1 |          1 |
|    3 |     1 |          2 |
|    4 |     2 |          3 |
|    5 |     3 |          5 |
|    6 |     5 |          8 |
|    7 |     8 |         13 |
|    8 |    13 |         21 |
|    9 |    21 |         34 |
|   10 |    34 |         55 |
+------+-------+------------+


```

