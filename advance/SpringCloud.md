

## 微服务

将传统的单体服务，根据业务拆分成一个个的服务，可以拥有独立的数据库

## 和Dubbo区别



## Eureka-Server

## 配置中心

### 1、Config-Server



### 2、Config-Client



## Hystrix





## Zuul

### 1、路由配置

### 2、过滤器

实现：只需继承ZuulFilter并实现抽象函数即可

简单介绍一下函数：

- filterType：过滤器的类型。默认有四种
  - pre：在请求路由之前调用
  - routing：在路由请求时调用
  - post：在routing和error过滤器之后调用
  - error：处理请求发生错误时调用

- filterOrder：定义过滤器执行顺序，数值越小优先级越高
- shoulFilter：判断该过滤器是否要执行，可以通过此方法指定过滤器的有效范围
- run：

```java
package zcs.apigateway;

import com.netflix.zuul.ZuulFilter;
import com.netflix.zuul.context.RequestContext;
import com.netflix.zuul.exception.ZuulException;

import javax.servlet.http.HttpServletRequest;

/**
 * 过滤器
 * 需要在应用主类为其创建Bean
 */
public class AccessFilter extends ZuulFilter {

    /**
     * 过滤器的类型
     * 决定过滤器在请求的哪个生命周期执行
     * @return pre代表在请求路由之前执行
     */
    @Override
    public String filterType() {
        return "pre";
    }

    /**
     * 执行顺序
     * 存在多个过滤器时，根据该值决定执行顺序
     * @return
     */
    @Override
    public int filterOrder() {
        return 0;
    }

    /**
     * 判断该过滤器是否需要被执行
     * @return
     */
    @Override
    public boolean shouldFilter() {
        return true;
    }

    /**
     * 具体逻辑
     * @return
     * @throws ZuulException
     */
    @Override
    public Object run() throws ZuulException {
        RequestContext ctx = RequestContext.getCurrentContext();
        HttpServletRequest request = ctx.getRequest();
        String accessToken = request.getParameter("name");
        if (accessToken == null) {
//            过滤该请求
            ctx.setSendZuulResponse(false);
            ctx.setResponseStatusCode(401);
            ctx.addZuulResponseHeader("content-type","text/html;charset=utf-8");
            ctx.setResponseBody("无参数访问");
        }
        return null;
    }
}

```

主类

```java
@EnableZuulProxy
@SpringBootApplication
public class ApiGatewayApplication {
    @Bean
    public AccessFilter accessFilter(){
        return new AccessFilter();
    }

    public static void main(String[] args) {
        SpringApplication.run(ApiGatewayApplication.class, args);
    }

}
```

可以通过过滤器定义一些与业务无关的通用逻辑从而实现对请求的过滤和拦截，比如：签名校验、权限校验、请求限流等。