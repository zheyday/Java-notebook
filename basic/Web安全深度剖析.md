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

### 反射型

XSS代码出现在URL中，作为输入提交到服务器处理，然后把带有XSS代码的数据发送给浏览器，浏览器解析后造成XSS漏洞。

### 存储型

攻击者提交XSS代码会存储到服务器，下次有用户请求时，服务器会查询数据并显示，在浏览器上解析执行

### DOM型

修改页面的DOM节点

```html
<div id="t">
</div>
<input tpye="text" id="text" value=""/>
<input tpye="button" value="write" onclick="test()"/>
<script>
	function test(){
        var str=document.getElementById("text").value;
        document.getElementById("t").innerHTML="<a href=' "+str+"' >testLink</a>";
    }
</script>

如果输入 
' onclick=alert(/xss/)//
代码就变成了
<a href='' onclick=alert(/xss/)//' >testLink</a>
或者输入
'><img src=# onerror=alert(/xss2/) /><'
代码就变成了
<a href=''><img src=# onerror=alert(/xss2/) /><'' >testLink</a>
```

![1615434995893](Web%E5%AE%89%E5%85%A8%E6%B7%B1%E5%BA%A6%E5%89%96%E6%9E%90/1615434995893.png)

## XSS构建



# CSRF

Cross Site Request Forgery，跨站点请求伪造

## 防御

### 验证码

CSRF攻击通常是在用户不知道的情况下构造了网络请求，使用验证码强制用户与应用进行交互，但是用户体验差

### Refer Check









