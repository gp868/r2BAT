# MySQL

# 索引

- 分类
  - 按「数据结构」分类：B+tree索引、Hash索引、Full-text索引
  - 按「物理存储」分类：主键索引、二级索引
  - 按「字段特性」分类：主键索引、唯一索引、普通索引、前缀索引
  - 按「字段个数」分类：单列索引、联合索引


- 主键索引的 B+Tree 和二级索引的 B+Tree 区别如下：

  - 主键索引的 B+Tree 的叶子节点存放的是实际数据，所有完整的用户记录都存放在主键索引的 B+Tree 的叶子节点里；
  - 二级索引的 B+Tree 的叶子节点存放的是主键值，而不是实际数据。


- 联合索引的最左匹配原则

  联合索引的最左匹配原则，在遇到范围查询（如 >、<）的时候，就会停止匹配，也就是范围查询的字段可以用到联合索引，但是在范围查询字段的后面的字段无法用到联合索引。注意，对于 >=、<=、BETWEEN、like 前缀匹配的范围查询，并不会停止匹配。

  例子：

  `select * from t_table where a > 1 and b = 2`， a 字段用到了联合索引进行索引查询，而 b 字段并没有使用到联合索引；

  `select * from t_table where a >= 1 and b = 2`，a 和 b 字段都用到了联合索引进行索引查询；

  `SELECT * FROM t_table WHERE a BETWEEN 2 AND 8 AND b = 2`，a 和 b 字段都用到了联合索引进行索引查询；

  `SELECT * FROM t_user WHERE name like 'j%' and age = 22`， a 和 b 字段都用到了联合索引进行索引查询；

- 什么时候适用索引？
  - 字段有唯一性限制的，比如商品编码；
  - 经常用于 `WHERE` 查询条件的字段，这样能够提高整个表的查询速度，如果查询条件不是一个字段，可以建立联合索引。
  - 经常用于 `GROUP BY` 和 `ORDER BY` 的字段，这样在查询的时候就不需要再去做一次排序了，因为我们都已经知道了建立索引之后在 B+Tree 中的记录都是排序好的。

- 什么时候不需要创建索引？
    - WHERE条件， GROUP BY， ORDER BY 里用不到的字段；
    - 字段中存在大量重复数据，不需要创建索引；
    - 表数据太少的时候，不需要创建索引；
    - 经常更新的字段不用创建索引；

- 如何优化索引

  - 前缀索引优化

    前缀索引顾名思义就是使用某个字段中字符串的前几个字符建立索引，使用前缀索引是为了减小索引字段大小，可以增加一个索引页中存储的索引值，有效提高索引的查询速度。在一些大字符串的字段作为索引时，使用前缀索引可以帮助我们减小索引项的大小。

  - 覆盖索引优化

    覆盖索引是指 SQL 中 query 的所有字段，在索引 B+Tree 的叶子节点上都能找得到的那些索引，从二级索引中查询得到记录，而不需要通过聚簇索引查询获得，可以避免回表的操作。

  - 主键索引最好是自增的

    如果我们使用自增主键，那么每次插入的新数据就会按顺序添加到当前索引节点的位置，不需要移动已有的数据，当页面写满，就会自动开辟一个新页面。因为每次插入一条新记录，都是追加操作，不需要重新移动数据，因此这种插入数据的方法效率非常高。

    如果我们使用非自增主键，由于每次插入主键的索引值都是随机的，因此每次插入新的数据时，就可能会插入到现有数据页中间的某个位置，这将不得不移动其它数据来满足新数据的插入，甚至需要从一个页面复制数据到另外一个页面，我们通常将这种情况称为页分裂。页分裂还有可能会造成大量的内存碎片，导致索引结构不紧凑，从而影响查询效率。

  - 防止索引失效

    - 使用左或者左右模糊匹配，也就是 `like %xx` 或者 `like %xx%`这两种方式都会造成索引失效；
    - 查询条件中对索引列做了计算、函数、类型转换操作，这些情况下都会造成索引失效；
    - 联合索引要能正确使用需要遵循最左匹配原则，也就是按照最左优先的方式进行索引的匹配，否则就会导致索引失效。
    - 在 WHERE 子句中，如果在 OR 前的条件列是索引列，而在 OR 后的条件列不是索引列，那么索引会失效。

- MySQL 默认的存储引擎 InnoDB 采用的是 B+ 作为索引的数据结构，原因有：

  - B+ 树的非叶子节点不存放实际的记录数据，仅存放索引，因此数据量相同的情况下，相比存储即存索引又存记录的 B 树，B+树的非叶子节点可以存放更多的索引，因此 B+ 树可以比 B 树更「矮胖」，查询底层节点的磁盘 I/O次数会更少。

  - B+ 树有大量的冗余节点（所有非叶子节点都是冗余索引），这些冗余索引让 B+ 树在插入、删除的效率都更高，比如删除根节点的时候，不会像 B 树那样会发生复杂的树的变化；

  - B+ 树叶子节点之间用链表连接了起来，有利于范围查询，而 B 树要实现范围查询，因此只能通过树的遍历来完成范围查询，这会涉及多个节点的磁盘 I/O 操作，范围查询效率不如 B+ 树。

# 事务

- 事务的ACID特性：

  - 原子性（Atomicity）

    一个事务中的所有操作，要么全部完成，要么全部不完成，不会结束在中间某个环节，而且事务在执行过程中发生错误，会被回滚到事务开始前的状态，就像这个事务从来没有执行过一样；

  - 一致性（Consistency）

    数据库总是从一个一致性的状态转移到另一个一致性的状态。一致性确保了即使在执行第三、第四条语句之间时系统崩溃，前面执行的第一、第二条语句也不会生效，因为事务最终没有提交，所有事务中所作的修改也不会保存到数据库中。

  - 隔离性（Isolation）

    一个事务的执行不能其它事务干扰，一个事务内部的操作及使用的数据对其它并发事务是隔离的，并发执行的各个事务之间不能互相干扰。

  - 持久性（Durability）

    事务处理结束后，对数据的修改就是永久的，即便系统故障也不会丢失。


- InnoDB 引擎通过什么技术来保证事务的这四个特性的呢？

  - 原子性是通过 undo log（回滚日志） 来保证的；

  - 一致性则是通过持久性+原子性+隔离性来保证；

  - 隔离性是通过 MVCC（多版本并发控制） 或锁机制来保证的；
  - 持久性是通过 redo log （重做日志）来保证的；

- 并发事务引发的问题

  在同时处理多个事务的时候，就可能出现**脏读、不可重复读、幻读**的问题。

  - 脏读

    读到其他事务未提交的数据；

  - 不可重复读

    前后读取的数据不一致；

  - 幻读

    前后读取的记录数量不一致。

- 事务的隔离级别

  三种现象的严重性排序：脏读 > 不可重复读 > 幻读

  四种隔离级别：

  - 读未提交

    一个事务还没提交时，它做的变更就能被其他事务看到；

  - 读已提交

    一个事务提交之后，它做的变更才能被其他事务看到；

  - 可重复读

    一个事务执行过程中看到的数据，一直跟这个事务启动时看到的数据是一致的，MySQL InnoDB 引擎的默认隔离级别；

  - 串行化

    会对记录加上读写锁，在多个事务对这条记录进行读写操作时，如果发生了读写冲突的时候，后访问的事务必须等前一个事务执行完成，才能继续执行；

  隔离水平高低排序：串行化  > 可重复读 > 读已提交 > 读未提交，隔离级别越高，性能效率就越低。

  - 在「读未提交」隔离级别下，可能发生脏读、不可重复读和幻读现象；
  - 在「读已提交」隔离级别下，可能发生不可重复读和幻读现象，但是不可能发生脏读现象；
  - 在「可重复读」隔离级别下，可能发生幻读现象，但是不可能脏读和不可重复读现象；
  - 在「串行化」隔离级别下，脏读、不可重复读和幻读现象都不可能会发生。

  所以，要解决脏读现象，就要升级到「读提交」以上的隔离级别；要解决不可重复读现象，就要升级到「可重复读」的隔离级别，要解决幻读现象不建议将隔离级别升级到「串行化」。

  MySQL InnoDB 引擎的默认隔离级别虽然是「可重复读」，但是它很大程度上避免幻读现象，解决的方案有两种：

  - 针对快照读（普通 select 语句），是通过 MVCC 方式解决了幻读，因为可重复读隔离级别下，事务执行过程中看到的数据，一直跟这个事务启动时看到的数据是一致的，即使中途有其他事务插入了一条数据，是查询不出来这条数据的，所以就很好了避免幻读问题；
  - 针对当前读（select ... for update 等语句），是通过 next-key lock（记录锁+间隙锁）方式解决了幻读，因为当执行 select ... for update 语句的时候，会加上 next-key lock，如果有其他事务在 next-key lock 锁范围内插入了一条记录，那么这个插入语句就会被阻塞，无法成功插入，所以就很好了避免幻读问题。

  对于「读提交」和「可重复读」隔离级别的事务来说，它们是通过 Read View 来实现的，它们的区别在于创建 Read View 的时机不同：

  - 「读提交」隔离级别是在每个 select 都会生成一个新的 Read View，也意味着，事务期间的多次读取同一条数据，前后两次读的数据可能会出现不一致，因为可能这期间另外一个事务修改了该记录，并提交了事务。
  - 「可重复读」隔离级别是启动事务时生成一个 Read View，然后整个事务期间都在用这个 Read View，这样就保证了在事务期间读到的数据都是事务启动前的记录。

  这两个隔离级别实现是通过「事务的 Read View 里的字段」和「记录中的两个隐藏列」的比对，来控制并发事务访问同一个记录时的行为，这就叫 MVCC（多版本并发控制）。

# 语法

- 各关键字的使用顺序

  ```
  select--from--where--group by--having--order by--limit
  ```

- SELECT... FROM ...

- 去重：SELECT DISTINCT... FROM ...

- 顺序：SELECT ... FROM  ... ORDER BY ... ; 

- 倒序：SELECT ... FROM ... ORDER BY ... DESC; (descending order)

- 多变量排序

  ```mysql
  SELECT
      cust_id,
      order_num
  FROM
      Orders
  ORDER BY
      cust_id,
      order_date DESC;
  ```

- 范围筛选

  ```mysql
  SELECT 
      prod_name,
      prod_price
  FROM
      Products
  WHERE
      prod_price >= 3 AND prod_price <= 6
  ORDER BY
      prod_price;
  ```

  ```mysql
  SELECT 
      prod_name,
      prod_price
  FROM
      Products
  WHERE
      prod_price BETWEEN 3 AND 6
  ORDER BY
      prod_price;
  ```

  ```mysql
  SELECT
      order_num,
      prod_id,
      quantity
  FROM
      OrderItems 
  WHERE
      quantity >= 100
      AND
      prod_id in ('BR01', 'BR02', 'BR03');
  ```

- 筛选记录

  `LIMIT`关键字：

  ```mysql
  SELECT ... FROM ... LIMIT [offset,] rows
  ```

  `LIMIT`关键字⽤于强制 SELECT 语句返回指定的记录数。LIMIT 接受⼀个或两个数字参数，参数必须是⼀个整数常量。

  - 如果给定两个参数，第⼀个参数指定起始记录⾏的**偏移量**，第⼆个参数指定末尾记录行；

    ```mysql
    SELECT * FROM table LIMIT 10,15; // 检索记录⾏11-15
    ```

  - 如果只给定⼀个参数，它表示返回记录行的数⽬；

    ```mysql
    SELECT * FROM score ORDER BY grade LIMIT 5
    // 提取出前5条记录
    ```

  - `offset`关键字

    ```mysql
    select * from user limit 3 offset 1;
    // 取第 1 行后面第2, 3, 4行三条数据
    ```

- 模糊查找

  关键词：like

  用法：[字符] like '%_[]'

  - %表示任何字符出现任意次数
  - _表示单个字符
  - []表示一个字符集

  ```mysql
  SELECT
      prod_name, 
      prod_desc
  FROM
      Products
  WHERE
      prod_desc LIKE '%toy%'; // 出现 toy 字段
  ```

  ```mysql
  SELECT
    prod_name,
    prod_desc
  FROM
    Products
  WHERE
    prod_desc NOT LIKE '%toy%' // 未出现 toy 字段
  ORDER BY
    prod_name;
  ```

  ```mysql
  SELECT
      prod_name,
      prod_desc
  FROM
      Products
  WHERE
      prod_desc LIKE '%toy%' AND prod_desc LIKE '%carrotS%' // 同时出现 toy 和 carrots 两个字段
  ```

  ```mysql
  SELECT
      prod_name,
      prod_desc
  FROM
      Products
  WHERE
      prod_desc LIKE '%toy%carrots%'; // 以先后顺序同时出现 toy 和 carrots 字段
  ```

  ```mysql
  SELECT 
      order_num,
      order_date
  FROM
      Orders
  WHERE
      order_date like '2020-01%' // 筛选出一月份的数据
  ORDER BY
      order_date
  ```

- 别名

  ```mysql
  SELECT 
      vend_id,
      vend_name vname,
      vend_address vaddress,
      vend_city vcity
  FROM
      Vendors
  ORDER BY
      vend_name
  ```

  ```mysql
  SELECT 
      prod_id,
      prod_price,
      prod_price * 0.9 sale_price
      round(prod_price * 0.9, 1) sale_price // 四舍五入保留一位小数
  FROM
      Products;
  ```

- 字符操作

  关键词：substring, concat, upper

  用法：

  - 字符串的截取：substring(字符串，起始位置，截取字符数）
  - 字符串的拼接：concat(字符串1，字符串2，字符串3,...)
  - 字母大写：upper(字符串）

  ```mysql
  SELECT
      cust_id,
      cust_name,
      upper(concat(substring(cust_name,1,2), substring(cust_city,1,3))) as user_login
  FROM
      Customers
  ```


- 聚合函数

  | 函数名   | COUNT | SUM  | AVG      | MAX    | MIN    |
  | -------- | ----- | ---- | -------- | ------ | ------ |
  | **作用** | 计数  | 求和 | 求平均值 | 最大值 | 最小值 |

  - MAX(column)：返回某列的最大值
  - MIN(column)：返回某列的最高值 
  - COUNT(column)：返回某列的总行数 
  - COUNT(*)：返回表的总行数
  - SUM(column)：返回某列的相加总和
  - AVG(column)：返回某列的平均值

- 聚合函数只作用非null，因为null数据不参与运算。

  ```mysql
  select avg(comm), avg(ifnull(comm, 0)) from emp;
  ```

  两个计算结果是不一样的，ifnull函数的作用就是发现值为null后将其值变为0。聚合函数使用时注意空值的情况，要配合`ifnull`函数使用。

- 分组 GROUP BY

  GROUP BY 语句根据一个或多个列对结果集进行分组。在分组的列上通常配合 COUNT, SUM, AVG等函数一起使用。

  例如：求每个部门所有工资总和。通过deptno字段对表数据进行分组后，然后通过sum(sal)来计算每个分组的总和。

  ```mysql
  SELECT
  	deptno, SUM(sal)
  FROM 
  	emp
  GROUP BY
  	deptno;
  ```

  例如：查询每个部门工资大于1500的的人数。

  ```mysql
  SELECT
  	deptno, COUNT(*)
  FROM 
  	emp
  WHERE
  	sal > 1500
  GROUP BY
  	deptno;
  ```

- HAVING

  HAVING用于**分组后**的再次筛选，只能用于分组。

  例如：求工资总和大于9000的部门，并按照工资总和排序。

  ```mysql
  SELECT
  	deptno, SUM(sal)
  FROM 
  	emp
  GROUP BY
  	deptno
  HAVING
  	SUM(sal) > 9000
  ORDER BY
  	SUM(sal);
  ```

  - **having和where区别：**

  1. having是分组后，where是分组前；
  2. where不用使用聚合函数，having可以使用聚合函数；
  3. where在分组之前就会进行筛选，过滤掉的数据不会进入分组；



