---
title: java面试题总结
tags:
---

# 基础

### 8种基本数据类型

boolean、char、byte、short、int、long、float、double

### "i"和new String("i")

"i"：jvm会将其分配到常量池中

new String("i")：会分配到堆内存中

### Comparable和Comparator

Comparable接口用于定义对象的自然排序，是排序接口,`compareTo(Object obj)`方法用来排序，类只能有一个Comparable接口

Comparator接口用于用户自定义对象的顺序，是比较接口，`compare(Object obj1, Object obj2)`方法用来排序，类可以有多个Comparator接口

### 异常

![image-20200330145411951](https://tva1.sinaimg.cn/large/008i3skNly1gptsku22ymj30oy0fxtae.jpg)

Throwable是顶层父类，派生了Error和Exception。Error代表JVM的错误，程序员不用关心。

**运行时异常**(不受检异常)：RuntimeException类及其子类表示JVM运行期间可能发生的错误，编译器不会检查此类错误，也不强制要求处理异常，比如NullPoint、ArrayIndexOutBound

**非运行时异常**(受检异常)：Exception中除了RuntimeException类及其子类的其他异常，比如IOException、SQLException

#### 异常处理

throws：用在函数上，后面跟异常类，用来声明异常

throw：用在函数内，后面跟异常对象，抛出具体的问题对象

try catch捕获

下列程序输出什么

```java
class Annoyance extends Exception {
}

class Sneeze extends Annoyance {
}

class Human {
    public static void main(String[] args) throws Exception {
        try {
            try {
                throw new Sneeze();
            } catch (Annoyance a) {
                System.out.println("Caught Annoyance");
                throw a;
            }
        } catch (Sneeze s) {
            System.out.println("Caught Sneeze");
            return;
        } finally {
            System.out.println("Hello World!");
        }
    }
}
//输出
//Caught Annoyance
//Caught Sneeze
//Hello World!
```

### Integer的缓存策略

在类加载时创建-128到127的Integer对象，保存在cache数组，程序调用Integer.valueOf(i)，如果在这区间就直接取缓存，类似的还有Short、Long。通过new Integer()创建的一律不相等

### 强制类型转换

将小于int的转为int进行计算，得到的值也是int。short s=1;s=s+1;编译报错，必须要s=(short)(s+1);或者s+=1;

### equals和hashCode

如果两个对象equals相同，那么hashCode一定相同；反之不成立

### 拷贝

### String为什么是final

- 常量池：
- 安全：多线程安全；如果是可变的，当String作为Hash的key时，会改变键值的唯一性
- 效率：

### 内部类

外部类和内部类的互相访问分为两个方向，每个方向又分为访问静态和非静态属性/方法

1. 静态内部类可以有静态和非静态成员，而非静态内部类只能有非静态成员
2. 静态内部类只能访问外部类的静态成员，访问非静态成员需要创建外部类实例；而非静态内部类可以访问外部类的所有成员

静态内部类只能访问外部类的静态成员和静态方法，

```java
class Outer {
    private int o1 = 1;
    private static int o2 = 2;

    public void run() {
//       一、 访问非静态内部类
//        1 访问非静态属性需要一个内部类实例
        System.out.println(new Inner().i1);
//        2 静态属性：非法

//       二、 访问静态内部类
//        1 访问非静态属性需要创建内部类的实例
        System.out.println(new Outer.StaticInner().s1);
//        2 静态属性：可直接访问
        System.out.println(StaticInner.s2);

    }

    public static void run1() {
//       一、访问非静态内部类
//        1 访问非静态属性：因为这个是静态方法，所以首先需要一个外部类实例，然后再需要一个内部类实例
        System.out.println(new Outer().new Inner().i1);
//        2 无

//       二、 访问静态内部类
//        1 可直接访问静态属性
        System.out.println(StaticInner.s2);
//        2 访问非静态属性需要创建内部类的实例
        System.out.println(new Outer.StaticInner().s1);
    }

    class Inner {
        private int i1 = 3;
//        可以访问外部类的所有属性
        public void run() {
            System.out.println(o1);
            System.out.println(o2);
        }

//        非静态内部类不能定义静态属性和静态方法
//        private static int i2=4;
//        public static void run1() {
//
//        }
    }

    static class StaticInner {
        private int s1 = 5;
        private static int s2 = 6;

        public void run() {
//            非静态属性要创建外部类的实例
            System.out.println(new Outer().o1);
//            只能访问静态属性
            System.out.println(o2);
        }

        public static void r() {
            System.out.println(new Outer().o1);
            System.out.println(o2);
        }
    }
}
```



```java
public class Main {
    public static class StaticInner {
        public void method() {
        }
    }

    public class Inner {
        public void method() {
        }
    }

    public static void main(String[] args) {
        Main.StaticInner staticInner = new Main.StaticInner();
        Main.Inner inner = new Main().new Inner();
    }
}
```

### 抽象类和接口

|          | 抽象类                     | 接口                           |
| -------- | -------------------------- | ------------------------------ |
|          | 表示对象是什么，对类的抽象 | 表示对象能做什么，对行为的抽象 |
| 方法     | 抽象方法和普通方法         | 只能包含抽象方法和默认方法     |
| 成员变量 | 任意类型                   | 只能是public static final      |
| 构造器   | 可以有                     | 无                             |
|          |                            |                                |

### 序列化

内存中的数据对象只有转换成二进制流才可以进行数据持久化和网络传输。这个过程称为序列化，反过来称为反序列化。常用方法：Serializable、json、xml、protobuf

Java序列化

```java
public static void main(String[] args) throws Exception{
    FileOutputStream fileOutputStream = new FileOutputStream("stu.out");
    ObjectOutputStream objectOutputStream = new ObjectOutputStream(fileOutputStream);
    Student zcs = new Student("zcs");
    objectOutputStream.writeObject(zcs);

    FileInputStream fileInputStream = new FileInputStream("stu.out");
    ObjectInputStream objectInputStream = new ObjectInputStream(fileInputStream);
    Student o = (Student)objectInputStream.readObject();
    System.out.println(o.getName());
}

public class Student implements Serializable {
    private String name;

    public Student(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```

### 泛型擦除

泛型的本质是参数化类型，保证编译时类型安全

泛型参数会被擦除到它的第一个边界，如下，只会擦除到A类型

```java
public class Main<T extends A>
```

### 一致性Hash算法

https://zhuanlan.zhihu.com/p/34985026

将整个Hash值空间组成一个虚拟的圆环，假设某哈希函数H的值空间为0-2^32-1，将服务器进行Hash计算得到在环中的位置。将数据进行相同的Hash计算确定在环中的位置，顺时针遇到的第一台服务器就是应该定位的。

为了解决数据倾斜问题，引入虚拟节点机制，即对每个服务节点进行多个Hash计算，每个结果位置都放一个虚拟节点。

<img src="https://raw.githubusercontent.com/zheyday/BlogImg/master/img/image-20200429091750600.png" alt="image-20200429091750600" style="zoom:67%;" />

### 常见Hash算法



### Hash冲突解决

#### 一、开放地址

当关键字的hash地址p出现冲突时，以p为基础，d为增量序列
$$
fn=(f(key)+d)%m
$$
d是增量序列，根据取值方式不同，主要有三种方法

1. 线性探测：d=1,2,3,4,...
2. 二次探测：
   ![1615275382379](https://tva1.sinaimg.cn/large/008i3skNly1gptskwtuzvj30du021a9z.jpg)
3. 双哈希函数探测

#### 二、链地址



### 红黑树

**优点**



**特性**

1. 根节点是黑色
2. 每个节点是黑色或红色
3. 叶子节点是黑色(叶子节点是null)
4. 不能连续两代都是红色节点
5. 从任意节点到其每个叶子节点的所有路径上包含相同数据的黑节点

<img src="https://raw.githubusercontent.com/zheyday/BlogImg/master/img/RedBlackTree.png" alt="RedBlackTree" style="zoom: 80%;" />

**左旋**

<img src="https://raw.githubusercontent.com/zheyday/BlogImg/master/img/LeftRotation.png" alt="LeftRotation" style="zoom:67%;" />

**右旋**

<img src="https://raw.githubusercontent.com/zheyday/BlogImg/master/img/RightRotaion.png" alt="RightRotaion" style="zoom:67%;" />

**插入**

新节点设置为红色(因为这样只可能不满足特性4)，将其插入。如果父节点是红色，这时候会出现三种情况

1. 叔节点是红色：将父节点和叔节点设为黑色，将祖父节点设为红色，这样保证性质5。如果祖父节点是根结点，直接设为黑色即可；如果不是，将祖父节点设为当前节点，继续进行操作
   <img src="https://raw.githubusercontent.com/zheyday/BlogImg/master/img/image-20200502164030854.png" alt="image-20200502164030854" style="zoom: 80%;" />
2. 叔节点是黑色，新节点是左孩子：将父节点设为黑色，破坏了性质5，将祖父节点设为红色并右旋，
   <img src="https://raw.githubusercontent.com/zheyday/BlogImg/master/img/image-20200502173237960.png" alt="image-20200502173237960" style="zoom:80%;" />
3. 叔节点是黑色，新节点是右孩子：将P左旋见下图，然后再像情况2进行右旋
   <img src="https://raw.githubusercontent.com/zheyday/BlogImg/master/img/image-20200502182512948.png" alt="image-20200502182512948" style="zoom:80%;" />

**删除**

如果节点是叶子节点，直接删除；如果只有一个子节点，将子节点上移；如果有两个子节点，选择一个合适的节点作为新的根结点，称为**继承节点**，可以从左子树找最大或者右子树找最小的。如果删除节点是黑色的，需要进行结构调整

### 流

### 限流算法

- 漏桶算法：先存储请求，满的话直接返回，然后以一定速率放出请求，能强行限制数据的传输速率，无法应对突发流量
- 令牌桶算法：以固定速率生产令牌，请求到来之后，如果有令牌则访问，没有直接返回

### IO多路复用

#### select

无差别轮询负责的所有socket，当某个socket有数据时通知用户进程，默认socket是1024

select为每个socket引入一个poll逻辑，用于收集socket发生的事件。当用户process调用select的时候，select会将监控的fds集合拷贝到内核空间（假设监控的是读事件），然后遍历监控的socket，调用poll逻辑检查是否有可读事件，遍历完之后如果没有，则进入睡眠。如果等待期间某个socket有数据可读或者等待超时，则调用select的process被唤醒，然后遍历socket集合进行处理

存在三个问题：

1. fds集合限制为1024
2. fds集合需要拷贝到内核空间
3. 当fds有数据可读时，需要遍历整个集合，而我们希望更精确一些

#### poll

解决了第一个问题，使用pollfd代替select的fd_set

#### epoll

epoll提供了三个函数，

- epoll_create：创建一个epoll对象，存放事件
- epoll_ctl：添加socket到等待队列，向内核注册回调函数，当事件发生时调用回调函数将socket添加到就绪列表
- epoll_wait：返回就绪链表，有一个timeout参数，为-1则阻塞到有事件发生，为0不阻塞

第一个问题，epoll支持的fd上限是最大可以打开文件的数量

第二个问题，解决方案是epoll_ctl函数，每次注册新的事件会把对应的fd拷贝到内核

第三个问题，在epoll_ctl时添加socket到等待队列，向内核注册回调函数，当事件发生时调用回调函数将socket添加到就绪列表。epoll_wait就是等待就绪链表中有无就绪的fd

epoll_wait有两种触发方式：

- 水平触发（LT）：fd有数据可读，就出触发epool_wait
- 边缘触发（ET）：有I/O事件才会触发

现在有这么个场景：

1. A和B建立连接，A在epoll上注册了rfd
2. B向rfd写了2kB的数据
3. epoll_wait告诉A来数据了，可以读了
4. A只读了1kB，B缓冲区还剩1kB

在边缘触发模式下，epoll_wait不会再被触发，这就造成了互相等待导致永久阻塞。所以socket需要设置非阻塞，A收到消息后一直读到出现EAGAIN错误

在水平触发模式下，如果缓冲区有数据一定会触发epoll_wait，那么第4步之后会再次触发epoll_wait。如果要提高并发，要使用非阻塞

### 数据库缓存双写一致性

**先更新缓存，再更新数据库**

一般没人这么写

**先更新数据库，再更新缓存**

这是普遍反对的

1. 线程安全角度，如果请求A和B进行更新操作，那么会出现

   - A更新数据库
   - B更新数据库
   - B更新缓存
   - A更新缓存

   因为网络原因B先比A更新了缓存，就导致了脏数据

**先删除缓存，再更新数据库**

1. A写操作，删除缓存
2. B发现缓存不存在
3. B查询数据库得到旧值
4. B将旧值写入缓存
5. A将新值写入数据库

第4步会导致缓存永远都是脏数据

解决：A写操作采用延时双删策略，延迟是保证能删除B的旧缓存

睡眠会导致相应变慢，可以通过消息队列实现

```java
public void write(String key,Object data){
        redis.delKey(key);
        db.updateData(data);
        Thread.sleep(1000);
        redis.delKey(key);
}
```

**先更新数据库，再删除缓存**

A查询，B更新

1. 缓存失效
2. A查询数据库，得到旧值
3. B将新值写入数据库
4. B删除缓存
5. A将旧值写入缓存

如果发生这种情况，会出现脏数据，但是2读数据要比3快，所以2之后会发生5，5比4要先发生，这样B就把A写的缓存删掉了



# SpringBoot

## 监听器

首先配置监听器类，`@WebListener`表示该类为Listener，常用的几个监听器有：

- ServletContextListener：监听ServletContext对象的创建和销毁
- ServletContextAttributeListener：监听ServletContext对象中属性的改变
- ServletRequestListener：监听Request对象的创建和销毁
- ServletRequestAttributeListener：监听Request对象中属性的改变
- HttpSessionListener：监听session对象的创建和销毁
- HttpSessionAttributeListener：监听session对象中属性的改变

```java
@WebListener
public class MyContextListener implements ServletContextListener {
    @Override
    public void contextInitialized(ServletContextEvent sce) {
        System.out.println("监听器初始化");
    }

    @Override
    public void contextDestroyed(ServletContextEvent sce) {
        System.out.println("监听器销毁");
    }
}
```

然后在主类上添加`@ServletComponentScan`注解，括号里面不加好像也行

```java
@MapperScan("zcs.demo.mapper")
@ServletComponentScan(basePackages = "zcs.demo.*")
@SpringBootApplication()
public class Demo1Application {
    public static void main(String[] args) {
        SpringApplication.run(Demo1Application.class, args);
    }
}
```

## 拦截器

1、首先创建拦截器

```java
import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

/**
 * @author zcs
 */
public class MyInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("pre");
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println("post");
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        System.out.println("after");
    }
}
```

2、配置拦截器，使之生效

```java
@Configuration
public class MyWebMvcConfigurer implements WebMvcConfigurer {
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
      //配置拦截url，是两个*
        registry.addInterceptor(new MyInterceptor()).addPathPatterns("/**");
    }
}
```

## 过滤器

```java
public class TestFilter implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        Filter.super.init(filterConfig);
    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        System.out.println("testFilter");
        filterChain.doFilter(servletRequest,servletResponse);
    }

    @Override
    public void destroy() {
        Filter.super.destroy();
    }
}
```

第一种配置：使用配置类，可以设置执行顺序，Order越小，优先级越高

```java
@Configuration
public class TestFilterConfig {
    @Bean
    public FilterRegistrationBean<TestFilter> registrationBean(){
        FilterRegistrationBean<TestFilter> registration = new FilterRegistrationBean<>();
        registration.setFilter(new TestFilter());
        registration.setOrder(1);
        registration.addUrlPatterns("/*");
        registration.setName("testFilter");
        return registration;
    }

    @Bean
    public FilterRegistrationBean<Test1Filter> registrationBean1(){
        FilterRegistrationBean<Test1Filter> registration = new FilterRegistrationBean<>();
        registration.setFilter(new Test1Filter());
        registration.setOrder(2);
        registration.addUrlPatterns("/*");
        registration.setName("test1Filter");
        return registration;
    }
}
```

第二种配置：使用注解，无法指定顺序，按照类名排序执行。有的博客说使用`@Order`注解，实测是不行的

```java
@WebFilter(urlPatterns = "/*",filterName = "testFilter")
public class TestFilter implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        Filter.super.init(filterConfig);
    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        System.out.println("testFilter");
        filterChain.doFilter(servletRequest,servletResponse);
    }

    @Override
    public void destroy() {
        Filter.super.destroy();
    }
}
```

## 数据绑定

当多个类有相同属性时，使用`@InitBinder`进行区分配置，url也需要对输入参数进行区分

```java
@RestController
@RequestMapping("/indexCategory")
public class IndexCategoryController {
    @GetMapping("info")
    public String info(Admin admin,User user){
        return admin.toString()+" "+user.toString();
    }
    @InitBinder("user")
    public void initUser(WebDataBinder binder){
        binder.setFieldDefaultPrefix("user.");
    }
    @InitBinder("admin")
    public void initAdmin(WebDataBinder binder){
        binder.setFieldDefaultPrefix("admin.");
    }
}
```











# 容器

![image-20200214203248875](https://tva1.sinaimg.cn/large/008i3skNly1gptskx9c8rj30sj0g5jsx.jpg)

### fail-fast

当遍历一个集合时，如果集合的结构被修改了，就会抛出ConcurrentModificationException异常

- 在单线程下，在使用iterator遍历集合过程中，使用list.remove修改了集合，应使用iterator.remove
- 在多线程下，对集合进行修改

以ArrayList为例讲解fail-fast机制：

当使用ArrayList.iterator()返回一个迭代器对象时，迭代器对象的expectedModCount属性被赋值为调用时的modCount，expectedModCount不会再发生变化。如果之后调用add、remove方法，modCount自增，那么再次调用next()时，会先检查两者是否相等，不相等则抛异常

```java
final void checkForComodification() {
    if (modCount != expectedModCount)
        throw new ConcurrentModificationException();
}
```

而iterator.remove()不进行检查，并且手动设置expectedModCount=modCount

### fail-safe

针对线程安全的集合类，并发修改不会抛出异常。iterator对象保存了快照副本，可以并发读取，但是不保证是最新的状态

### Collections常用方法

synchronizedList：将容器转为线程安全容器

sort

binarySearch：查找元素，返回索引

reverse：反转元素

copy

## List

### LinkedList

双向链表，不能随机访问

### ArrayList

使用数组形式实现，默认初始容量10，扩容为1.5倍，非线程安全

### Vector

使用数组形式实现，默认初始容量10，扩容为2倍，线程安全

### Stack

peek：返回栈顶元素

pop：出栈

push：入栈

## Set

### HashSet

底层是由HashMap实现的，值存放在HashMap的key上，HashMap的value统一为PRESENT

```java
private static final Object PRESENT = new Object();
```

### TreeSet

由TreeMap实现，值存放在key上，value统一为PRESENT

## Map

![image-20200403164227892](https://tva1.sinaimg.cn/large/008i3skNly1gptskvtp9gj30mu092dgs.jpg)

### TreeMap

基于红黑树实现，键值对的键所属的类必须实现Comparable接口，重写compareTo()方法，或者TreeSet创建时传入Comparator

### HashTable

不允许空的键值对；同步；遗留类，不建议使用

默认初始容量是11，直接使用key的hashCode，定位桶的时候直接求余

### HashMap

key允许一个为null，value没有限制；非同步

默认初始长度是16，扩容或手动初始化的长度必须是2的幂

看一下几个重要部分

#### Node

HashMap的key和value都被封装到内部类Node中，这是一个单向链表结构

```java
//节点的hash值是由key和value的hashCode异或得到的
public final int hashCode() {
    return Objects.hashCode(key) ^ Objects.hashCode(value);
}
```

#### tableSizeFor()

返回大于指定容量的2的幂

```java
static final int tableSizeFor(int cap) {
    int n = cap - 1;
    n |= n >>> 1;
    n |= n >>> 2;
    n |= n >>> 4;
    n |= n >>> 8;
    n |= n >>> 16;
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}
```

#### resize()

https://segmentfault.com/a/1190000015812438?utm_source=tag-newest

第一次使用或当容器中容量达到threshold(length*load_factor)时自动扩容

```java
final Node<K,V>[] resize() {
        //oldTab 为当前表的哈希桶
        Node<K,V>[] oldTab = table;
        //当前哈希桶的容量 length
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        //当前的阈值
        int oldThr = threshold;
        //初始化新的容量和阈值为0
        int newCap, newThr = 0;
        //如果当前容量大于0
        if (oldCap > 0) {
            //如果当前容量已经到达上限 2^30
            if (oldCap >= MAXIMUM_CAPACITY) {
                //则设置阈值是2的31次方-1
                threshold = Integer.MAX_VALUE;
                //同时返回当前的哈希桶，不再扩容
                return oldTab;
            }//否则新的容量为旧的容量的两倍。 
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)//如果旧的容量大于等于默认初始容量16
                //那么新的阈值也等于旧的阈值的两倍
                newThr = oldThr << 1; // double threshold
        }//如果当前表是空的，但是有阈值。代表是初始化时指定了容量、阈值的情况
        else if (oldThr > 0) // initial capacity was placed in threshold
            newCap = oldThr;//那么新表的容量就等于旧的阈值
        else {//如果当前表是空的，而且也没有阈值。代表是初始化时没有任何容量/阈值参数的情况               // zero initial threshold signifies using defaults
            newCap = DEFAULT_INITIAL_CAPACITY;//此时新表的容量为默认的容量 16
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);//新的阈值为默认容量16 * 默认加载因子0.75f = 12
        }
        if (newThr == 0) {//如果新的阈值是0，对应的是  当前表是空的，但是有阈值的情况
            float ft = (float)newCap * loadFactor;//根据新表容量 和 加载因子 求出新的阈值
            //进行越界修复
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        //更新阈值 
        threshold = newThr;
        @SuppressWarnings({"rawtypes","unchecked"})
        //根据新的容量 构建新的哈希桶
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        //更新哈希桶引用
        table = newTab;
        //如果以前的哈希桶中有元素
        //下面开始将当前哈希桶中的所有节点转移到新的哈希桶中
        if (oldTab != null) {
            //遍历老的哈希桶
            for (int j = 0; j < oldCap; ++j) {
                //取出当前的节点 e
                Node<K,V> e;
                //如果当前桶中有元素,则将链表赋值给e
                if ((e = oldTab[j]) != null) {
                    //将原哈希桶置空以便GC
                    oldTab[j] = null;
                    //如果当前链表中就一个元素，（没有发生哈希碰撞）
                    if (e.next == null)
                        //直接将这个元素放置在新的哈希桶里。
                        //注意这里取下标 是用 哈希值 与 桶的长度-1 。 由于桶的长度是2的n次方，这么做其实是等于 一个模运算。但是效率更高
                        newTab[e.hash & (newCap - 1)] = e;
                        //如果发生过哈希碰撞 ,而且是节点数超过8个，转化成了红黑树（暂且不谈 避免过于复杂， 后续专门研究一下红黑树）
                    else if (e instanceof TreeNode)
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    //如果发生过哈希碰撞，节点数小于8个。则要根据链表上每个节点的哈希值，依次放入新哈希桶对应下标位置。
                    else { // preserve order
                        //因为扩容是容量翻倍，所以原链表上的每个节点，现在可能存放在原来的下标，即low位， 或者扩容后的下标，即high位。 high位=  low位+原哈希桶容量
                        //低位链表的头结点、尾节点
                        Node<K,V> loHead = null, loTail = null;
                        //高位链表的头节点、尾节点
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;//临时节点 存放e的下一个节点
                        do {
                            next = e.next;
                            //这里又是一个利用位运算 代替常规运算的高效点： 利用哈希值 与 旧的容量，可以得到哈希值去模后，是大于等于oldCap还是小于oldCap，等于0代表小于oldCap，应该存放在低位，否则存放在高位
                            if ((e.hash & oldCap) == 0) {
                                //给头尾节点指针赋值
                                if (loTail == null)
                                    loHead = e;
                                else
                                    loTail.next = e;
                                loTail = e;
                            }//高位也是相同的逻辑
                            else {
                                if (hiTail == null)
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;
                            }//循环直到链表结束
                        } while ((e = next) != null);
                        //将低位链表存放在原index处，
                        if (loTail != null) {
                            loTail.next = null;
                            newTab[j] = loHead;
                        }
                        //将高位链表存放在新index处
                        if (hiTail != null) {
                            hiTail.next = null;
                            newTab[j + oldCap] = hiHead;
                        }
                    }
                }
            }
        }
        return newTab;
    }
```

1. 旧数组容量>0，如果>最大容量，将阈值设为Integer.MAX_VALUE，然后返回，否则扩容2倍
2. 旧数组容量==0且旧阈值>0，表示构造函数使用了HashMap(int initialCapacity)和HashMap(int initialCapacity, float loadFactor)，导致旧数组为null，旧阈值为用户指定的容量，那么就把旧阈值赋给新容量，因为这两个构造函数是将容量赋值给了阈值`this.threshold = tableSizeFor(initialCapacity);`
3. 旧数组容量==0且旧阈值<=0，表示构造函数使用了HashMap()，所有值均是0，那么就把新数组容量设为默认初始容量16，阈值设为容量的0.75倍
4. 创建一个容量为newCap的新Node数组，如果旧数组为null，直接返回新数组
5. 遍历旧数组中不为null的桶，如果桶中只有一个节点，直接根据(newCap-1)&hash放到新数组中；如果是红黑树，则拆分数；如果是链表，如果hash&oldCap==0，将节点放入lo链表，否则放入hi链表。如果lo链表非空，就放到新数组中的当前索引j，同样将hi链表放到新数组的j+oldCap位置

**(e.hash & oldCap) == 0**

oldCap是2的幂，假设是2^m^，newCap是2^(m+1)^，桶定位是根据hash&(n-1)进行的，相当于取hash的低m位

假设oldCap=16，即m=4，hash&(n-1)相当于取低4位，记为abcd

扩容后，节点的新位置就变成了hash&(32-1)，相当于取hash的低5位，低5位的值只有0abcd和1abcd两种，如果是0abcd就和之前一样，是1abcd的话就相当于oldCap+0abcd，所以新旧位置的关系就等效于hash&oldCap

#### putVal

```java
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

```java
transient Node<K,V>[] table;

final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
    	//1、判断table是否为空，为空则使用resize()初始化
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
    	//2、根据当前key的hash值定位到具体的桶中并判断，为空则表示没有hash冲突，直接创建一个新桶
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else {
            Node<K,V> e; K k;
            //3、如果桶有值，那么比较当前桶的key、key的hashcode和写入的key是否相等，相等就赋值给e，后续更新值即可
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            //4、如果当前桶为红黑树，则按照红黑树方式操作
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                //5、如果是个链表，遍历
                for (int binCount = 0; ; ++binCount) {
                    //6、将下一个节点赋值给e 并判断是否遍历到最后一个节点
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        //链表长度大于等于8并且容量>=64才转换为红黑树
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    //7、比较下一个节点
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }
```

#### hash(key)

key的hash值是通过hashCode()的高16位异或低16位实现的

原来的hashCode：1111 1111 1111 1111 0100 1100 0000 1010

移位后的：		    0000 0000 0000 0000 1111 1111 1111 1111

进行异或运算：	  1111 1111 1111 1111 1011 0011 1111 0101

好处：高位和低位的信息混合，随机性增大

```java
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

#### 长度为什么是2的n次方

提高效率，定位的时候使用的是hash&(n-1)，实际是相当于hash%n，但是位运算比取余效率高，两者相等的前提是n必须是2的n次方

#### 多线程下问题

put操作时，会导致元素覆盖

### ConcurrentHashMap

#### sizeCtl

正常状态相当于HashMap中的threshold，当元素个数超过这个值就进行扩容；

等于-1，说明有一个线程正在执行初始化

#### initTable

1. 循环判断table是否为空，为空继续进行操作
2. 如果sizeCtl<0，表明有线程正在进行初始化操作，直接将当前线程变为就绪状态
3. 如果sizeCtl>=0，当前线程尝试通过CAS将sizeCtl的值修改为-1，修改失败就进入下轮循环
4. 如果修改成功，进行双重检查table是否为空，为空的话就进行初始化，并将sizeCtl修改为容量的0.75倍

```java
/**
 * Initializes table, using the size recorded in sizeCtl.
 */
private final Node<K,V>[] initTable() {
    Node<K,V>[] tab; int sc;
    // #1
    while ((tab = table) == null || tab.length == 0) {
        // sizeCtl的默认值是0，所以最先走到这的线程会进入到下面的else if判断中
        // #2
        if ((sc = sizeCtl) < 0)
            Thread.yield(); // lost initialization race; just spin
        // 尝试原子性的将指定对象(this)的内存偏移量为SIZECTL的int变量值从sc更新为-1
        // 也就是将成员变量sizeCtl的值改为-1
        // #3
        else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
            try {
                // 双重检查：如果thread1没执行到#5，恰巧thread2执行完#1然后失去CPU，thread1执行完#6之后thread2又继续执行，那么会thread2会进入到这里，如果没有双重检查，就会重复初始化
                // #4
                if ((tab = table) == null || tab.length == 0) {
                    int n = (sc > 0) ? sc : DEFAULT_CAPACITY; // 默认初始容量为16
                    @SuppressWarnings("unchecked")
                    Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                    // #5
                    table = tab = nt; // 创建hash表，并赋值给成员变量table
                    sc = n - (n >>> 2);
                }
            } finally {
                // #6
                sizeCtl = sc;
            }
            break;
        }
    }
    return tab;
}
```

#### put

采用CAS+synchronized

1. 如果key和value为null，抛出异常
2. 根据key的hashCode重新计算hash值，(h ^ (h >>> 16)) & 0x7fffffff
3. 开始循环，如果table为空，进行初始化
4. 根据(n - 1) & hash定位到桶，如果为空，利用CAS尝试写入，失败则自旋保证成功
5. 如果当前桶的hash值=MOVED=-1，则说明当前正在扩容，帮助一起扩容
6. 如果都不满足，利用synchronizeed对桶加锁。再次判断桶是否发生变化，如果没有，并且hash值小于0，表示是树，则将节点放入红黑树中；否则遍历桶，如果找到hash值和key都相等的节点，则根据标识符onlyIfAbsent进行更新。如果没有找到，在链表尾部插入新节点

```java
final V putVal(K key, V value, boolean onlyIfAbsent) {
    if (key == null || value == null) throw new NullPointerException();
    int hash = spread(key.hashCode());
    int binCount = 0;
    for (Node<K,V>[] tab = table;;) {
        Node<K,V> f; int n, i, fh;
        if (tab == null || (n = tab.length) == 0)
            tab = initTable();
        else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
            if (casTabAt(tab, i, null,
                         new Node<K,V>(hash, key, value, null)))
                break;                   // no lock when adding to empty bin
        }
        else if ((fh = f.hash) == MOVED)
            tab = helpTransfer(tab, f);
        else {
            V oldVal = null;
            synchronized (f) {
                    if (tabAt(tab, i) == f) {
                        if (fh >= 0) {
                            binCount = 1;
                            for (Node<K,V> e = f;; ++binCount) {
                                K ek;
                                if (e.hash == hash &&
                                    ((ek = e.key) == key ||
                                     (ek != null && key.equals(ek)))) {
                                    oldVal = e.val;
                                    if (!onlyIfAbsent)
                                        e.val = value;
                                    break;
                                }
                                Node<K,V> pred = e;
                                if ((e = e.next) == null) {
                                    pred.next = new Node<K,V>(hash, key,
                                                              value, null);
                                    break;
                                }
                            }
                        }
                        else if (f instanceof TreeBin) {
                            Node<K,V> p;
                            binCount = 2;
                            if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                           value)) != null) {
                                oldVal = p.val;
                                if (!onlyIfAbsent)
                                    p.val = value;
                            }
                        }
                    }
                }
                if (binCount != 0) {
                    if (binCount >= TREEIFY_THRESHOLD)
                        treeifyBin(tab, i);
                    if (oldVal != null)
                        return oldVal;
                    break;
                }
        }
    }
    addCount(1L, binCount);
    return null;
}
```

#### 扩容

扩容的时机

1. 当前容量超过阈值
2. 链表元素超过默认值(8个)，但数组长度<64
3. 发现其他线程扩容时，帮助扩容

**transferIndex**

```java
private transient volatile int transferIndex;
/**
扩容线程每次最少要迁移16个hash桶
*/
private static final int MIN_TRANSFER_STRIDE = 16;
```

扩容索引，表示table数组中已经分配给扩容线程的索引位置，协调多个线程并发安全的获取桶

**扩容过程**

第一个扩容线程将transferIndex设为n，并创建2倍容量的nextTable。

开始循环，`advance`表示是否需要继续操作，通过CAS修改`transferIndex=transferIndex-stride`，记bound=transferIndex为左边界，`i`为右边界

![img](java%E9%9D%A2%E8%AF%95%E9%A2%98%E6%80%BB%E7%BB%93/6283837-7e10aa6066673c79.png)

~~~java
private final void transfer(Node<K,V>[] tab, Node<K,V>[] nextTab) {
    int n = tab.length, stride;
    //一个线程迁移节点的个数 最小为16
    if ((stride = (NCPU > 1) ? (n >>> 3) / NCPU : n) < MIN_TRANSFER_STRIDE)
        stride = MIN_TRANSFER_STRIDE; // subdivide range
    //第一个线程进来先扩容
    if (nextTab == null) {            
        try {
            @SuppressWarnings("unchecked")
            Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n << 1];
            nextTab = nt;
        } catch (Throwable ex) {      // try to cope with OOME
            sizeCtl = Integer.MAX_VALUE;
            return;
        }
        nextTable = nextTab;
        //从最右开始迁移
        transferIndex = n;
    }
    int nextn = nextTab.length;
    //用来标记桶节点处理完毕
    ForwardingNode<K,V> fwd = new ForwardingNode<K,V>(nextTab);
    boolean advance = true;
    boolean finishing = false; // to ensure sweep before committing nextTab
    //i表示当前处理的桶索引，bound表示需要处理的左边界
    for (int i = 0, bound = 0;;) {
        Node<K,V> f; int fh;
        while (advance) {
            //
            int nextIndex, nextBound;
            if (--i >= bound || finishing)
                advance = false;
            else if ((nextIndex = transferIndex) <= 0) {
                i = -1;
                advance = false;
            }
            else if (U.compareAndSwapInt
                     (this, TRANSFERINDEX, nextIndex,
                      nextBound = (nextIndex > stride ?
                                   nextIndex - stride : 0))) {
                bound = nextBound;
                i = nextIndex - 1;
                advance = false;
            }
        }
        //接下面代码
        ```
    }
}
~~~

开始从右往左开始迁移。如果桶为空，写入ForwardingNode节点用于标记，如果已经处理过则跳过，如果没有处理过则进行迁移。

如果是链表，迁移过程类似HashMap，根据（hash&n）来决定在新table的位置。不同的是这里先遍历链表根据（hash&n）找到最后一个分割点，如图所示，如果该值为1，用hn指向，否则用ln指向。

![image-20200629150648865](https://tva1.sinaimg.cn/large/008i3skNly1gptskuekauj306e03nwee.jpg)

然后从头部开始，根据节点的信息新建节点，并保存到`ln`或`hn`链表的头部。最后迁移两个链表，并将当前table设置为ForwardingNode

```java
//判断迁移是否结束
if (i < 0 || i >= n || i + n >= nextn) {
    int sc;
    if (finishing) {
        nextTable = null;
        table = nextTab;
        sizeCtl = (n << 1) - (n >>> 1);
        return;
    }
    if (U.compareAndSwapInt(this, SIZECTL, sc = sizeCtl, sc - 1)) {
        if ((sc - 2) != resizeStamp(n) << RESIZE_STAMP_SHIFT)
            return;
        finishing = advance = true;
        i = n; // recheck before commit
    }
}
//当前桶为空
else if ((f = tabAt(tab, i)) == null)
    advance = casTabAt(tab, i, null, fwd);
//桶已经处理
else if ((fh = f.hash) == MOVED)
    advance = true; // already processed
else {
    //迁移
    synchronized (f) {
        if (tabAt(tab, i) == f) {
            Node<K,V> ln, hn;
            if (fh >= 0) {
                int runBit = fh & n;
                Node<K,V> lastRun = f;
                //找到最后一个分割点
                for (Node<K,V> p = f.next; p != null; p = p.next) {
                    int b = p.hash & n;
                    if (b != runBit) {
                        runBit = b;
                        lastRun = p;
                    }
                }
                if (runBit == 0) {
                    ln = lastRun;
                    hn = null;
                }
                else {
                    hn = lastRun;
                    ln = null;
                }
                //将所有的节点分成两个链表
                for (Node<K,V> p = f; p != lastRun; p = p.next) {
                    int ph = p.hash; K pk = p.key; V pv = p.val;
                    if ((ph & n) == 0)
                        ln = new Node<K,V>(ph, pk, pv, ln);
                    else
                        hn = new Node<K,V>(ph, pk, pv, hn);
                }
                //迁移两个链表
                setTabAt(nextTab, i, ln);
                setTabAt(nextTab, i + n, hn);
                setTabAt(tab, i, fwd);
                advance = true;
            }
            else if (f instanceof TreeBin) {
                TreeBin<K,V> t = (TreeBin<K,V>)f;
                TreeNode<K,V> lo = null, loTail = null;
                TreeNode<K,V> hi = null, hiTail = null;
                int lc = 0, hc = 0;
                for (Node<K,V> e = t.first; e != null; e = e.next) {
                    int h = e.hash;
                    TreeNode<K,V> p = new TreeNode<K,V>
                        (h, e.key, e.val, null, null);
                    if ((h & n) == 0) {
                        if ((p.prev = loTail) == null)
                            lo = p;
                        else
                            loTail.next = p;
                        loTail = p;
                        ++lc;
                    }
                    else {
                        if ((p.prev = hiTail) == null)
                            hi = p;
                        else
                            hiTail.next = p;
                        hiTail = p;
                        ++hc;
                    }
                }
                ln = (lc <= UNTREEIFY_THRESHOLD) ? untreeify(lo) :
                    (hc != 0) ? new TreeBin<K,V>(lo) : t;
                hn = (hc <= UNTREEIFY_THRESHOLD) ? untreeify(hi) :
                    (lc != 0) ? new TreeBin<K,V>(hi) : t;
                setTabAt(nextTab, i, ln);
                setTabAt(nextTab, i + n, hn);
                setTabAt(tab, i, fwd);
                advance = true;
            }
        }
    }
}
```

# 设计模式

### 六大原则

- 开闭：软件实体应该可以扩展，但不可以修改，对扩展开放，对修改关闭
- 单一职责：一个类一个方法只负责一件事
- 里式替换：基类可以替换为子类
- 依赖倒置：高层模块不应该依赖于底层模块，两者应该依赖于其抽象
- 接口隔离：使用多个隔离的接口，比使用单个接口好
- 迪米特：一个类尽量减少对其他对象的依赖

### 单例模式

- 避免频繁创建销毁对象，减少资源消耗
- 避免对共享资源的多重占用
- 扩展困难
- 违背单一职责

1. 懒汉式，需要的时候才创建，缺点：需要进行同步处理

   ```java
   public class Singleton {
       private static volatile Singleton instance = null;
       //构造器私有化
       private Singleton(){}
       public static Singleton getInstance() {
           if (instance == null) { // Single Checked
               synchronized (Singleton.class) {
                   if (instance == null) { // Double checked
                       instance = new Singleton();
                   }
               }
           }
           return instance;
       }
   }
   ```

2. 饿汉式，类加载的时候就会创建，缺点：如果没有使用就会造成内存浪费

   ```java
   public class Singleton{
       private Singleton(){}
       private static final Singleton instance = new Singleton();
       public static Singleton newInstance(){
           return instance;
       }
   }
   ```

3. 静态内部类

   ```java
   public class Singleton{
       private static class SingletonHolder{
           private static final Singleton instance=new Singleton();
       }
       private Singleton(){}
   	public static Singleton getInstance(){
           return SingletonHolder.instance;
       }
   }
   ```
   
4. 枚举

   ```java
   public class Singleton {
       private Singleton() {
       }
   
       private enum Inner {
           INSTANCE;
           private final Singleton singleton;
   
           Inner() {
               singleton = new Singleton();
           }
   
           public Singleton getSingleton() {
               return singleton;
           }
       }
   
       public static Singleton getInstance() {
           return Inner.INSTANCE.getSingleton();
       }
   }
   ```

### 工厂模式

### 代理模式

静态代理、动态代理

### 装饰者模式

动态地将责任附加到对象上。InputStream和OutputStream都是装饰者模式

几个角色：

- Component：抽象类或接口
- ConcreteComponent：是Component的实现类
- Decorator：抽象类，继承或实现了Component，同时持有一个Component对象的引用
- ConcreteDecorator：具体的装饰者对象，负责添加具体的责任，是Decorator的实现类，

![image-20200629195505581](https://tva1.sinaimg.cn/large/008i3skNly1gptskvdczrj311n0lzgql.jpg)

### 建造者模式



### 适配器模式

可以让任何两个没有关联的类一起运行，提高了类的复用，增加了类的透明度，灵活性好。分为类适配器和对象适配器

过多地使用适配器，会让系统非常零乱。 JAVA 至多继承一个类，所以至多只能适配一个适配者类，而且目标类必须是抽象类。

**类适配器**：通过继承连接两个接口，是静态的定义方式

```java
//接口A
public interface Player{
      void action();
}

//接口B 要被适配的
public interface MP4{
    void play();
}
//实现类
public class ExpensiveMP4 implement MP4{
    public void play(){
            // todo
    }
}
```

Player中action()要写的操作就是ExpensiveMP4的play，所以就没必要再重写action

```java
public class ExpensiveAdapter extends ExpensiveMP4 implement Player{
    public void action(){
        play();
    }
}
```

类适配器不灵活，而且在java中少用继承，多用组合

**对象适配器**：使用组合的方式，可以把源类和其子类都适配到目标接口

```java
public class PlayerAdapter implement Player{
    public MP4 mp4;
    
    public PlayerAdapter (MP4 mp4){
        this.mp4 = mp4;
    }     

    public void action(){
        if(mp4!= null){
             mp4.play();
        }
    }
}
```

