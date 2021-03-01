---
title: SpringCloud Zuul整合Swagger2
date: 2019-09-30 19:13:28
tags:
- SpringCloud
- Restful
---

Swagger可以为Spring MVC中的接口生成文档，但是微服务化后API都散落在各个微服务中，该如何生成API文档呢？既然需要集中生成，那么自然想到通过Zuul来实现这个功能。

## Service模块

对之前建立的eureka-client进行改造。

引入swagger依赖

```xml
	   <dependency>
            <groupId>com.spring4all</groupId>
            <artifactId>swagger-spring-boot-starter</artifactId>
            <version>1.9.0.RELEASE</version>
        </dependency>
```

在controller上添加`@EnableSwagger2Doc`注解，在具体方法上添加相应注解。

```java
    @ApiOperation(value = "获取端口号",notes = ".")
    @ApiImplicitParam(name = "name",value = "用户名",required = true,dataType = "String")
    @GetMapping("/hi")
    public String dc(String name) {
        return "hello" + name + " i am from " + port;
    }
```

`@ApiOperation`：说明方法的作用

`@ApiImplicitParam`：解释参数。当有多个参数时，需要使用`@ApiImplicitParams`({})

```javascript
 @ApiOperation(value = "用户登陆", notes = ".")
    @ApiImplicitParams({
            @ApiImplicitParam(name = "name", value = "用户名", required = true, dataType = "String"),
            @ApiImplicitParam(name = "psd", value = "密码", required = true, dataType = "String")
    })
    @PostMapping("/login")
    public String login(String name, String psd) {
        return "hello" + name + " i am from " + port;
    }

```

application.yml中也有一些配置

```xml
swagger:
  base-package: zcs.user_center.controller
  title: 用户中心接口文档
```

`base-package`表示需要扫描的包，也就是在这个包下的类才会生成API文档。默认是全部

`title`表示该文档的标题

## Zuul模块

新建一个Spring模块，引入jar包。

```xml
	    <dependency>
            <groupId>com.spring4all</groupId>
            <artifactId>swagger-spring-boot-starter</artifactId>
            <version>1.9.0.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
        </dependency>	
```

application.yml配置

```xml
zuul:
  routes:
    eureka-client: /eureka-client/**
    user-center: /user-center/**
swagger:
  title: SpringCloud项目开发文档
  description: 秒杀/12306
  version: 1.0
```

`zuul.routes`表示zuul需要拦截的请求，这里配置两个路径。

Application类中新增

    @ConfigurationProperties("zuul")
    public ZuulProperties zuulProperties() {
        return new ZuulProperties();
    }
表示从yml中获取配置。

新建一个swagger配置类

```java
package zcs.apigateway.config;

import com.spring4all.swagger.EnableSwagger2Doc;
import io.swagger.models.Swagger;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.cloud.netflix.zuul.filters.ZuulProperties;
import org.springframework.context.annotation.Primary;
import org.springframework.stereotype.Component;
import springfox.documentation.swagger.web.SwaggerResource;
import springfox.documentation.swagger.web.SwaggerResourcesProvider;

import java.util.ArrayList;
import java.util.List;
import java.util.Set;

@EnableSwagger2Doc
@Component
@Primary
public class Swagger2 implements SwaggerResourcesProvider {
    @Autowired
    ZuulProperties zuulProperties;

    @Override
    public List<SwaggerResource> get() {
        Set<String> routeList=zuulProperties.getRoutes().keySet();
        List<SwaggerResource> resources = new ArrayList<>();
        for (String s : routeList) {
            resources.add(swaggerResource(s,"/"+s+"/v2/api-docs","2.0"));
        }
        return resources;
    }

    private SwaggerResource swaggerResource(String name,String location,String version){
        SwaggerResource swaggerResource=new SwaggerResource();
        swaggerResource.setName(name);
        swaggerResource.setLocation(location);
        swaggerResource.setSwaggerVersion(version);
        return swaggerResource;
    }
}

```

`routeList`中包含yml中所有的路径。

## 测试

启动两个服务和网关服务，访问http:localhost:9110/swagger-ui.html

