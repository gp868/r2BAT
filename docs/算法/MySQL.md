# MySQL

- SELECT... FROM ...

- 去重：select distinct ... from ...

- 顺序：select ... from ... order by ... ; 

- 倒序：select ... from ... order by ... desc; (descending order)

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

- 筛选前几名

  ```mysql
  // 查询score表里的grade列，使用grade列进行排序，并且提取出前5条记录
  select * from score order by grade limit 5
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
      order_date like '2020-01%' // shai
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
      round(prod_price * 0.9, 1) sale_price
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

  











