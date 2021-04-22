---
title: Spring框架学习(一)：Ioc
date: 2019-09-24 16:58:23
tags: Spring框架
---

对tiny-spring项目的学习，转载自黄亿华大佬的<<1000行代码读懂Spring（一）- 实现一个基本的IoC容器>>。

原网址：

http://my.oschina.net/flashsword/blog/192551

https://my.oschina.net/flashsword/blog/194481

概念的讲解转自 https://blog.csdn.net/Tritoy/article/details/81010595

# IOC

## 什么是IOC？

来自知乎：https://www.zhihu.com/question/23277575/answer/24259844

Inversion of Control，即控制反转，是一种设计思想。

传统程序设计中，创建对象的主动权是由自己把控的；现在由Spring的IoC容器根据配置文件来创建及注入依赖对象，而不是由对象主动去找。

## 什么是DI？

Dependency Injection，即依赖注入，和IOC是同一个概念不同角度的描述。组件之间的依赖关系由容器在运行期决定，动态的将依赖关系注入到组件之中。DI的目的是为了提升组件重用的频率，并为系统搭建一个灵活、可扩展的平台。

## 举个栗子

### IOC

找女朋友一般情况就是到处去看哪里有符合自己审美的，然后打听她们的兴趣爱好、电话等等。传统程序开发也是如此，在一个对象中，如果要使用其他的对象，我们需要手动new一个。

Ioc相当于一个婚恋平台，你只需要告诉它们你的要求，然后它们会根据要求给你提供一个女朋友，整个过程都是由平台来控制。Spring的开发方式也是如此。所有的类都会在容器中登记，告诉spring它是什么，需要什么，然后spring会在运行到适当的时候把你要的东西给你，同时也把你给其他需要你的东西。所有类的创建、销毁都有spring控制。以前是对象控制其他对象，现在是spring控制所有对象，所以叫控制反转。

### DI

IoC的一个重点就是在系统运行中，动态的向某个对象提供它所需要的其他对象，这就是通过DI实现的。

比如对象A需要操作数据库，以前我们是自己编码获取一个Connction对象，现在我们只需要告诉spring，A中需要一个Connection，至于Connection怎么构造、何时构造我们都不需要知道。spring会在适当的时候向A注射一个Connection，这就是依赖注入。

## tiny-spring分析

克隆项目到本地

```js
git clone https://github.com/code4craft/tiny-spring.git
```

我在原来项目的基础上添加了一些中文注释，github项目地址：https://github.com/zcsherrydc/zcs-spring.git

每一步的命令需要更改为对应的序号，如第一步就是step1，第二步就是step2

```js
git checkout origin/step1
```

### step1-最基本的容器

```js
git checkout step-1-container-register-and-get
```



IoC最基本的角色有两个：容器（BeanFactory）和Bean本身。<code>BeanDefinition</code>封装了Bean对象。

![1569327273103](Spring-IoC/1569327273103.png)

### step2-在工厂中创建bean

```js
git checkout step-2-abstract-beanfactory-and-do-bean-initilizing-in-it
```

step1中的bean是初始化好之后再set进去的，实际使用中我们希望容器来管理bean的创建。为了方便扩展，抽象出BeanFactory接口和AbstractBeanFactory类。

![image-20200325112410383](Spring-IoC/image-20200325112410383.png)

AbstractBeanFactory里面最重要的是抽象方法<code>doCreateBean()</code>。通过反射获取bean的实例，放入BeanDefinition中，并注册到BeanFactory。

```java
//代码与最终代码略有不同
public void registerBeanDefinition(String name, BeanDefinition beanDefinition) {
        Object bean = doCreateBean(beanDefinition);
        beanDefinition.setBean(bean);
        beanDefinitionMap.put(name, beanDefinition);
}
```

测试

```java
    @Test
    public void test() {
//        初始化
        BeanFactory beanFactory = new AutowireCapable();

//        注入bean
        BeanDefinition beanDefinition = new BeanDefinition();
        beanDefinition.setBeanClassName("zcs.ioc.HelloWorldService");
        beanFactory.registerBeanDefinition("helloWorldService",beanDefinition);

//        获取bean
        HelloWorldService helloWorldService= (HelloWorldService) beanFactory.getBean("helloWorldService");
        helloWorldService.helloWorld();
    }
```

我们需要给BeanFactory传入<code>完整类名</code>和<code>实例名称</code>。这些都可以通过xml文件获取，在step4中完成。还有一个问题就是现在是通过无参构造函数创建的对象，内部成员变量都是null，所以下一步是对成员变量赋值。

### step3-为bean注入属性

```java
git checkout step-3-inject-bean-with-property
```

创建两个类，<code>PropertyValue</code>和<code>PropertyValues</code>，前者保存一个字段的名称和对应值，后者有一个集合保存了所有的<code>PropertyValue</code>。每个BeanDefinition有一个<code>PropertyValues</code>。

<code>AutowireCapable</code>中新增属性注入方法。Spring使用setter进行注入，这里为了简单使用Field。

```java
protected void applyPropertyValues(Object bean, BeanDefinition mbd) throws Exception {
        for (PropertyValue propertyValue : mbd.getPropertyValues().getPropertyValues()) {
            Field declaredField = bean.getClass().getDeclaredField(propertyValue.getName());
            declaredField.setAccessible(true);
            declaredField.set(bean, propertyValue.getValue());
        }
    }
```

HelloWorldService新增text字段。

### step4-读取xml来初始化bean

```
git checkout step-4-config-beanfactory-with-xml
```

看一下文件结构：

Resource

| 类名           | 说明                                                    |
| -------------- | ------------------------------------------------------- |
| Resource       | 接口，通过<code>getInputStream()</code>获取资源的输入流 |
| UrlResource    | 实现Resource接口，通过URL获取资源                       |
| ResourceLoader | 资源加载类，通过getResource()获取一个Resource对象       |



| 类名                         | 说明                                                         |
| ---------------------------- | ------------------------------------------------------------ |
| BeanDefinition               | Bean的包装类，包括Bean的名字、类型、属性键值对               |
| BeanDefinitionReader         | 解析 `BeanDefinition` 的接口。通过 `loadBeanDefinitions(String)` 从一个地址来加载类定义。 |
| AbstractBeanDefinitionReader | 实现 `BeanDefinitionReader` 接口的抽象类                     |
| XmlBeanDefinitionReader      | 具体实现了 `loadBeanDefinitions()` 方法，从 XML 文件中读取类定义 |

XmlBeanDefinitionReader负责读取xml，把<bean>标签封装成BeanDefinition注册到registry，之后把registry都注册到AutowireCapableBeanFactory中

![image-20200325134442636](Spring-IoC/image-20200325134442636.png)

整个读取流程：

![1569395793717](Spring-IoC/1569395793717.png)

1. new一个`ResourceLoader`放入`XmlBeanDefinition`中，通过`loadBeanDefinition`传入xml文件地址

- 获取xml文件的输入流
- 通过`DocumentBuilder`解析出`Document`。先获取所有的bean，然后根据bean名称和类名构建空实例，对当前的bean继续解析，根据`property`实现对bean属性的赋值

2. 将解析好的bean注册到BeanFactory

### step5-为bean注入bean

```
git checkout step-5-inject-bean-to-bean
```

目前为止，我们只是处理了bean之间没有依赖的情况，还无法处理bean之间的依赖。

我们定义一个BeanReference来表示这个属性是对另一个Bean的引用。在`XMLBeanDefinition`的`processProperty`方法中加入判断。如果是ref，就new一个BeanReference封装到PropertyValue.

```java
    private void processProperty(Element ele, BeanDefinition beanDefinition) {
        NodeList propertyNode = ele.getElementsByTagName("property");
        for (int i = 0; i < propertyNode.getLength(); i++) {
            Node node = propertyNode.item(i);
            if (node instanceof Element) {
                Element propertyEle = (Element) node;
                String name = propertyEle.getAttribute("name");
                String value = propertyEle.getAttribute("value");
                //---------------新加入代码----------------------
//                name对应的是value
                if (value!=null && value.length()>0) {
                    beanDefinition.getPropertyValues().addPropertyValue(new PropertyValue(name, value));
                }else{
//                    name对应的是ref
                    String ref=propertyEle.getAttribute("ref");
                    if (ref == null || ref.length() == 0) {
                        throw new IllegalArgumentException("Configuration problem: <property> element for property '"
                                + name + "' must specify a ref or value");
                    }
                    BeanReference beanReference = new BeanReference(ref);
                    beanDefinition.getPropertyValues().addPropertyValue(new PropertyValue(name, beanReference));
                }
            }
        }
    }
```

{%asset_img  1569402791468.png %}

![1569402791468](Spring-IoC/1569402791468.png)

解析完成之后，BeanFactory中只有<String, BeanDefinition>信息，并没有bean实例，即bean为null。

同时为了解决循环依赖的问题，我们使用懒加载的方式，即在需要使用的时候才创建。如果属性对应的bean找不到，那么就先创建。因为总是先创建后注入，所以不会存在循环依赖的问题。

```java
//        获取bean
        HelloWorldService helloWorldService = (HelloWorldService) beanFactory.getBean("helloWorldService");
        helloWorldService.helloWorld();				
```

xml中的配置

```xml
  <bean name="helloWorldService" class="zcs.ioc.HelloWorldService">
        <property name="text" value="Hello World!"></property>
        <property name="outputService" ref="outputService"></property>
    </bean>

    <bean name="outputService" class="zcs.ioc.OutputService">
        <property name="helloWorldService" ref="helloWorldService"></property>
    </bean>
```

### step6-ApplicationContext登场

```
git checkout step-6-invite-application-context
```

现在功能差不多齐全了，但是使用起来比较麻烦，于是引入`ApplicationContext`接口，并在`refresh()`方法下进行bean的初始化。

![1569404506979](Spring-IoC/1569404506979.png)

```JAVA
public ClassPathXmlApplicationContext(String configLocation, AbstractBeanFactory beanFactory) throws Exception {
        super(beanFactory);
        this.configLocation = configLocation;
        refresh();
    }
```

所以使用的时候只需要传入xml地址就可以了

```JAVA
public void test() throws Exception{
        ApplicationContext applicationContext=new ClassPathXmlApplicationContext("tinyioc.xml");
        HelloWorldService helloWorldService = (HelloWorldService) applicationContext.getBean("helloWorldService");
        helloWorldService.helloWorld();
    }
```

至此为止，tiny-spring的IoC部分基本结束。

# AOP

## 什么是AOP

即面向切面编程，将分散在各个业务逻辑代码中相同的部分通过横向切割的方式提取到一个独立的模块中

比如日志记录、安全控制、事务处理、异常处理等

![image-20200327095900404](Spring-IoC/image-20200327095900404.png)

Spring Aop主要是通过动态代理实现的，先看一下代理的相关内容

## 什么是代理

给目标对象提供一个代理对象，并由代理对象控制对目标对象的引用。

使用代理的最主要原因就是在不改变目标对象的情况下对方法进行增强。比如，增加日志等。

### 举个栗子

有一个接口，只有hello的方法

```java
public interface IHello{
    void hello();
}
```

有一个实现类

```java
public class HelloImpl implements IHello{
    @Override
    public void hello(){
        System.out.println("hello");
    }
}
```

现在想在方法被调用的时候添加日志。如果直接在实现类上改，那么所有的实现类都需要改，显而易见，这是非常繁琐的一件事。当新增实现类时，也需要做这种繁琐重复的事情，代码可维护性极低。

那么该怎么才能一次性完成这项工作呢？这就需要代理了。

### 静态代理

举个栗子理解一下

有一排瓶子放在那不动，任务是给每个瓶子贴上标签。

方法一：每次需要改标签你要自己一个个去贴。

方法二：现在有了流水线，你把之前贴标签的任务交给了流水线上的机器来做，只要往机器里输入标签内容，把需要改的瓶子放入流水线，出来的就是改过标签的瓶子了，这样就方便多了。

这里的瓶子就相当于实现类，静态代理就相当于方法二，代理类相当于流水线的机器，在代理类中进行方法的修改（比如添加日志）。只要将需要修改的实现类（委托类）传给代理类即可。

`target`是需要修改的实现类，通过构造函数传入。

```java
#实现接口或者继承委托类都可以
public class HelloProxy implements IHello {
    private IHello target;

    public HelloProxy(IHello target) {
        this.target = target;
    }

    @Override
    public void hello() {
        System.out.println("start");
        target.hello();
        System.out.println("end");
    }
}
```



```java
 	@Test
	public void test(){
        IHello hello=new HelloImpl();
        HelloProxy helloProxy=new HelloProxy(hello);
        helloProxy.hello();
    }
```

输出结果如下：

![1571807561145](Spring-IoC/1571807561145.png)

**优点**

客户端不需要知道实现类是什么，只需知道代理类即可

**缺点**

- 代理类和实现类（委托类）实现了相同的接口，接口类每增加一个方法，代理类和实现类都需要实现，增加了维护的复杂度。例如想在每个方法都添加日志，那么代理类的每个实现方法也都要添加
- 一个代理类只对应一种接口，如果想对两个接口都添加日志，那么就需要两个代理类。静态代理只能为特定的接口服务

### 动态代理

在程序开发中，静态代理会产生许多代理类，所以就需要动态代理了。

动态代理是通过反射机制实现的，能够代理各种类型的对象。缺点：委托类一定要有接口

实现动态代理需要两个类：

-  java.lang.reflect.InvocationHandler接口

```java
  public Object invoke(Object proxy, Method method, Object[] args) throws Throwable;
```

该接口只有一个方法，代理类需要实现这个方法来实现方法的调用

-  java.lang.reflect.Proxy类

   ```java
   public static Object newProxyInstance(ClassLoader loader,
                                             Class<?>[] interfaces,
                                             InvocationHandler h)
           throws IllegalArgumentException
   ```

   通过该方法得到一个代理类对象

代理类

```java
public class DynamicProxy{
    private Object target;

    //传入实现类的实例
    public DynamicProxy(Object target) {
        this.target = target;
    }

    public Object getProxy() {
        return Proxy.newProxyInstance(target.getClass().getClassLoader(),
                target.getClass().getInterfaces(), new InvocationHandler(){
                    @Override
                    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                        System.out.println("start");
                        Object ret = method.invoke(target, args);
                        System.out.println("end");
                        return ret;
                    }
                });
    }  
}
```

测试

```java
@Test
    public void test(){
//        动态代理
        DynamicProxy dynamicProxy = new DynamicProxy(new HelloImpl());
        IHello hello1 = (IHello) dynamicProxy.getProxy();
        hello1.hello();
    }
```

![1571816689909](Spring-IoC/1571816689909.png)

#### 原理

细说一下底层实现原理，主要实现就是`newProxyInstance`方法：

~~~java
@CallerSensitive
public static Object newProxyInstance(ClassLoader loader,
                                      Class<?>[] interfaces,
                                      InvocationHandler h)
    throws IllegalArgumentException
{
	```
    /*
     * 生成代理类的class
     */
    Class<?> cl = getProxyClass0(loader, intfs);

    /*
     * 使用我们实现的InvocationHandler作为参数调用构造方法来获得代理类的实例
     */
    try {
        ```
        final Constructor<?> cons = cl.getConstructor(constructorParams);
        ```
        return cons.newInstance(new Object[]{h});
    } catch ···
}
~~~

`getProxyClass0`方法是取缓存

```java
private static final WeakCache<ClassLoader, Class<?>[], Class<?>>
        proxyClassCache = new WeakCache<>(new KeyFactory(), new ProxyClassFactory());

private static Class<?> getProxyClass0(ClassLoader loader,
                                       Class<?>... interfaces) {
    // 对代理进行了缓存
    return proxyClassCache.get(loader, interfaces);
}
```

- 包装ClassLoader作为第一层缓存的key，取出第二层缓存
- 接口数组作为第二层缓存的key，取出value，如果不为空直接返回
- 如果为空， 调用ProxyClassFactory.apply()
- 调用ProxyGenerator.generateProxyClass()，最终调用generateClassFile()生成代理类的字节码
- 通过defineClass0()将字节码转化为Class实例

~~~java
//采用两层缓存
private final ConcurrentMap<Object, ConcurrentMap<Object, Supplier<V>>> map
        = new ConcurrentHashMap<>();

final class WeakCache<K, P, V> {    
    public V get(K key, P parameter) {
        ```
        //第一层key是包装后的ClassLoader
        ConcurrentMap<Object, Supplier<V>> valuesMap = map.get(cacheKey);
        //第二层key是接口数组的封装，用来查找代理类
        Object subKey = Objects.requireNonNull(subKeyFactory.apply(key, parameter));
	    Supplier<V> supplier = valuesMap.get(subKey);
        Factory factory = null;

        while (true) {
            if (supplier != null) {
                //创建代理类 调用了Factory.get()，又调用了ProxyClassFactory.apply()
                V value = supplier.get();
                if (value != null) {
                    return value;
                }
            }
            if (factory == null) {
                factory = new Factory(key, parameter, subKey, valuesMap);
            }
            if (supplier == null) {
                supplier = valuesMap.putIfAbsent(subKey, factory);
                if (supplier == null) {
                    supplier = factory;
                }
            } else {
                if (valuesMap.replace(subKey, supplier, factory)) {
                    supplier = factory;
                } else {
                    supplier = valuesMap.get(subKey);
                }
            }
        }
	}
}

//ProxyClassFactory是Proxy的静态内部类，实现了BiFunction接口
private static final class ProxyClassFactory
        implements BiFunction<ClassLoader, Class<?>[], Class<?>>{
    @Override
    public Class<?> apply(ClassLoader loader, Class<?>[] interfaces) {
        ```
        //生成代理类的字节码
        byte[] proxyClassFile = ProxyGenerator.generateProxyClass(
    			proxyName, interfaces, accessFlags);
        try {
            //根据二进制字节码返回Class实例
            return defineClass0(loader, proxyName,
                                proxyClassFile, 0, proxyClassFile.length);
        }
    }
}

public static byte[] generateProxyClass(final String name, Class<?>[] interfaces, int accessFlags) {
        ProxyGenerator gen = new ProxyGenerator(name, interfaces, accessFlags);
        // 真正生成字节码的方法
        final byte[] classFile = gen.generateClassFile();
	    return classFile;
}	
~~~

代理类生成的最终方法ProxyGenerator.generateClassFile()

```java
private byte[] generateClassFile() {
        //增加 hashcode、equals、toString方法
        addProxyMethod(hashCodeMethod, Object.class);
        addProxyMethod(equalsMethod, Object.class);
        addProxyMethod(toStringMethod, Object.class);
        // 将所有接口中的所有方法添加到代理方法中
        for (Class<?> intf : interfaces) {
            for (Method m : intf.getMethods()) {
                addProxyMethod(m, intf);
            }
        }
        //为类中的方法生成字段信息和方法信息
        // 生成代理类的构造函数、代理方法
        // 为代理类生成静态代码块，对一些字段进行初始化
        // 编写最终类文件
        try {
            //magic;
            dout.writeInt(0xCAFEBABE);
            //次要版本、主版本、访问标识、本类名、父类名、接口、字段、方法、类文件属性：对于代理类来说没有类文件属性;
        } 
        return bout.toByteArray();
    }
```

生成的代理类

```java
// 继承了Proxy类和实现IHello接口
public final class $Proxy0 extends Proxy implements IHello {
  // 变量，都是private static Method  XXX
  private static Method m1;
  private static Method m3;
  private static Method m2;
  private static Method m0;
 
  // 代理类的构造函数，其参数正是是InvocationHandler实例，Proxy.newInstance方法就是通过通过这个构造函数来创建代理实例的
  public $Proxy0(InvocationHandler paramInvocationHandler){
    super(paramInvocationHandler);
  }
  
  // 接口代理方法
  public final void sayHello(){
    this.h.invoke(this, m3, null);
  }
  // 静态代码块对变量进行一些初始化工作
  static
  {
    try
    {
	  // 这里每个方法对象 和类的实际方法绑定
      m1 = Class.forName("java.lang.Object").getMethod("equals", new Class[] { Class.forName("java.lang.Object") });
      m3 = Class.forName("com.jpeony.spring.proxy.jdk.IHello").getMethod("sayHello", new Class[0]);
      m2 = Class.forName("java.lang.Object").getMethod("toString", new Class[0]);
      m0 = Class.forName("java.lang.Object").getMethod("hashCode", new Class[0]);
      return;
    }
  }
}
```

每个代理实例都有一个关联的InvocationHandler，以m3为例，绑定了IHello接口的sayHello()，当代理实例调用方法时，根据方法参数调用InvocationHandler实例的invoke()，

### CGLIB

肯定不可能所有的类都实现了接口，所以动态代理并不适用所有类

这时候就需要CGLIB登场，原理是在通过ASM指令动态生成被代理类的子类，并重写所有的非private、非final的方法。<font color='red'>委托类、方法不能是final/staic</font>

写一个国际通用入门类

```java
public class HelloWorld {
    public void hello(){
        System.out.println("hello world");
    }
}
```

写CGLIB代理类，需要实现`MethodInterceptor`接口

```java
import net.sf.cglib.proxy.Enhancer;
import net.sf.cglib.proxy.MethodInterceptor;
import net.sf.cglib.proxy.MethodProxy;

import java.lang.reflect.Method;

public class CGLibTest{
    
    public Object getInstance(Class<?> target){
        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(target);
        enhancer.setCallback(new MethodInterceptor() {
            @Override
            public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
                System.out.println("开始");
                Object invoke = methodProxy.invokeSuper(o, objects);
                System.out.println("结束");
                return invoke;
            }
        });
        return enhancer.create();
    }

    public static void main(String[] args) {
        HelloWorld helloWorld= (HelloWorld) new CGLibTest().getInstance(HelloWorld.class);
        helloWorld.hello();
    }
}

================测试====================
开始
hello world
结束
```

#### 原理

使用ASM来操作字节码生成新类

## AOP原理

底层原理是动态代理，使用了两种：

- JDK动态代理，默认，实现接口的类，创建速度快
- CGLib动态代理，适用没有实现接口的类，运行速度快

如果是单例，最好使用CGLib代理

## 相关概念

### Join point（连接点）

能够被拦截的地方，每个成员方法都可以是连接点

### Pointcut（切点）

具体定位的连接点(方法)就是切点

### Advice（通知/增强）

表示在切点上具体需要执行的操作 ，提供了前置、后置、环绕、异常、返回这5种操作

### Aspect（切面）

类似Java中的类声明，由切点和相应的`Advice`结合而成

### Weaving（织入）

把Advice应用到目标对象来创建新的代理对象的过程

## Spring对AOP的支持 

- 基于代理：现在基本不使用
  首先看一下增强接口的继承关系图
  ![image-20200327100340454](Spring-IoC/image-20200327100340454.png)


  分为5类：

  ![image-20200327100413081](Spring-IoC/image-20200327100413081.png)

- xml配置

- 注解

## tiny-spring分析

继续之前的代码分析

### step7-使用JDK动态代理实现AOP织入

我们知道，想要成功代理的话需要委托实例和委托实例的接口

这是一个完整的动态代理结构图，我们一点一点分析

![image-20200327115224334](Spring-IoC/image-20200327115224334.png)

我们把实例和Class(其实是Interface)封装进TargetSource，把TargetSource和MethodInterceptor封装进AdvisedSupport，那么MethodInterceptor是个什么东西呢？

AOP中有两个重要的角色：MethodInterceptor和MethodInvocation，它们分别对应上文提到的Advice和Join Point两个概念，我们知道AOP的作用就是在哪干，干什么，Join Point表示在哪个方法上执行，而Advice就是要干什么

```java
//对应advice
public interface MethodInterceptor extends Interceptor {
    Object invoke(MethodInvocation var1) throws Throwable;
}
```

```java
//对应joined point
public interface MethodInvocation extends Invocation {
    Method getMethod();
}
```

所以我们需要做的事情都要写在MethodInterceptor中，而且在上一部分中我们了解到它代表环绕增强，表示在目标方法前后实施增强

让我们来写一个小代码练练手，还是用之前的HelloWorldService接口

1、先写一个MethodInterceptor实现类，表示我们需要做什么
这个接口只有一个方法，重写即可，invocation.proceed()代表执行原方法，前后的代码是我们新增的非业务逻辑代码，这样就写好了要做什么

```java
public class TimerInterceptor implements MethodInterceptor {

   @Override
   public Object invoke(MethodInvocation invocation) throws Throwable {
      long time = System.nanoTime();
      System.out.println("Invocation of Method " + invocation.getMethod().getName() + " start!");
      Object proceed = invocation.proceed();
      System.out.println("Invocation of Method " + invocation.getMethod().getName() + " end! takes " + (System.nanoTime() - time)
            + " nanoseconds.");
      return proceed;
   }
}
```

2、接下来写要在哪做，通过前面的分析我们知道，需要实现MethodInvocation接口
这是MethodInvocation的继承关系图，可以看到，顶层接口是Joinpoint，也就是连接点

![image-20200327115709633](Spring-IoC/image-20200327115709633.png)

新建一个实现类，需要重写上述接口的所有方法

```java
public class ReflectiveMethodInvocation implements MethodInvocation {

    private Object target;

    private Method method;

    private Object[] args;

    public ReflectiveMethodInvocation(Object target, Method method, Object[] args) {
        this.target = target;
        this.method = method;
        this.args = args;
    }

    @Override
    public Method getMethod() {
        return method;
    }

    @Override
    public Object[] getArguments() {
        return args;
    }

    @Override
    public Object proceed() throws Throwable {
        return method.invoke(target, args);
    }

    @Override
    public Object getThis() {
        return target;
    }

    @Override
    public AccessibleObject getStaticPart() {
        return method;
    }
}
```

3、新建一个动态代理类，通过getProxy反射获取一个实例对象，然后调用invoke方法实现动态代理

```java
public class JdkDynamicAopProxy implements AopProxy, InvocationHandler {

   private AdvisedSupport advised;

   public JdkDynamicAopProxy(AdvisedSupport advised) {
      this.advised = advised;
   }

    @Override
   public Object getProxy() {
      return Proxy.newProxyInstance(getClass().getClassLoader(), new Class[] { advised.getTargetSource().getTargetClass() }, this);
   }

   @Override
   public Object invoke(final Object proxy, final Method method, final Object[] args) throws Throwable {
      MethodInterceptor methodInterceptor = advised.getMethodInterceptor();
      return methodInterceptor.invoke(new ReflectiveMethodInvocation(advised.getTargetSource().getTarget(), method,
            args));
   }
}
```

4、测试

```java
		// --------- helloWorldService without AOP
		ApplicationContext applicationContext = new ClassPathXmlApplicationContext("tinyioc.xml");
		HelloWorldService helloWorldService = (HelloWorldService) applicationContext.getBean("helloWorldService");
		helloWorldService.helloWorld();

		// --------- helloWorldService with AOP
		// 1. 设置被代理对象(Joinpoint)
		AdvisedSupport advisedSupport = new AdvisedSupport();
		TargetSource targetSource = new TargetSource(helloWorldService, HelloWorldService.class);
		advisedSupport.setTargetSource(targetSource);

		// 2. 设置拦截器(Advice)
		TimerInterceptor timerInterceptor = new TimerInterceptor();
		advisedSupport.setMethodInterceptor(timerInterceptor);

		// 3. 创建代理(Proxy)
		JdkDynamicAopProxy jdkDynamicAopProxy = new JdkDynamicAopProxy(advisedSupport);
		HelloWorldService helloWorldServiceProxy = (HelloWorldService) jdkDynamicAopProxy.getProxy();

        // 4. 基于AOP的调用
        helloWorldServiceProxy.helloWorld();
```

### step8-使用AspectJ管理切面

在完成了织入之后，需要考虑的问题就是对什么类以及什么方法进行织入，也就是切点。Spring中`ClassFilter`和`MethodMatcher`分别表示类和方法做匹配，怎么匹配呢？

AspectJ是对java的aop增强的一门语言，有单独的编译器，Spring借鉴了AspectJ风格的切点表达式，新建AspectJExpressionPointcut实现上述两个接口，通过切点解析器将AspectJ风格的表达式解析成切点表达式

pointcutParser就是切点解析器，supportedPrimitives是一个set集合，表示AspectJ语言的关键字

pointcutParser将String类型的expression解析包装成PointcutExpression，然后可以就可以匹配类和方法了

```java
public AspectJExpressionPointcut(Set<PointcutPrimitive> supportedPrimitives) {
        pointcutParser = PointcutParser
.getPointcutParserSupportingSpecifiedPrimitivesAndUsingContextClassloaderForResolution(supportedPrimitives);
    }

	protected void checkReadyToMatch() {
        if (pointcutExpression == null) {
            pointcutExpression = buildPointcutExpression();
        }
    }
	//    字符串转切点表达式
    private PointcutExpression buildPointcutExpression() {
        return pointcutParser.parsePointcutExpression(expression);
    }
 	// 将表达式和类做匹配，返回匹配结果
    @Override
    public boolean matches(Class<?> targetClass) {
        checkReadyToMatch();
        return pointcutExpression.couldMatchJoinPointsInType(targetClass);
    }

    // 将表达式和方法做匹配，返回匹配结果
    @Override
    public boolean matches(Method method, Class<?> targetClass) {
        checkReadyToMatch();
        ShadowMatch shadowMatch = pointcutExpression.matchesMethodExecution(method);
        if (shadowMatch.alwaysMatches()) {
            return true;
        } else if (shadowMatch.neverMatches()) {
            return false;
        }
        // TODO:其他情况不判断了！见org.springframework.aop.aspectj.RuntimeTestWalker
        return false;
    }
```

AspectJ包帮我们做好了匹配的事情，只不过我们使用动态代理而不是它的编译器实现了织入

### step9-将AOP融入Bean的创建过程中

现在我们已经解决了在哪做，做什么的问题，接下来就要把AOP融入IOC容器的Bean中，我们首先要解决在IOC容器的哪里织入AOP，然后是要为哪些对象织入

Spring提供了BeanPostProcessor，只要Bean实现了这个接口，那么在初始化的时候会优先找到它们，并且在初始化的过程中调用这个接口，从而实现对BeanFactory无侵入的扩展

在 `postProcessorAfterInitialization` 方法中，使用动态代理的方式，返回一个对象的代理对象。解决了 **在 IOC 容器的何处植入 AOP** 的问题。

```java
public interface BeanPostProcessor {
	//初始化方法之前调用
    Object postProcessBeforeInitialization(Object bean, String beanName) throws Exception;
	//初始化方法之后调用
    Object postProcessAfterInitialization(Object bean, String beanName) throws Exception;
}
```

AspectJAwareAdvisorAutoProxyCreator是实现AOP织入的关键类，它实现了BeanPostProcessor接口，它会扫描所有的切点（AspectJExpressionPointcutAdvisor包装了Pointcut和AspectJExpressionPointcut），并对bean做织入

![image-20200410161806277](Spring-IoC/image-20200410161806277.png)

```java
public Object postProcessAfterInitialization(Object bean, String beanName) throws Exception {
    //提供代理pointcut与advice的AspectJExpressionPointcutAdvisor不需要处理
    if (bean instanceof AspectJExpressionPointcutAdvisor)
        return bean;

    if (bean instanceof MethodInterceptor)
        return bean;

    List<AspectJExpressionPointcutAdvisor> advisors = beanFactory
            .getBeansForType(AspectJExpressionPointcutAdvisor.class);
    for (AspectJExpressionPointcutAdvisor advisor : advisors) {
        if (advisor.getPointcut().getClassFilter().matches(bean.getClass())) {
            AdvisedSupport advisedSupport = new AdvisedSupport();
            advisedSupport.setMethodInterceptor((MethodInterceptor) advisor.getAdvice());
            advisedSupport.setMethodMatcher(advisor.getPointcut().getMethodMatcher());

            TargetSource targetSource = new TargetSource(bean, bean.getClass().getInterfaces());
            advisedSupport.setTargetSource(targetSource);

            return new JdkDynamicAopProxy(advisedSupport).getProxy();
        }
    }
    return bean;
}
```

动态代理类的invoke

```java
public Object invoke(final Object proxy, final Method method, final Object[] args) throws Throwable {
      MethodInterceptor methodInterceptor = advised.getMethodInterceptor();
      // 判断是否为要拦截的方法，是则执行 Advice 逻辑 否则正常执行
if (advised.getMethodMatcher() != null
      && advised.getMethodMatcher().matches(method, advised.getTargetSource().getTarget().getClass())) {
   return methodInterceptor.invoke(new ReflectiveMethodInvocation(advised.getTargetSource().getTarget(),
         method, args));
} else {
   return method.invoke(advised.getTargetSource().getTarget(), args);
}
  }
```

xml

```xml
<bean id="autoProxyCreator" class="zcs.aop.AspectJAwareAdvisorAutoProxyCreator"></bean>

<bean id="timeInterceptor" class="zcs.aop.TimerInterceptor"></bean>

<!-- Creator 将对 Advisor 类型的 bean 进行扫描和处理 -->
<bean id="aspectjAspect" class="zcs.aop.AspectJExpressionPointcutAdvisor">
    <!-- 调用对应的 setter 方法进行 Property 的注入 -->
    <property name="advice" ref="timeInterceptor"></property>
    <property name="expression" value="execution(* zcs.*.*(..))"></property>
</bean>
```

**整个流程如下**

1. `AutoProxyCreator`（实现了BeanPostProcessor接口）最先被实例化
2. 其他Bean开始实例化，`AutoProxyCreator`加载BeanFactory中所有的`PointcutAdvisor`，然后依次使用`PointcutAdvisor`内置的ClassFilter判断当前对象需不需要拦截
3. 如果是，生成`AdvisedSupport`交给AopProxy生成代理对象
4. `AopProxy`实现了`InvocationHandler`接口，在invoke函数中，首先使用advised.getMethodMatcher()判断是不是要拦截的方法，如果是则交给Advice（自定义的`MethodInterceptor`）执行，如果不是，则直接执行。

### step10-使用CGLib进行类的织入

前面说到，JDK动态代理只能对接口实现代理，如果类没有实现接口，那么必须要用CGLib来实现。

我们需要新建一个ProxyFactory根据TargetSource类型来自动创建代理

```JAVA
public class ProxyFactory extends AdvisedSupport implements AopProxy {
    // 在 creator 中，生成相应的动态代理的时候就可以使用工厂类的 getProxy()
    @Override
    public Object getProxy() {
        return createAopProxy().getProxy();
    }

    protected final AopProxy createAopProxy() {
        return new Cglib2AopProxy(this);
    }

    //...可以根据 TargetSource 决定使用不同的动态代理
    protected final AopProxy createAopProxy(TargetSource targetSource) {
        return new JdkDynamicAopProxy(this);
    }
}
```

TargetSource修改变量

```java
	public class TargetSource {

        private Class<?> targetClass;

        private Class<?>[] interfaces;

        private Object target;
    }
```

整体流程：

![image-20200621205456418](Spring-IoC/image-20200621205456418.png)

