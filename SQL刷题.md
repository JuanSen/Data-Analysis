# SQL刷题

## 一、连续出现n次的数据

题目：连续出现三次的数据(t.180)https://leetcode.cn/problems/consecutive-numbers/

解法：

+ 如果一个num连续出现时，那么它（出现的真实序列）-（它出现的次数）一定是个定值，使用两次row_number()

+ ```sql
  #出现的真实序列
  row_number() over(order by id字段)
  #出现的次数
  row_number() over(partition by 值字段 order by id 字段)
  ```

## 二、连续多条满足条件的记录

题目：每行的人数大于或等于 100 且 id 连续的三行或更多行记录（t.601）https://leetcode.cn/problems/human-traffic-of-stadium/

解法：

+ 解法一：三表联合查询，注意条件

  ```sql
  (a.id = b.id-1 and b.id = c.id -1) or
  (a.id = b.id-1 and a.id = c.id +1) or
  (a.id = b.id+1 and b.id = c.id +1)
  ```

+ 解法二：窗口函数

  ```sql
  id-row_number()over(order by visit_date) as k1 #判断是否连续
  count(1) over(partition by k1) #统计连续的行数
  ```
  

## 三、累计求和

### 1. 截止某一日期前的累计求和

题目：在此日期之前玩家所玩的游戏总数（t.534）https://leetcode.cn/problems/game-play-analysis-iii/；不同性别每日分数总计(t.1308)

+ 解法一：自连接/两表查询，注意条件和分组

  ```sql
  where 
  	a.id字段 = b.id字段
  and
  	a.日期字段 >= b.日期字段
  group by
  	a.id字段,a.日期字段
  order by
  	a.id字段,a.日期字段
  ```

+ 解法二：窗口函数

  ```sql
  select
  	id字段, 日期字段, sum(求和字段) over(paritition by id字段 order by 日期字段)
  from
  	表
  order by 
  	id字段, 日期字段
  ```

### 2. 某一日期前后区间累计求和

题目：每个月的近三个月的累计薪水（不足三个月也要计算）（t.579）https://leetcode.cn/problems/find-cumulative-salary-of-an-employee/

+ 解法一：自连接两次/两表查询

  ```sql
  #自连接
  left join	
  	b
  on 
  	a.id字段 = b.id字段
  and 
  	a.月份字段 = b.月份字段 - 1
  left join	
  	c
  on 
  	a.id字段 = c.id字段
  and 
  	a.月份字段 = c.月份字段 - 1
  # 两表查询
  where 
  	a.id字段 = b.id字段
  and
  	a.月份字段 >= b.月份字段
  and
  	a.月份字段 < b.月份字段 + 3
  ```

+ 解法二：窗口函数

  ```sql
  sum(求和字段) over (partition by id字段 order by 月份字段 range 2 preceding)
  ```


## 四、求中位数

### 1. 分组求中位数

题目：各公司员工薪水中位数（t.569）https://leetcode.cn/problems/median-employee-salary/

解法：

```SQL
select
    a.id, a.company, a.salary
from
    (
        select 
            row_number() over(partition by company order by salary) as `rank`, 
            count(salary) over(partition by company) as cnt,
            company, 
            salary,
            id
        from 
            Employee
    ) a
where 
    a.`rank` >= a.cnt / 2
and 
    a.`rank` <= a.cnt / 2 + 1


```

### 2. 根据频率求中位数

题目：给定数字的频率查询中位数（t.571）https://leetcode.cn/problems/find-median-given-frequency-of-numbers/

解法：

```SQL
'当某一数字的正序和逆序累计均大于整个序列的数字个数的一半时即为中位数，将最后选定的一个或两个中位数进行求均值即可。'
select 
    avg(num) as median
from
    (
        select 
            num, 
            sum(frequency) over(order by num asc) as asc_amount, 
            sum(frequency) over(order by num desc) as desc_amount,
            sum(frequency) over() as total_num
        from
            Numbers
    ) a
where
    asc_amount >= total_num / 2 
and 
    desc_amount >= total_num / 2 
```



## 五、判断某个数在某一字段中是否唯一

题目：2016年成功投资（t.585）https://leetcode.cn/problems/investments-in-2016/

+ 解法一：分组函数+count()

  ```sql
  #不唯一 count > 1
  where
  	查询字段 in (select 查询字段 from 表 group by 查询字段 having count(*)>1)
  # 两字段唯一
  where
  	concat(LAT, LON) in (select concat(LAT, LON) from insurance group by LAT, LON having count(*) = 1)s
  ```

+ 解法二：窗口函数

  ```sql
  select
  	count(*) over(partition by 查询字段) as k
  where 
  	k > 1
  ```

## 六、求任意两点间距离

题目：两点间最小距离（t.613）https://leetcode.cn/problems/shortest-distance-in-a-line/

+ 解法

```sql
select
    sqrt(min(pow(c.x,2) + pow(c.y, 2))) as shortest
from
    (
        select 
            a.x as x, b.x as y
        from 
            point a, point b
        where
            a.x != b.x
    ) c
```

+ 注意

  min的位置以及避免重复计算

## 七、表格格式转化

### 1. 行转列

题目：（t.1777）（t.1179）

解法：

```SQL
"原表格有基准ID：group by+sum/max/min(case when)"
"t.1777"

SELECT product_id,
       SUM(CASE store WHEN 'store1' THEN price ELSE NULL END) AS 'store1',
       SUM(CASE store WHEN 'store2' THEN price ELSE NULL END) AS 'store2',
       SUM(CASE store WHEN 'store3' THEN price ELSE NULL END) AS 'store3'
FROM products
GROUP BY product_id

"原表格无基准ID:row_number + group by+sum/max/min(case when)"
"t.618"

select 
    max(case continent when 'America' then name else null end) as America,
    max(case continent when 'Asia' then name else null end) as Asia,
    max(case continent when 'Europe' then name else null end) as Europe
    
from 
    (
        select
            *,
            row_number() over(partition by continent order by name) as rk 
        from
            Student 
     ) t 
group by
    rk
;


```

注意：

+ 观察题中是否有基准id,若题中无基准id，需要先使用row_number()进行构造
+ 使用的聚合函数sum/max/min是根据列中的数据格式决定的

## 八、根据已有表创建新表

题目：用户购买平台（t.1127）

## 九、找到连续区间的开始和结束数字

题目：t.1285 https://leetcode.cn/problems/find-the-start-and-end-number-of-continuous-ranges/

思路：

```
一个连续区间内的数 减去 某个规律的数 结果应该是一样的
log_id  |  rank  |  diff
1       |  1     |  0
3       |  2     |  1
4       |  3     |  1
7       |  4     |  3
8       |  5     |  3
```

解法：

```sql
select
	min(log_id) as start_id,
	max(log_id) as end_id
from
	(
    	select
			log_id,
        	log_id - row_number() over() as diff
		from
			Logs
	) temp
group by
	diff;
```

## 十、sum（）判断

题目：t.1322https://leetcode.cn/problems/ads-performance/

```sql
SELECT ad_id AS 'ad_id', 
    ROUND(
         IFNULL(
             SUM(action='Clicked') / SUM(action IN ('Clicked', 'Viewed')) * 100
             , 0)
         , 2) 
        AS 'ctr'
FROM Ads
GROUP BY ad_id
ORDER BY ctr DESC, ad_id ASC 
;


```

