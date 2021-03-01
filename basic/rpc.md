---
title: 基于netty和zookeeper实现的rpc
tags: rpc
---

参考：https://my.oschina.net/huangyong/blog/361751

https://www.w3cschool.cn/zookeeper/zookeeper_api.html

RPC，即 Remote Procedure Call（远程过程调用），调用远程计算机上的服务，就像调用本地服务一样。RPC可以很好的解耦系统。

整体框架

![image-20200320201649193](rpc/image-20200320201649193.png)

**使用的技术：**

服务的发布与订阅：使用Zookeeper作为注册中心

通信：Netty框架

Spring：配置服务，加载Bean，扫描注解

动态代理

消息编解码：使用Protostuff序列化和反序列化消息

## 一、编写服务接口

```java
public interface HelloService {
    String hello(String msg);
}
```

## 二、编写实现类

### 定义注解

```java
//说明修饰的对象范围
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Component
public @interface RpcService {
    Class<?> value();
}
```

这里简单说一下注解的使用。注解通过 @interface 关键字进行定义，并且通过元注解进行注解。元注解有个@Retention、@Documented、@Target、@Inherited、@Repeatable 5 种

https://blog.csdn.net/qq1404510094/article/details/80577555

**@Retention** ：定义注解的保留策略

RetentionPolicy.SOURCE：注解仅存在于源码中，在编译时会被丢弃

RetentionPolicy.CLASS：默认，在class字节码中存在，不会被加载到JVM

RetentionPolicy.RUNTIME：一直存在

**@Target**：定义注解的作用目标

ElementType.TYPE：接口、类、枚举、注解

ElementType.METHOD：描述方法

**@Documented**：将注解中的元素包含到 Javadoc 中

**@Inherited**：如果一个超类被 @Inherited 注解过的注解进行注解的话，那么如果它的子类没有被任何注解应用的话，那么这个子类就继承了超类的注解

```java
@Inherited
@Retention(RetentionPolicy.RUNTIME)
@interface Test {}
-----------------------------------
@Test
public class A {}
public class B extends A {}
```

类 B 也拥有 Test 这个注解

**@Repeatable**：注解的值可以同时取多个。一个人他既是程序员又是产品经理,同时他还是个画家

```java
@interface Persons {
    Person[]  value();
}
@Repeatable(Persons.class)
@interface Person{
    String role default "";
}
@Person(role="artist")
@Person(role="coder")
@Person(role="PM")
public class SuperMan{
}
```

### 实现类

```java
@RpcService(HelloService.class)
public class HelloServiceImpl implements HelloService {
    @Override
    public String hello(String name) {
        return "hello" + name;
    }
}
```

## 三、编写服务端

![image-20200320201706144](rpc/image-20200320201706144.png)

### ApplicationContextAware

实现这个接口（只有一个方法），Spring会把上下文对象传入进来，就可以获得ApplicationContext中的所有bean

```java
	@Override
    public void setApplicationContext(ApplicationContext ctx) throws BeansException {
//        扫描RpcService注解生成的bean  (RpcService注解实现了@Component，因此生成了bean)
        Map<String, Object> serviceBeanMap = ctx.getBeansWithAnnotation(RpcService.class);
        if (!serviceBeanMap.isEmpty()) {
            for (Object serviceBean : serviceBeanMap.values()) {
//                获取类的接口的类全名
                String name = serviceBean.getClass().getAnnotation(RpcService.class).value().getName();
                handlerMap.put(name, serviceBean);
            }
        }
```

### InitializingBean

接口为bean提供了初始化方法的方式，它只包括afterPropertiesSet方法，凡是继承该接口的类，在初始化bean的时候都会执行该方法。

在这个方法中创建服务器，并注册到zookeeper上

```java
	@Override
    public void afterPropertiesSet() {
        EventLoopGroup bossGroup = new NioEventLoopGroup();
        EventLoopGroup workerGroup = new NioEventLoopGroup();

        try {
            ServerBootstrap serverBootstrap = new ServerBootstrap();
            serverBootstrap.group(bossGroup, workerGroup)
                    .channel(NioServerSocketChannel.class)
                    .option(ChannelOption.SO_BACKLOG, 128)
                    .childOption(ChannelOption.SO_KEEPALIVE, true)
                    .childHandler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel socketChannel) throws Exception {
                            socketChannel.pipeline()
                                    .addLast(new RpcDecoder(RpcRequest.class))
                                    .addLast(new RpcEncoder(RpcResponse.class))
                                    .addLast(new RpcServerHandler(handlerMap));
                        }
                    });

//            解析地址
            String[] addressArray = serviceAddress.split(":");
            String host = addressArray[0];
            int port = Integer.parseInt(addressArray[1]);
            ChannelFuture channelFuture = serverBootstrap.bind(host, port).sync();
            System.out.println("服务启动");
//            注册到zookeeper
            for (String interfaceName : handlerMap.keySet()) {
                serviceRegistry.register(interfaceName, serviceAddress);
                System.out.println(interfaceName + "注册服务 => " + serviceAddress);
            }

            channelFuture.channel().closeFuture().sync();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }
    }
```

### rpc.properties

```xml
#服务器地址
rpc.service_address=127.0.0.1:7000
#zookeeper地址
rpc.server_address=192.168.186.128:2181
```

### server-spring.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="dubborpc.provider"/>
    <context:property-placeholder location="classpath:rpc.properties"/>

    <bean id="serviceRegistry" class="dubborpc.registry.ServiceRegistry">
        <constructor-arg name="registryAddress" value="${rpc.server_address}"/>
    </bean>

    <bean id="rpcServer" class="dubborpc.server.RpcServer">
        <constructor-arg name="serviceAddress" value="${rpc.service_address}"/>
        <constructor-arg name="serviceRegistry" ref="serviceRegistry"/>
    </bean>
</beans>
```

## 四、服务注册

使用zookeeper轻松实现

用到的常量

```java
public interface ZkConstant {
    int ZK_SESSION_TIMEOUT = 5000;
//   随意写 必须以/开头
    String ZK_REGISTRY_PATH="/services";
}
```

注册类是为了将服务器中的bean和ip注册到zookeeper中，类中唯一的方法如下

```java
public void register(String serviceName, String serviceAddress) {
        try {
            ZooKeeper zooKeeper = new ZooKeeper(serverAddress, ZkConstant.ZK_SESSION_TIMEOUT, new Watcher() {
                @Override
                public void process(WatchedEvent watchedEvent) {
                    if (watchedEvent.getState() == Event.KeeperState.SyncConnected) {
                        latch.countDown();
                    }
                }
            });
//            等待连接成功
            latch.await();
//            创建注册根节点
            Stat exists = zooKeeper.exists(ZkConstant.ZK_REGISTRY_PATH, false);
            if (exists == null) {
                zooKeeper.create(ZkConstant.ZK_REGISTRY_PATH, new byte[0], ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
            }

//            创建bean永久节点
            String servicePath = ZkConstant.ZK_REGISTRY_PATH + "/" + serviceName;
            if (zooKeeper.exists(servicePath, false) == null) {
                zooKeeper.create(servicePath, new byte[0], ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
            }

//            创建临时节点存储ip 如果服务器掉线zookeeper可以感知
            zooKeeper.create(servicePath + "/ip", serviceAddress.getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL);

        } catch (Exception e) {
            e.printStackTrace();
        }
    }
```

## 五、启动服务器并发布

```java
public class ServerBootstrap {
    public static void main(String[] args) {
        new ClassPathXmlApplicationContext("server-spring.xml");
    }
}
```

```xml
[zk: localhost:2181(CONNECTED) 21] get /services/dubborpc.api.HelloService/ip
127.0.0.1:7000
```

## 六、实现服务发现

根据类名去Zookeeper中寻找服务

```java
public class ServiceDiscovery {

    private String serverAddress;

    private ZooKeeper zookeeper;

    private volatile List<String> dataList = new ArrayList<>();

    private CountDownLatch latch = new CountDownLatch(1);

    public ServiceDiscovery(String serverAddress) {
        this.serverAddress = serverAddress;
        zookeeper = connect();
    }

    public String discover(String name) {
        try {
            String parentPath = ZkConstant.ZK_REGISTRY_PATH + "/" + name;
            List<String> children = zookeeper.getChildren(parentPath, true);
            List<String> tmpList = new ArrayList<>();
            for (String node : children) {
                byte[] data = zookeeper.getData(parentPath + "/" + node, false, null);
                tmpList.add(new String(data));
            }
            dataList=tmpList;
        } catch (Exception e) {
            e.printStackTrace();
        }
        int size = dataList.size();
        if (size == 0) {
            throw new RuntimeException("没发现服务");
        }else if (size == 1) {
            return dataList.get(0);
        } else {
            return dataList.get(ThreadLocalRandom.current().nextInt(size));
        }
    }

    private ZooKeeper connect() {
        ZooKeeper zooKeeper = null;
        try {
            zooKeeper = new ZooKeeper(serverAddress, ZkConstant.ZK_SESSION_TIMEOUT, watchedEvent -> {
                if (watchedEvent.getState() == Watcher.Event.KeeperState.SyncConnected) {
                    latch.countDown();
                }
            });
            latch.await();
        } catch (IOException | InterruptedException e) {
            e.printStackTrace();
        }
        return zooKeeper;
    }
}
```

## 七、实现客户端

![image-20200320201720368](rpc/image-20200320201720368.png)

```java
public class RpcClient extends SimpleChannelInboundHandler<RpcResponse> {

    private RpcResponse response;

    @Override
    public void channelRead0(ChannelHandlerContext ctx, RpcResponse response) throws Exception {
        this.response=response;
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        ctx.close();
    }

    public RpcResponse send(RpcRequest request,String serviceAddress) throws Exception {
        NioEventLoopGroup group = new NioEventLoopGroup();
        try {
            Bootstrap bootstrap = new Bootstrap();
            bootstrap.group(group)
                    .channel(NioSocketChannel.class)
                    .option(ChannelOption.TCP_NODELAY, true)
                    .handler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel socketChannel) throws Exception {
                            socketChannel.pipeline()
                                    .addLast(new RpcEncoder(RpcRequest.class))
                                    .addLast(new RpcDecoder(RpcResponse.class))
                                    .addLast(RpcClient.this);
                        }
                    });

            //            解析地址
            String[] addressArray = serviceAddress.split(":");
            String host = addressArray[0];
            int port = Integer.parseInt(addressArray[1]);
            ChannelFuture channelFuture = bootstrap.connect(host, port).sync();

            Channel channel = channelFuture.channel();
            channel.writeAndFlush(request).sync();
            channel.closeFuture().sync();
            return response;
        } finally {
            group.shutdownGracefully();
        }
    }
}
```

## 八、实现代理

使用动态代理实现

```java
public class RpcProxy {

    private String serviceAddress;

    private ServiceDiscovery serviceDiscovery;

    public RpcProxy(String serviceAddress) {
        this.serviceAddress = serviceAddress;
    }

    public RpcProxy(ServiceDiscovery serviceDiscovery) {
        this.serviceDiscovery = serviceDiscovery;
    }

    @SuppressWarnings("unchecked")
    public <T> T create(final Class<?> interfaceClass) {
        return  (T) Proxy.newProxyInstance(interfaceClass.getClassLoader(),
                new Class<?>[]{interfaceClass},
                (proxy, method, args) -> {
                    RpcRequest request = new RpcRequest();
                    request.setRequestId(UUID.randomUUID().toString());
                    request.setInterfaceName(method.getDeclaringClass().getName());
                    request.setMethodName(method.getName());
                    request.setParameterTypes(method.getParameterTypes());
                    request.setParameters(args);
                    request.setServiceVersion("1.0");

                    if (serviceDiscovery != null) {
                        serviceAddress = serviceDiscovery.discover(interfaceClass.getName());
                        System.out.println("发现服务" + interfaceClass.getName() + "ip：" + serviceAddress);
                    }
                    RpcClient client = new RpcClient();
                    RpcResponse response = client.send(request, serviceAddress);
                    if (response==null) {
                        throw new RuntimeException("response is null");
                    }
                    return response.getResult();
                }
        );
    }
}
```

client-spring.xml

```xml
 <context:component-scan base-package="dubborpc.provider"/>
    <context:property-placeholder location="classpath:rpc.properties"/>

    <bean id="serviceDiscovery" class="dubborpc.registry.ServiceDiscovery">
        <constructor-arg name="serverAddress" value="${rpc.server_address}"/>
    </bean>

    <bean id="rpcProxy" class="dubborpc.client.proxy.RpcProxy">
        <constructor-arg name="serviceDiscovery" ref="serviceDiscovery"/>
```

## 九、测试

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = "classpath:client-spring.xml")
public class HelloRpc {
    @Autowired
    private RpcProxy rpcProxy;

    @Test
    public  void test() {
        HelloService helloService = rpcProxy.create(HelloService.class);
        System.out.println(helloService.hello("world"));
    }
}
```

