---
title: nginx
tags: nginx
---

## 负载均衡

### 四层负载均衡

基于ip和端口选择服务器，修改请求报文中的目标ip，然后直接发给服务器

LVS、F5(硬件负载均衡器)

### 七层负载均衡

内容交换，基于应用层信息选择服务器

nginx、apache、haproxy

## 什么是nginx

nginx是一个高性能的HTTP服务器，常用作动静分离、反向代理和负载均衡

单线程，非阻塞，高并发连接

## 为什么性能高

### 多进程单线程模型

包括一个master进程和多个worker进程。

master负责接收外部请求、管理worker

worker进程的个数一般设置为CPU核心数，过多的话会导致竞争，带来不必要的上下文切换

### 网络事件模块

采用异步非阻塞事件处理机制

## 什么是反向代理

既然有反向代理，那么肯定就有正向代理。那么什么是正向代理呢？

### 正向代理

如果我们要访问google，正常情况下肯定时访问不了的，因为被外网被墙了

<img src="nginx/image-20191101135528584.png" alt="image-20191101135528584" style="zoom:80%;" />

为了能从google获得内容，我们需要通过在电脑上配置VPN，通过向VPN发送一个请求，然后代理服务器转交请求从google中取得内容并返回到我们的电脑上，这个过程google不知道我们的存在，只知道是VPN在请求，这就是正向代理。

代理端代理的是客户端，和服务端没关系

### 反向代理

假如现在没有墙了，我们的电脑可以直接访问google

google不可能只有一台服务器吧，那么我们怎么知道去访问那一台呢？这个时候反向代理就出来了，我们只需要访问反向代理服务器，而反向代理服务器会自动选择一个服务器，并将获得的结果返回给我们。

代理端代理的是服务端，和客户端没关系

<img src="nginx/image-20191101141157973.png" alt="image-20191101141157973" style="zoom:80%;" />

来一个总图理解一下

<img src="nginx/image-20191101141645466.png" alt="image-20191101141645466" style="zoom: 80%;" />

### 反向代理的优点

可以隐藏服务器的存在和特征

## 负载均衡算法

1. 轮询（默认）
2. 加权轮询：见下
3. ip_hash：每个请求按访问ip的hash结果分配，这样每个访客固定访问一个后端服务器，可以解决session的问题
4. fair（第三方）：根据页面大小和加载时间来分配请求

加权轮询算法：

每个服务器有三个权重变量：

- weight是配置文件中的；
- currentWeight是服务器目前的权重，初始值为0
- effectiveWeight是有效权重，初始值为weight，如果发现节点异常则-1，之后再次选取本节点，调用成功一次则+1，直到weight

算法逻辑：

- 请求到来时，currentWeight+=effectiveWeight，选出最大的一个节点
- 将所有的currentWeight相加得到totalWeight，选中的节点currentWeight-=totalWeight

## 惊群现象

master进程绑定端口监听socket事件，当事件发生时，worker进程被同时唤醒，但是只有一个会成功，其他会失败，浪费资源

处理：使用互斥锁，worker进程执行accept()之前先获取锁，accept()成功之后再解锁

## 限流

### limit_req

使用ngx_http_limit_req_module模块，采用漏桶算法，可以限制单个IP的请求频率

- binary_remote_addr：是$remote_addr（客户端IP，占7-15个字节）的二进制格式，占4个字节
- zone=one:10m：缓存区名称为one，占用10M
- rate=1r/s：针对同一客户端IP，平均处理的请求频率为1秒1次
- burst=2：超过访问频次限制的请求先放到缓冲区，如果没设置，同一时刻同一IP来了两个请求，那么有一个会直接返回503
- nodelay：超过频率限制而且缓冲区也满，那么会返回503。比如同一时刻同一IP来了三个请求，nginx会立即处理，下两秒又进来一个直接返回503。如果没有这个参数，会严格执行1秒1次

```bash
limit_req_zone $binary_remote_addr zone=one:10m rate=1r/s;
 
server {
    location /search/ {
        limit_req zone=one burst=2 nodelay;
	}
}
```

### limit_conn

限制连接数

- zone=addr:10m：定义名为addr的空间，分配内存10m，如果耗尽，后续请求都返回503
- limit_conn addr 1：同一个ip和服务器连接超过1个，会被返回503

```bash
limit_conn_zone $binary_remote_addr zone=addr:10m;
server {
    location /api/ {
        limit_conn addr 1;
        limit_conn_status 503; 
    }
}
```

如果连接不释放，通过这个连接发送再多的request都可以，但是如果要再建立连接就不行

## 配置

```nginx
	upstream m_pool {
    //fail_timeout服务器无响应后隔30s再去连接服务器
        server 127.0.0.1:12001 max_fails=1 fail_timeout=30s;
        server 127.0.0.1:12002 max_fails=1 fail_timeout=30s;
        server 127.0.0.1:12003 max_fails=1 fail_timeout=30s;
    }

    server {
        listen       80;
        server_name  m.xxx.com;
        location / {
            proxy_pass http://m_pool;
            proxy_set_header Host $host:$server_port;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Scheme $scheme;
        	//1s内服务器无响应立即跳转到其他服务器
            proxy_connect_timeout 1;
            proxy_read_timeout 20;
            proxy_send_timeout 20;
            access_log off;
            break;
        }
```



## 常用命令

start nginx/nginx.exe	启动

tasklist /fi "imagename eq nginx.exe"	查看nginx进程 会出现两个 一个是主进程 一个是工作进程

taskkill /fi "imagename eq nginx.EXE" /f  杀进程

nginx -s stop	快速关闭

nginx -s quit	平稳关闭

nginx -s reload	重载配置

