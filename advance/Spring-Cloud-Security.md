---
title: Spring-Cloud-Security
tags:
---

三个关键配置类，最后一个是资源服务器配置

- WebSecurityConfigurerAdapter
- AuthorizationServerConfigurerAdapter
- ResourceServerConfigurerAdapter

### WebSecurityConfigurerAdapter

先从最基础的配置类

FormLoginConfigurer

OAuth2AuthenticationProcessingFilter会使用OAuth2AuthenticationManager来验证token

重写三个configure函数

1. configure(AuthenticationManagerBuilder auth)用来配置用户，可以基于内存和数据库，我们选择从数据库中读取用户信息，`userServiceDetail`

```java
@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
    auth.userDetailsService(userServiceDetail).passwordEncoder(passwordEncoder());
}
```

