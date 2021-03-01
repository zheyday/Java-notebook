---
title: SpringBoot整合ElasticSearch7.2.0
date: 2019-09-26 20:30:47
tags: ElasticSearch
---

## <font color="red">先说一下踩的坑</font>

新建一个SpringBoot项目，版本 2.1.8.RELEASE

起初准备引用`spring-boot-starter-data-elasticsearch`依赖，但是报错。

客户端报错

```
NoNodeAvailableException[None of the configured nodes are available: [{#transport#-1}{4iYNiV-ERCqSsQU75n9TJw}{127.0.0.1}{127.0.0.1:9300}]
]
```

服务端提示

```
java.lang.IllegalStateException: Received message from unsupported version: [6.4.3] minimal compatible version is: [6.8.0]  
```

也就是客户端版本太低，SpingBoot2的这个依赖只支持ES2.x。而Spring-data-elasticsearch也支持不到7.x。
下图是版本对照：

{% asset_img 1569501826203.png%}

## 步入正题

封装好的不能用，只能选择官方推荐的client了。



