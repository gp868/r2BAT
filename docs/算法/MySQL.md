# MySQL

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

  







