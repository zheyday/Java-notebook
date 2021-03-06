![1614739331045](Web%E5%AE%89%E5%85%A8%E6%B7%B1%E5%BA%A6%E5%89%96%E6%9E%90/1614739331045.png)

# SQL注入漏洞

### 获取元数据

1. 查询数据库

   ```sql
   select schema_name from information_schema.schemata
   ```

2. 查询当前数据库的表

   ```sql
   select table_name from information_schema.tables where table_schema= (select database())
   ```

3. 查询指定表的所有字段

   ```sql
   select column_name from information_schema.columns where table_name='student' 
   ```


# XSS

## 类型

### 反射性

XSS代码出现在URL中，作为输入提交到服务器处理，然后把带有XSS代码的数据发送给浏览器，浏览器解析后造成XSS漏洞。

### 存储型

攻击者提交XSS代码会存储到服务器，下次有用户请求时，服务器会查询数据并显示，在浏览器上解析执行

### DOM

















