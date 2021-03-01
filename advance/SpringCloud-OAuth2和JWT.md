---
title: SpringCloud：OAuth2和JWT
tags: SpringCloud
date: 2019-10-07 18:12:51
---

搭建一个oauth2服务器，包括认证、授权和资源服务器

项目地址：https://github.com/zheyday/SpringCloudStudy

oauth分支



**参考资料**：

https://www.cnblogs.com/fp2952/p/8973613.html

https://juejin.im/post/5c5ae6566fb9a049b3486e38

[Spring OAuth2官方文档](https://projects.spring.io/spring-security-oauth/docs/oauth2.html) 



本文分为两个部分

- 第一部分比较简单，将客户端信息和用户信息固定在程序里，令牌存储在内存中
- 第二部分从数据库读取用户信息，使用jwt生成令牌

## 一、简化版

使用Spring Initializr新建项目，勾选如下三个选项

{%asset_img  1570420518382.png %}

![1570420518382](SpringCloud-OAuth2和JWT/1570420518382.png)

pom.xml

```xml
 <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
     //只需要引用这一个 
     //集成了spring-security-oauth2 spring-security-jwt spring-security-oauth2-autoconfigure
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-oauth2</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
```



### 配置Spring Security

新建类WebSecurityConfig 继承 WebSecurityConfigurerAdapter，并添加@Configuration @EnableWebSecurity注解，重写三个方法，代码如下，详细讲解在代码下面

```java
@Configuration
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
    @Autowired
    private UserServiceDetail userServiceDetail;

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(userServiceDetail).passwordEncoder(passwordEncoder());
        //内存存储
//        auth
//                .inMemoryAuthentication()
//                .passwordEncoder(passwordEncoder())
//                .withUser("user")
//                .password(passwordEncoder().encode("user"))
//                .roles("USER");

    }
    //忽略url
	@Override
    public void configure(WebSecurity web) throws Exception {
        web.ignoring().antMatchers("/js/**","/fonts/**","/css/**","/login.html");
    }

    /**
     * 配置了默认表单登陆以及禁用了 csrf 功能，并开启了httpBasic 认证
     *
     * @param http
     * @throws Exception
     */
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http    // 配置登陆页/login并允许访问
                .formLogin().permitAll()
                // 登出页
                .and().logout().logoutUrl("/logout").logoutSuccessUrl("/")
                // 其余所有请求全部需要鉴权认证
                .and().authorizeRequests().anyRequest().authenticated()
                // 由于使用的是JWT，我们这里不需要csrf
                .and().csrf().disable();
    }

    @Override
    @Bean
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();
    }
}
```

主要讲解一下

```java
protected void configure(AuthenticationManagerBuilder auth) throws Exception
```

这个方法是用来验证用户信息的。将用户名和密码与数据库匹配，如果有这个用户才能认证成功。我们注入了一个`UserServiceDetail`，这个service的功能就是验证用户的。`.passwordEncoder(passwordEncoder())`是使用加盐解密。

**UserServiceDetail**

实现了`UserDetailsService`接口，所以需要实现唯一的方法

```java
package zcs.oauthserver.service;

import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.stereotype.Service;
import zcs.oauthserver.model.UserModel;

import java.util.ArrayList;
import java.util.List;

@Service
public class UserServiceDetail implements UserDetailsService {
    @Override
    public UserDetails loadUserByUsername(String s) throws UsernameNotFoundException {
        List<SimpleGrantedAuthority> authorities = new ArrayList<>();
        authorities.add(new SimpleGrantedAuthority("ROLE"));
        return new UserModel("user","user",authorities);
    }
}
```

<font color='red'>这里先用假参数实现功能，后面添加数据库</font>

参数s是前端输入的用户名，通过该参数查找数据库，获取密码和角色权限，最后将这三个数据封装到`UserDetails`接口的实现类中返回。这里封装的类可以使用`org.springframework.security.core.userdetails.User`或者自己实现`UserDetails`接口。

**UserModel**

实现`UserDetails`接口

```java
package zcs.oauthserver.model;

import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;

import java.util.Collection;
import java.util.List;

public class UserModel implements UserDetails {
    private String userName;

    private String password;

    private List<SimpleGrantedAuthority> authorities;

    public UserModel(String userName, String password, List<SimpleGrantedAuthority> authorities) {
        this.userName = userName;
        this.password = new BCryptPasswordEncoder().encode(password);;
        this.authorities = authorities;
    }

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return authorities;
    }

    @Override
    public String getPassword() {
        return password;
    }

    @Override
    public String getUsername() {
        return userName;
    }

    @Override
    public boolean isAccountNonExpired() {
        return true;
    }

    @Override
    public boolean isAccountNonLocked() {
        return true;
    }

    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }

    @Override
    public boolean isEnabled() {
        return true;
    }
}

```

新增username、password和authorities，最后一个存储的是该用户的权限列表，也就是用户拥有能够访问哪些资源的权限。**密码加盐处理**。

### 配置Oauth2认证服务器

新建配置类AuthorizationServerConfig 继承 AuthorizationServerConfigurerAdapter，并添加@Configuration
@EnableAuthorizationServer注解表明是一个认证服务器

重写三个函数

- `ClientDetailsServiceConfigurer`：用来配置客户端详情服务，客户端详情信息在这里进行初始化，你能够把客户端详情信息写死在这里或者是通过数据库来存储调取详情信息。客户端就是指第三方应用
- `AuthorizationServerSecurityConfigurer`：用来配置令牌端点(Token Endpoint)的安全约束.
- `AuthorizationServerEndpointsConfigurer`：用来配置授权（authorization）以及令牌（token）的访问端点和令牌服务(token services)。

```java
@Configuration
@EnableAuthorizationServer
public class AuthorizationServerConfig extends AuthorizationServerConfigurerAdapter {
	//从WebSecurityConfig加载
    @Autowired
    private AuthenticationManager authenticationManager;
    //内存存储令牌
    private TokenStore tokenStore = new InMemoryTokenStore();

    /**
     * 配置客户端详细信息
     *
     * @param clients
     * @throws Exception
     */
    @Override
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
        clients.inMemory()
            	//客户端ID
                .withClient("zcs")
                .secret(new BCryptPasswordEncoder().encode("zcs"))
                //权限范围
                .scopes("app")
            	//授权码模式
                .authorizedGrantTypes("authorization_code")
                //随便写
                .redirectUris("www.baidu.com");
//        clients.withClientDetails(new JdbcClientDetailsService(dataSource));
    }

    @Override
    public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
        endpoints.tokenStore(tokenStore)
                .authenticationManager(authenticationManager);
    }

    /**
     * 在令牌端点定义安全约束
     * 允许表单验证，浏览器直接发送post请求即可获取tocken
     * 这部分写这样就行
     * @param security
     * @throws Exception
     */
    @Override
    public void configure(AuthorizationServerSecurityConfigurer security) throws Exception {
        security
                // 开启/oauth/token_key验证端口无权限访问
                .tokenKeyAccess("permitAll()")
                // 开启/oauth/check_token验证端口认证权限访问
                .checkTokenAccess("isAuthenticated()")
                .allowFormAuthenticationForClients();
    }
}

```

客户端详细信息同样也是测试用，后续会加上数据库。令牌服务暂时是用内存存储，后续加上jwt。

先实现功能最重要，复杂的东西一步步往上加。

### 配置资源服务器

资源服务器也就是服务程序，是需要访问的服务器

新建ResourceServerConfig继承ResourceServerConfigurerAdapter

```java
@Configuration
@EnableResourceServer
public class ResourceServerConfig extends ResourceServerConfigurerAdapter {
    @Override
    public void configure(HttpSecurity http) throws Exception {
        http
//                antMatcher表示只能处理/user的请求
                .antMatcher("/user/**")
                .authorizeRequests()
                .antMatchers("/user/test1").permitAll()
                .antMatchers("/user/test2").authenticated()
//                .antMatchers("user/test2").hasRole("USER")
//                .anyRequest().authenticated()
        ;
    }
}
```

`ResourceServerConfigurerAdapter`的Order默认值是3，小于`WebSecurityConfigurerAdapter`，值越小优先级越大

关于`ResourceServerConfigurerAdapter`和`WebSecurityConfigurerAdapter`的详细说明见

https://www.jianshu.com/p/fe1194ca8ecd

新建UserController

```java
@RestController
public class UserController {
    @GetMapping("/user/me")
    public Principal user(Principal principal) {
        return principal;
    }

    @GetMapping("/user/test1")
    public String test() {
        return "test1";
    }

    @GetMapping("/user/test2")
    public String test2() {
        return "test2";
    }

}
```



### 测试

1. 获取code
   浏览器访问`http://127.0.0.1:9120/oauth/authorize?client_id=zcs&response_type=code&redirect_uri=www.baidu.com`，然后跳出登陆页面，
   ![1570432456538](SpringCloud-OAuth2和JWT/1570432456538.png)

登陆

认证

![1570442575560](SpringCloud-OAuth2和JWT/1570442575560.png)

地址栏会出现回调页面，并且带有code参数  `http://127.0.0.1:9120/oauth/www.baidu.com?code=FGQ1jg`

2. 获取token
   postman访问`http://127.0.0.1:9120/oauth/token?code=FGQ1jg&grant_type=authorization_code&redirect_uri=www.baidu.com&client_id=zcs&client_secret=zcs`，code填写刚才得到的code，使用POST请求

![1570442089823](SpringCloud-OAuth2和JWT/1570442089823.png)

3. 访问资源
   /user/test2是受保护资源，我们通过令牌访问
   ![1570442119823](SpringCloud-OAuth2和JWT/1570442119823.png)

## 二、升级版

### JWT

有很多人会把JWT和OAuth2来作比较，其实它俩是完全不同的概念，没有可比性。

JWT是一种认证协议，提供一种用于发布接入令牌、并对发布的签名接入令牌进行验证的方法。

OAuth2是一种授权框架，提供一套详细的授权机制。

Spring Cloud OAuth2集成了JWT作为令牌管理，因此使用起来很方便

`JwtAccessTokenConverter`是用来生成token的转换器，而token令牌默认是有签名的，且资源服务器需要验证这个签名。此处的加密及验签包括两种方式：
对称加密、非对称加密（公钥密钥）
对称加密需要授权服务器和资源服务器存储同一key值，而非对称加密可使用密钥加密，暴露公钥给资源服务器验签，本文中使用非对称加密方式。

通过jdk工具生成jks证书，通过cmd进入jdk安装目录的bin下，运行命令

`keytool -genkeypair -alias oauth2-keyalg RSA -keypass mypass -keystore oauth2.jks -storepass mypass`

会在当前目录生成oauth2.jks文件，放入resource目录下。

maven默认不加载resource目录下的文件，所以需要在pom.xml中配置，在build下添加配置

```xml
	  <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
        <resources>
            <resource>
                <directory>src/main/resources</directory>
                <includes>
                    <include>**/*.*</include>
                </includes>
            </resource>
        </resources>
    </build>
```

在原来的AuthorizationServerConfig中更改部分代码

```java
	@Autowired
    private TokenStore tokenStore;	

	@Override
    public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
//        endpoints.tokenStore(tokenStore)
//                .authenticationManager(authenticationManager);
        endpoints.authenticationManager(authenticationManager)
                .accessTokenConverter(jwtAccessTokenConverter())
                .tokenStore(tokenStore);
    }

    @Bean
    public TokenStore tokenStore() {
        return new JwtTokenStore(jwtAccessTokenConverter());
    }

    /**
     * 非对称加密算法对token进行签名
     * @return
     */
    @Bean
    public JwtAccessTokenConverter jwtAccessTokenConverter() {
        final JwtAccessTokenConverter converter = new CustomJwtAccessTokenConverter();
        // 导入证书
        KeyStoreKeyFactory keyStoreKeyFactory =
                new KeyStoreKeyFactory(new ClassPathResource("oauth2.jks"), "mypass".toCharArray());
        converter.setKeyPair(keyStoreKeyFactory.getKeyPair("oauth2"));
        return converter;
    }
```

`jwtAccessTokenConverter`方法中有一个`CustomJwtAccessTokenConverter`类，这是继承了`JwtAccessTokenConverter`，自定义添加了额外的token信息

```java
/**
 * 自定义添加额外token信息
 */
public class CustomJwtAccessTokenConverter extends JwtAccessTokenConverter {
    @Override
    public OAuth2AccessToken enhance(OAuth2AccessToken accessToken, OAuth2Authentication authentication) {
        DefaultOAuth2AccessToken defaultOAuth2AccessToken = new DefaultOAuth2AccessToken(accessToken);
        Map<String, Object> additionalInfo = new HashMap<>();
        UserModel user = (UserModel)authentication.getPrincipal();
        additionalInfo.put("USER",user);
        defaultOAuth2AccessToken.setAdditionalInformation(additionalInfo);
        return super.enhance(defaultOAuth2AccessToken,authentication);
    }
}
```

### Security

之前登陆是用假数据，现在通过连接数据库进行验证。

建立三个表，user存储用户账号和密码，role存储角色，user_role存储用户的角色

{%asset_img 1570598886447.png%}



user表

{%asset_img 1570598990454.png%}

role表

{%asset_img 1570598945536.png%}

user_role表

{%asset_img 1570598962212.png%}

使用mybatis-plus生成代码，改造之前的`UserServiceDetail`和`UserModel`

{%asset_img 1570598325161.png%}



UserServiceDetail

```java
@Service
public class UserServiceDetail implements UserDetailsService {
    private final UserMapper userMapper;
    private final RoleMapper roleMapper;

    @Autowired
    public UserServiceDetail(UserMapper userMapper, RoleMapper roleMapper) {
        this.userMapper = userMapper;
        this.roleMapper = roleMapper;
    }

    @Override
    public UserDetails loadUserByUsername(String s) throws UsernameNotFoundException {
        QueryWrapper<User> userQueryWrapper = new QueryWrapper<>();
        userQueryWrapper.eq("username", s);
        User user = userMapper.selectOne(userQueryWrapper);
        if (user == null) {
            throw new RuntimeException("用户名或密码错误");
        }

        user.setAuthorities(roleMapper.selectByUserId(user.getId()));
        return user;
    }
}
```

通过UserMapper查询用户信息，然后封装到User中，在自动生成的User上实现UserDetails接口

User

```java
public class User implements Serializable, UserDetails {

    private static final long serialVersionUID = 1L;

    @TableId(value = "id", type = IdType.AUTO)
    private Integer id;

    @TableId(value = "username")
    private String username;

    @TableId(value = "password")
    private String password;

    @TableField(exist = false)
    private List<Role> authorities;

    public User() {
    }

    public Integer getId() {
        return id;
    }

    public String getUsername() {
        return username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = new BCryptPasswordEncoder().encode(password);
    }

    public void setAuthorities(List<Role> authorities) {
        this.authorities = authorities;
    }

    @Override
    public boolean isAccountNonExpired() {
        return true;
    }

    @Override
    public boolean isAccountNonLocked() {
        return true;
    }

    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }

    @Override
    public boolean isEnabled() {
        return true;
    }

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return authorities;
    }


    @Override
    public String toString() {
        return "User{" +
                "id=" + id +
                ", username=" + username +
                ", password=" + password +
                "}";
    }
}

```

**解释说明：**

UserDetails中需要重写一个方法，是存储用户权限的

```java
@Override
public Collection<? extends GrantedAuthority> getAuthorities()
```

所以新增了一个变量，并且打上注解表示这不是一个字段属性

```java
@TableField(exist = false)
private List<Role> authorities;
```

在Role上实现GrantedAuthority接口，只需要权限名称就可以了

```java
public class Role implements Serializable, GrantedAuthority {

    private static final long serialVersionUID = 1L;

    private String name;

    @Override
    public String toString() {
        return name;
    }

    @Override
    public String getAuthority() {
        return name;
    }
}
```

在RoleMapper.java中新增方法，通过用户id查询拥有的角色

```java
   @Select("select name from role r INNER JOIN user_role ur on ur.user_id=1 and ur.role_id=r.id")
    List<Role> selectByUserId(Integer id);
```



### 测试

测试方法和第一部分一样，获取令牌的时候返回如下

{%asset_img 1570459537790.png %}

![1570459537790](SpringCloud-OAuth2和JWT/1570459537790.png)


