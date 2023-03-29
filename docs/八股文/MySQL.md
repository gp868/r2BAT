# MySQL

- 各关键字的使用顺序

  ```sql
  select--from--where--group by--having--order by--limit
  ```

- SELECT... FROM ...

- 去重：SELECT DISTINCT... FROM ...

- 顺序：SELECT ... FROM  ... ORDER BY ... （ASC省略）; 

- 倒序：SELECT ... FROM ... ORDER BY ... DESC; (descending order)

- 多变量排序

  ```sql
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

  ```sql
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

  ```sql
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

  ```sql
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

  ```sql
  SELECT ... FROM ... LIMIT [offset,] rows
  ```

  `LIMIT`关键字⽤于强制 SELECT 语句返回指定的记录数。LIMIT 接受⼀个或两个数字参数，参数必须是⼀个整数常量。

  - 如果给定两个参数，第⼀个参数指定起始记录⾏的**偏移量**，第⼆个参数指定记录的**行数**；

    ```sql
    SELECT * FROM table LIMIT 10, 5; // 从第11行开始记录5行数据
    SELECT * FROM table LIMIT 10, 1; // 显示第11行
    ```

  - 如果只给定⼀个参数，它表示返回从第一行开始记录行的数⽬；

    ```sql
    SELECT * FROM score ORDER BY grade LIMIT 5
    // 显示成绩排名前5的j
    ```

  - `offset`关键字，后面的参数是记录行的**偏移量**；

    ```sql
    select * from user limit 3 offset 1;
    // 取第2, 3, 4行三条数据
    select * from user limit 1 offset 4;
    // 取第 5 行数据
    ```

- 模糊查找

  关键词：like

  用法：[字符] like '%_[]'

  - %表示任何字符出现任意次数
  - _表示单个字符
  - []表示一个字符集

  ```sql
  SELECT
      prod_name, 
      prod_desc
  FROM
      Products
  WHERE
      prod_desc LIKE '%toy%'; // 出现 toy 字段
  ```

  ```sql
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

  ```sql
  SELECT
      prod_name,
      prod_desc
  FROM
      Products
  WHERE
      prod_desc LIKE '%toy%' AND prod_desc LIKE '%carrotS%' // 同时出现 toy 和 carrots 两个字段
  ```

  ```sql
  SELECT
      prod_name,
      prod_desc
  FROM
      Products
  WHERE
      prod_desc LIKE '%toy%carrots%'; // 以先后顺序同时出现 toy 和 carrots 字段
  ```

  ```sql
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

  ```sql
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

  ```sql
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

  ```sql
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

  ```sql
  select avg(comm), avg(ifnull(comm, 0)) from emp;
  ```

  两个计算结果是不一样的，ifnull函数的作用就是发现值为null后将其值变为0。聚合函数使用时注意空值的情况，要配合`ifnull`函数使用。

- 分组 GROUP BY

  GROUP BY 语句根据一个或多个列对结果集进行分组。在分组的列上通常配合 COUNT, SUM, AVG等函数一起使用。

  例如：求每个部门所有工资总和。通过deptno字段对表数据进行分组后，然后通过sum(sal)来计算每个分组的总和。

  ```sql
  SELECT
  	deptno, SUM(sal)
  FROM 
  	emp
  GROUP BY
  	deptno;
  ```

  例如：查询每个部门工资大于1500的的人数。

  ```sql
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

  ```sql
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

  1. having用在分组后，where用在分组前；
  2. where不能使用聚合函数，having可以使用聚合函数；
  3. where在分组之前就会进行筛选，过滤掉的数据不会进入分组；

