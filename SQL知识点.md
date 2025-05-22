# SQL知识点

## 常用命令

- 登录服务器：

  ```
  mysql -uroot -p123456
  ```

  

- 退出mysql：

  ```
  exit
  ```

  

- 查看有哪些数据库：

  ```
  - show databases;
  ```

  使用数据库A：

  ```
  use A;
  ```

  

- 创建数据库A：

  ```
  create database A;
  ```

  

- 查看A数据库下有哪些表：

  ```
  use A；show tables;
  ```

  

- 查看数据库版本：

  ```
  select version();
  ```

  

- 查看当前使用的哪个数据库：

  ```
  select database();
  ```

  

- 终止一条命令的输入：

  ```
  \c
  ```

  

- 导入数据（需先use）：

  ```
  source 路径(不含中文)
  ```

  

- 字符串用单引号

- 所有标识符全部用小写，单词和单词之间用下划线连接

## 查询语句DQL

- 查看数据库版本：

  ```
  select version();
  ```

  

- 查看当前使用的哪个数据库：

  ```
  select database();
  ```

  

- 查看表中所有数据：

  ```
  select * from 表名;
  ```

  

- 查看表结构：

  ```
  desc dept; 
  ```

  

- 简单查询

  - 一个字段：

    ```
    select 字段名 from 表名;
    ```

    

  - 两个字段：

    ```
    select 字段名,字段名 from 表名;
    ```

    

  - 字段可以使用数学表达式：

    ```
    select 字段名*12 from 表名;
    ```

    

- 条件查询

  - 语法格式：

    ```
    select 字段名 from 表名 where 条件;
    ```

    

  - 精确条件：等于=，不等于!=/<>，大于>，大于等于>=，小于<，小于等于<=，闭区间between and ，为空is null，不为空is not null，并且and，或者or（and优先级高于or），包含in，

  - 起别名：

    ```
    select 字段名 as 别名 from 表名;
    select 字段名 as '别名' from 表名;（别名中有空格或为中文）
    ```

    

  - 模糊查询：like，%匹配任意多个字符，_匹配任意一个字符（查询字符串中有_时：'%\_%'）

- 排序

  - 语法格式：

    ```
    select 字段名 from 表名 (where 条件) order by 字段名;（默认升序）
    ```

    

  - 指定顺序：指定降序（order by 字段名 desc;）指定升序（order by 字段名 asc;）

  - 按两个字段排序：

    ```
    select 字段1,字段2 from 表名 (where 条件) order by 字段1,字段2;（先按字段1排序，字段1相同时再按字段2排序）
    ```

    

  - 语句执行顺序：from > where > select > order by 

- 单行处理函数

  - 转换小写：

    ```
    lower(字段名)
    ```

    

  - 转换大写：

    ```sql
    upper(字段名)
    ```

    

  - 取子串：

    ```sql
    substr(字段名/字符串,起始下标，截取长度)，起始下标从开始
    ```

    

  - 字符串拼接：

    ```sql
    concat(字符串1,字符串2)
    ```

    

  - 字符串长度：

    ```sql
    length(字符串)
    ```

    

  -  首字母大写：

    ```sql
    select concat(upper(substr(字段名,1,1)),substr(字段名,2,length(字段名)-1)) as 新字段名 from 表名;
    ```

    

  - 字符串去空格：

    ```sql
    trim(字符串)
    ```

    

  - 条件语句：

    ```sql
    case 要匹配字段名 when 条件值1 then 执行1 when 条件值2 then 执行2 else 执行3
    ```

    

  - 四舍五入：

    ```sql
    round(数值,n),n=0个位，n=1小数点后一位，n=-1十位
    ```

    

  - 数字格式化：

    ```sql
    format(数字,'格式')
    示例：薪水转化为千分位并显示
    select ename,format(sal,'$999,999') as sal from emp;
    ```

    

  - 生成随机数：

    ```sql
    rand()
    100以内随机数：
    select round(rand()*100,0) as random from 表名;
    ```

    

  - 将null转化为具体值：

    ```sql
    ifnull(字段名/数据,具体值)
    如果字段名/数据为空，返回具体值；如果不为空，返回原值
    ```

    

- 分组处理函数

  - 最大/小 值：

    ```sql
    select max/min(字段名) from 表名;
    ```

    

  - 求和/平均：

    ```sql
    select sum/avg(字段名) from 表名;
    ```

    

  - 计数

    - 统计某个字段下不为NULL的总数：

      ```sql
      select count(字段名) from 表名;
      ```

      

    - 统计表的行数：

      ```sql
      select count(*) from 表名;
      select count(1) from 表名;
      ```

      

  - 注意

    - 分组函数自动忽略NULL

    - 分组函数不能直接使用在where子句中

    - 所有分组函数可组合在一起使用

- 分组查询

  - 语法格式：

    ```sql
    select ...  from ... where ... group by ... having ... order by ...;
    ```

    

  - 执行顺序：from(获取)>where(一次过滤)>group by(分组)>having(二次过滤)>select(查询)>order by(排序)

  - 注意

    - select后只能跟分组字段和分组函数

    - 两个字段联合分组：group by 字段1,字段2

    - having：必须和group by 联合使用；优先选择where

- 去重

  - 一个字段去重：

    ```sql
    select distinct 字段1 from 表名;
    ```

    

  - 两个字段联合去重：

    ```sql
    select distinct 字段1 字段2 from 表名;
    ```

    

- 连接查询

  - 笛卡尔现象：

    ```sql
    select 字段1,字段2 from 表1  join 表2 ;
    ```

    

  - 内连接

    - 等值连接：

      ```sql
      select 字段1,字段2 from 表1 inner join 表2 on 等值条件;
      ```

      

    - 不等值连接：

      ```sql
      select 字段1,字段2 from 表1 inner join 表2 on 不等值条件;
      ```

      

    - 自连接：

      ```sql
      select 别名1.字段1,别名2.字段2 from 表1 别名1 inner join 表2 别名2  on 条件;
      ```

      

  - 外连接

    - 左外连接：

      ```sql
      select 字段1,字段2 from 表1 left join 表2 on 连接条件;
      左表为主表
      ```

      

    - 右外连接：

      ```sql
      select 字段1,字段2 from 表1 right  join 表2 on 连接条件;
      右表为主表
      ```

      

    - 三表连接：

      ```sql
      select 字段1,字段2,字段3 from 表1 right/left join 表2 on 连接条件1 right/left join 表3 on 连接条件2;
      ```

      

  - 子查询

    - where子句中的子查询（案例）：

      ```sql
      找出比最低工资高的员工姓名和工资
      select ename,sal from emp where sal > (select min(sal) from emp);
      ```

      

    - from子句中的子查询（案例）:

      ```sql
      找出每个岗位的平均工资的薪资等级
      select t.job,t.avgsal,s.grade 
      from (select job,avg(sal) as avgsal from emp group by job) t 
      inner join salgrade s on t.avgsal between s.losal and s.hisal; 
      ```

    - select子句中的子查询（案例）：

      ```sql
      找出每个员工的部门名称
      select e.ename, (select d.dname from dept d where e.deptno = d.deptno ) as dname from emp e;
      ```

      

  - 合并查询结果集

    - 案列：

      ```sql
      查询工作岗位是MANAGER和SALESMAN的员工
      select ename,job from emp where job = 'MANAGER' union select ename,job from emp where job = 'SALESMAN';
      ```

      

    - 注意：合并时要求两个结果集列数相同，数据类型可以不同

  - limit

    - 用法：limit 起始下标，长度或limit 长度

    - 语法：

      ```sql
      select ... from ... where ... group by ... having ... order by ... limit ...;
      ```

      

    - 执行顺序：1.from  (join on)2.where 3.group by 4.having 5.select 6.order by 7.limit

- 四大排名函数

  - 连续不重复：

    ```sql
    row_number() over(order by 字段名)12345
    ```

    

  - 不连续重复：

    ```sql
    rank() over(order by 字段名)12245
    ```

    

  - 连续重复：

    ```sql
    dense_rank() over(order by 字段名 )12234
    ```

    

  - 分组排序：

    ```sql
    ntile(分组数) over(order by字段名)
    ```

    

# 数据定义语句DDL

- 表的创建

  - 语法格式：

    ```sql
    create table 表名(字段1 数据类型(长度) default '默认值',字段2 数据类型,字段3 数据类型);
    ```

    

  - 常用数据类型

    - varchar：可变长度字符串（最长255）

    - char：定长字符串（最长255）

    - int：整数（最长11）

    - bigint：长整型

    - float：单精度浮点型

    - double：双精度浮点型

    - date：短日期类型（只包含年月日，默认格式%Y-%m-%d）

    - datetime：长日期类型（包含年月日时分秒，默认格式%Y-%m-%d %h:%i:%s）

    - clob：字符大对象（存储长度超过255的字符）

    - blob：二进制大对象（存储图片、声音、视频等流媒体对象）

  - 快速建表：

    ```sql
    create table 表名 as select 字段 from 表名;
    ```

    

- 快速删除表中数据

  - 语法格式：

    ```sql
    truncate table 表名;
    ```

    

  - 注意：速度快，不支持回滚

- 约束

  - 非空约束
    - 语法格式（只有列级约束）： 

      ```sql
      create table 表名(字段1 数据类型 not null,字段2 数据类型);
      ```
    
      
    
  - 唯一性约束

    - 语法格式

      - 列级约束： 

        ```sql
        create table 表名(字段1 数据类型 unique,字段2 数据类型);
        ```

        

      - 表级约束：

        ```sql
         create table 表名(字段1 数据类型,字段2 数据类型,unique(字段1,字段2));
        ```

        

    - not null 和unique 联合：

      ```sql
       create table 表名(字段1 数据类型 not null unique,字段2 数据类型);
       该字段自动变为主键字段
      ```

      

  - 主键约束

    - 相关术语：主键约束，主键字段，主键值

    - 特征：not null + unique

    - 作用：主键值是每一行记录的唯一标识（身份证号）；没有主键，表无效

    - 主键值建议类型：int，bigint，char

    - 分类：单一主键和复合主键，自然主键（推荐）和业务主键

    - 语法格式

      - 列级约束： 

        ```sql
        create table 表名(字段1 数据类型 primary key,字段2 数据类型);
        只能添加一个
        ```
      
        
      
      - 表级约束：
      
        ```sql
        单一主键 
        create table 表名(字段1 数据类型,字段2 数据类型,primary key(字段1));
        复合主键 
        create table 表名(字段1 数据类型,字段2 数据类型,primary key(字段1,字段2));
        ```
      
        
      
      - 自动维护： 
      
        ```sql
        create table 表名(字段1 数据类型 primary key auto_increment,字段2 数据类型);
        ```
      
        

  - 外键约束

    - 相关术语：外键约束，外键字段，外键值

    - 作用：节省空间，避免数据冗余、无效

    - 表的分类：被引用的为父表（先创建，先插入数据，后删除），外键所在表为子表（后创建，后插入数据，先删除数据）

    - 语法格式:

      ```sql
      create table 父表(被引用字段 数据类型 unique/primary key,字段 数据类型); 
      create table 子表(外键字段 数据类型 unique/primary key,字段 数据类型,foreign key(外键字段) references 父表(被引用字段));
      ```
    
      
    
    - 注意：父表中被引用字段不一定为主键字段，但必须具有unique约束;外键值可以为NULL

- 存储引擎

  - 含义：存储引擎是一个表存储/组织数据的方式

  - 添加/指定引擎：

    ```sql
    create table 表名()engine = 引擎名 default charset = utf8;
    ```
  
    
  
  - 查看MySQL支持的引擎：
  
    ```sql
    show engines \G
    ```
  
    
  
  - 常用引擎：MyISAM，InnoDB，MEMORY

# 数据操纵语句DML

- 插入数据

  - 语法格式

    - ```sql
      insert into 表名(字段1,字段2,字段3) values(字段1值,字段2值,字段3值);
      ```

      

    - ```sql
      insert into 表名 values(字段1值,字段2值,字段3值);
      不加字段名时默认所有字段
      ```

      

    - ```sql
      insert into 表名 values(),(),();
      一次插入多条数据
      ```

      

    - ```sql
      insert into 表名 select 字段 from 表名;
      将查询结果插入，
      ```

      

  - 注意

    - 字段名和值要一一对应（数量，数据类型）

    - insert语句成功必然添加一条记录，没给其他字段指定值的话，会自动添加默认值（一般为NULL）

  - 插入日期数据

    - 日期格式：%Y，%m，%s，%h，%i，%s

    - 将字符串varchar类型转化为date类型：

      ```sql
      str_to_date('字符串日期','日期格式')
      ```

      
    
    - 将date类型数据转化为一定格式的varchar类型：
    
      ```sql
      date_format(日期数据字段名,'日期格式')
      ```
    
      
    
    - 获取当前系统时间:
    
      ```sql
      now()
      ```
    
      
    
    - 计算两个日期字段的差：
    
      ```sql
      timestampdiff(间隔类型, 前一个日期字段(小日期), 后一个日期字段)
      日期间隔（SECOND   秒，MINUTE   分钟，HOUR   小时，DAY   天，WEEK   星期，MONTH   月，QUARTER   季度，YEAR   年）
      ```
    
      

- 修改数据

  - 类型1

    - 语法格式：

      ```sql
      update 表名 set 字段1 = 值1,字段2 = 值2,字段3 = 值3 where 条件;
      ```

      

    - 注意：不加where条件则整张表被修改

  - 类型2
    - 语法格式：
    
      ```sql
      update 表名 set 字段 = if(条件,条件成立时取值,条件不成立时取值);
      ```
    
      

- 删除数据

  - 语法格式：

    ```sql
    delete from 表名 where 条件;
    ```
  
    
  
  - 注意：不加where条件则整张表被删除；速度慢，支持回滚

# 常用

- 事务

  - 定义：一个完整的业务逻辑，是一个最小的工作单元，本质上一个事务是多条DML语句同时成功或同时失败

  - 语法

    - 提交事务：

      ```sql
      commit;
      ```

      

    - 回滚事务：

      ```sql
      rollback;
      ```

      

    - 关闭自动提交机制：

      ```sql
      start transaction;
      ```

      

  - 特性：原子性，一致性，隔离性，持久性

  - 事务隔离级别

    - 读未提交：read uncommitted，存在脏读

    - 读已提交：read committed，存在不可重复读取数据

    - 可重复读：repeatable read，存在幻影读

    - 序列化：serializable，解决所有问题，但效率最低

- 索引

  - 定义

    - 在数据库表的字段上添加，是为了提高查询效率的一种机制；

    - 在MySQL中索引是一种B-Tree数据结构，遵循左小右大原则存放，采用中序遍历方式遍历数据

  - 实现原理

    - 任何数据库中主键字段会自动添加索引，在MySQL中一个字段如果有unique约束，该字段会自动添加索引

    - 任何一张表的任何一条记录在硬盘存储上都都有一个硬盘的物理存储编号

    - 在MyISAM存储引擎中索引储存在.MYI文件中；InnoDB存储引擎存储在tablespace中；MEMORY存储引擎存储在内存中

  - 适合添加索引的字段：数据量庞大；该字段经常出现在where后面；该字段很少进行DML操作

  - 语法

    - 创建索引：

      ```sql
      create index 索引名 on 表名(字段名);
      ```

      

    - 删除索引：

      ```sql
      drop index 索引名 on 表名;
      ```

      

    - 查看某一SQL是否使用索引：

      ```sql
      explain select ... ;
      ```

      

  - 索引失效

    - 模糊查询以“%”开始会失效：

      ```sql
      select * from emp where ename like '%T';  （若ename 已添加索引）
      ```

      

    - 使用or时两边的条件有一个字段无索引：
    
      ```sql
      select  * from emp where ename = 'KING' or job = 'MANAGER';
      若ename字段有索引 job无索引，则该语句未使用索引查询
      ```
    
      
    
    - 使用复合索引时，没有使用左侧的字段查找：
    
      ```sql
      create index emp_job_sal_index on emp(job,sal); 
      select * from emp where sal = 800;
      ```
    
      
    
    - 在where中添加运算：
    
      ```sql
      select * from emp where sal +1 = 801;（若sal已添加索引）
      ```
    
      
    
    - 在where中使用函数：
    
      ```sql
      select * from emp where low(ename) = 'smith';
      ```
    
      

- 视图

  - 含义：站在不同角度去看待同一份数据

  - 特点：通过对视图的操作，会影响到原表数据

  - 作用：方便，简化开发，利于维护

  - 语法

    - 创建视图对象：

      ```sql
      create view 视图名 as select ... ;
      ```
    
      
    
    - 增删改查（与表相同）

- DBA常用命令

  - 数据导出：

    ```sql
    （在dos命令窗口中）
    mysqldump bjpowernode emp > D:\bjpowernode.sql -uroot -p123456
    ```

    

  - 数据导入：

    ```sql
    （登录MySQL服务器）
    create database bjpowernode; 
    use bjpowernode;
    source D:\bjpowernode.sql;
    ```

    

- 数据库设计三范式

  - 第一范式：要求任何一张表必须有主键，每一个字段原子性不可再分

  - 第二范式：建立在第一范式基础之上，要求所有非主键字段完全依赖主键，不产生部分依赖

  - 第三范式建立在第二范式基础之上，要求所有非主键字段直接依赖主键，不产生传递依赖

  - 注意：三范式是理论上的，为了满足客户需求，有时会拿冗余换执行速度（减少表的连接次数，降低代码编写难度）

- 建表口诀

  - 多对多，三张表，关系表两个外键

  - 一对多，两张表，多的表加外键

  - 一对一，外键唯一