

Vue跨域配置如下：

```js
proxyTable: {
      '/api': {
        target: 'http://localhost:4000',
        changeOrigin: true,
        pathRewrite: {
          '^/api': ''
        }
      }
    },
```

然而启动之后报错，意思是代理请求转发失败

```java
[HPM] Error occurred while trying to proxy request /position/40.10039,116.36868 from localhost:8082 to http://localhost:4000 (HPE_INVALID_CONSTANT) (https://nodejs.org/api
/errors.html#errors_common_system_errors)
```

![image-20210420234611482](https://gitee.com/zheyday/blog-picture-bed/raw/master/img/20210420234618.png)

proxyTable的配置肯定是生效了，但是没有配置正确，把localhost改为本机的ip即可。在许多地方用localhost会有点问题，具体原因待查，最好用ip。

```js
proxyTable: {
      '/api': {
        target: 'http://192.168.1.15:4000',
        changeOrigin: true,
        pathRewrite: {
          '^/api': ''
        }
      }
    },
```

