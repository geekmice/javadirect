

## 何为springboot ##

主要简化了使用spring的难度，简省看繁重的配置，提供了各种启动器，开发者能快速上手。



## springboot有哪些优点 ##

1. **容易上手，提升开发效率**，为 Spring 开发提供一个更快、更广泛的入门体验。
2. 开箱即用，远离繁琐的配置。
3. 提供了一系列**大型项目通用的非业务性功能**，例如：内嵌服务器、安全管理、运行数据监控、运行状况检查和外部化配置等。
4. 没有代码生成，也不需要XML配置。
5. 避免大量的 Maven 导入和各种版本冲突。



## springboot的核心注解有哪些? 它主要有哪些注解组成的（重点） ##

启动类上面的注解是**@SpringBootApplication**，它也是 Spring Boot 的核心注解，主要组合包含了以下 3 个注解：

@SpringBootConfiguration：组合了 @Configuration 注解，实现配置文件的功能。

@EnableAutoConfiguration：打开自动配置的功能，也可以关闭某个自动配置的选项，如关闭数据源自动配置功能： @SpringBootApplication(exclude = { DataSourceAutoConfiguration.class })。

@ComponentScan：Spring组件扫描。

2、@MapperScan: spring-boot支持mybatis组件的一个注解，通过此注解指定mybatis接口类的路径，即可完成对mybatis接口的扫描。



3、@ImportResource @Import @PropertySource 这三个注解都是用来导入自定义的一些配置文件

**@ImportResource(locations={}) 导入其他xml配置文件，需要标准在主配置类上。**

**导入property的配置文件 @PropertySource指定文件路径，这个相当于使用spring的<importresource/>标签来完成配置项的引入。**

**@import注解是一个可以将普通类导入到spring容器中做管理**



4、控制层controller

rest风格：@GetMapping,@PostMapping,@PutMapping,@DeleteMapping

@RestController: @Controller 和@ResponseBody的结合，一个类被加上@RestController 注解，数据接口中就不再需要添加@ResponseBody。更加简洁。



5、@Transactional： 通过这个注解可以声明事务，可以添加在类上或者方法上。

6、其他常用注解

- **@ControllerAdvice 和 @RestControllerAdvice：****通常和@ExceptionHandler、@InitBinder、@ModelAttribute一起配合使用。
- @ControllerAdvice 和 @ExceptionHandler 配合完成统一异常拦截处理。
- @RestControllerAdvice 是 @ControllerAdvice 和 @ResponseBody的合集，可以将异常以json的格式返回数据。



## springboot的自动装配原理是什么 ##

​	简单来说自动装配机制就是通过@EnableAutoConfiguration将配置为@Configuration类下面的@Bean方法加载到Spring容器中，其实spring boot自动装配机制就是为了满足其他插件进行扩展，由于外部很多bean我们无法管理，也不知道具体包路径，这时候使用自动装配机制可以让外部的类能够注入到Spring项目中，其次，Spring boot自动装配也就是Spring的自动装配，由于Spring里ImportSelector动态bean的装载机制实现了自动装载，同时使用了Meta-inf/spring.factories中spi机制实现了Spring自动扫描到自动装载的bean机制。

**参考下面理解:**

​	总而言之，Spring发展就是由xml文件到注解方式的一个循序渐进过程，比如@Component以及它的派生注解@Controller等。最后Spring直接将xml转换为@Configuration交给Spring容器来管理。这样一来，Springboot自动装载机制就是将外部xml文件中bean配置导入到自己项目中，让bean在自己的项目中运行，@EnableAutoConfiguration起中介作用。



## Springboot是否可以使用xml配置 ##

Spring Boot 推荐使用 Java 配置而非 XML 配置，但是 Spring Boot 中也可以使用 XML 配置，通过 @ImportResource 注解可以引入一个 XML 配置。



## springboot的核心配置文件是什么，Bootstrap.properties和application.proeprties有什么区别 ##

单纯做 Spring Boot 开发，可能不太容易遇到 bootstrap.properties 配置文件，但是在结合 Spring Cloud 时，这个配置就会经常遇到了，特别是在需要**加载一些远程配置文件**的时侯。

spring boot 核心的两个配置文件：

- bootstrap (. yml 或者 . properties)：boostrap 由父 ApplicationContext 加载的，比 applicaton 优先加载，配置在应用程序上下文的引导阶段生效。一般来说我们在 Spring Cloud Config 或者 Nacos 中会用到它。且 boostrap 里面的属性不能被覆盖；

- application (. yml 或者 . properties)： 由ApplicatonContext 加载，用于 spring boot 项目的自动化配置。

  

## 什么是spring profiles ##

​	Spring Profiles 允许用户根据配置文件（dev，test，prod 等）来注册 bean。因此，当应用程序在开发中运行时，只有某些 bean 可以加载，而在 PRODUCTION中，某些其他 bean 可以加载。假设我们的要求是 Swagger 文档仅适用于 QA 环境，并且禁用所有其他文档。这可以使用配置文件来完成。Spring Boot 使得使用配置文件非常简单。



## 比较一下spring security和shiro各自的优缺点 ##

1. Spring Security 是一个重量级的安全管理框架；Shiro 则是一个轻量级的安全管理框架
2. Spring Security 概念复杂，配置繁琐；Shiro 概念简单、配置简单
3. Spring Security 功能强大；Shiro 功能简单



## springboot跨域问题 ##

注解@CrossOrigin

gateway网关

nginx请求转发

jsonp前端



## spring-boot-starter-parent有什么用 ##

1. 定义了 Java 编译版本为 1.8 。
2. 使用 UTF-8 格式编码。
3. 继承自 spring-boot-dependencies，这个里边定义了依赖的版本，也正是因为继承了这个依赖，所以我们在写依赖时才不需要写版本号。
4. 执行打包操作的配置。
5. 自动化的资源过滤。
6. 自动化的插件配置。
7. 针对 application.properties 和 application.yml 的资源过滤，包括通过 profile 定义的不同环境的配置文件，例如 application-dev.properties 和 application-dev.yml。



## springboot打成的jar和普通的jar有什么区别 ##

Spring Boot 项目最终打包成的 jar 是可执行 jar ，这种 jar 可以直接通过 `java -jar xxx.jar` 命令来运行，这种 jar 不可以作为普通的 jar 被其他项目依赖，即使依赖了也无法使用其中的类。



## 运行springboot有几种方式（重点） ##

1）打包用命令或者放到容器中运行-打包jar放在linux运行

2）用 Maven/ Gradle 插件运行

3）直接执行 main 方法运行



## springboot需要独立的容器运行吗 ##

内置tomcat,jetty



## 14、如何使用springboot实现异常处理 ##

它提供了一个 **@ControllerAdvice**注解以及 **@ExceptionHandle**r注解，前者是用来**开启全局的异常捕获**，后者则是说明**捕获哪些异常**，对那些异常进行处理。



## 微服务中如何实现session共享《待定》 ##

1、redis 集中管理session(常用的方式)

redis : 对于key-》生成唯一的随机值id, 对于value->用户数据

cookie : 把redis生成key的值存放在cookie里面

访问项目其他模块，发送请求带着cookie进行发送，获取cookie数值，拿着cookie进行各种操作，

获取cookie之后，去redis里面进行查询，根据key进行查询，如果查询就是登录。



**2.cookie同步session 如JWT(json web token)**

token:按照一定规则生成字符串包含用户信息



这种完全把客户的登陆信息保存在客户端的cookie中，每次请求带着cookie中的Token
优点：由于完全舍弃了session 会减轻服务器端的压力
缺点：是把信息暴露在外，就算有加密算法还是存在安全问题。禁止使用cookie的情况下无效。



## springboot如何实现定时任务 ##

方案一：使用内置注解@Scheduled 

方案二：使用第三方框架Quartz



## springboot启动原理《追源码》 ##

https://www.jianshu.com/p/ef6f0c0de38f

1. 通过 SpringFactoriesLoader加载 META-INF/spring.factories⽂件，获取并创建 SpringApplicationRunListener对象 ,应用开始启动了。

2. 然后由 SpringApplicationRunListener来发出 starting 消息 
3. 创建参数，并配置当前 SpringBoot 应⽤将要使⽤的 Environment 
4. 完成之后，依然由 SpringApplicationRunListener来发出 environmentPrepared 消息 
5. 创建 ApplicationContext 
6. 初始化 ApplicationContext，并设置 Environment，加载相关配置等 
7. 由 SpringApplicationRunListener来发出 contextPrepared消息，告知SpringBoot 应⽤使⽤的 ApplicationContext已准备OK 
8. 将各种 beans 装载⼊ ApplicationContext，继续由 SpringApplicationRunListener来发出 contextLoaded 消息，告 知 SpringBoot 应⽤使⽤的 ApplicationContext已装填OK 
9. refresh ApplicationContext，完成IoC容器可⽤的最后⼀步 
10. 由 SpringApplicationRunListener来发出 started 消息 
11. 完成最终的程序的启动 
12. 由 SpringApplicationRunListener来发出 running 消息，告知程序已运⾏起来了



## 实现springboot接口等幂性校验 ##

### 唯一索引 ###

给表加唯一索引，此方法最简单，当数据重复插入时，直接报SQL异常，对应用影响不大；

alter table 表名 add unique(字段)

示例，两个字段为唯一索引，如果出现完全一样的 order_name， create_time 就直接重复报异常；

```
alter table `order`  add unique(order_name,create_time)
```



### 锁 ###

分布式锁也可以实现接口等幂次校验，知识追寻者有写过一篇使用redis实现分布式锁思路的一篇文件，小伙伴们可以参考下《为什么你不会redis分布式锁？因为你没看到这篇文章》

使用乐观锁（基于版本号实现），或者 悲观锁（表锁或者行锁）实现；



### 先查询后判断 ###

入库时先查询是否有该数据，无插入，否则不插入；



### token 机制 ###

​	token 机制 也就是本篇文章的重点；大致实现思路就是 **发起请求的时候先去 redis 获取 token ， 将获取的token 放入 请求的hearder ， 当请求到达服务端的时候拦截请求，对请求的 hearder 中的token，进行校验，如果校验通过则 放开拦截，删除token，否则 使用自定义异常返回错误信息；**

 校验：非空，是否过期，是否已经存在

## 有了springboot为什么还需要springcloud（重点）

一个服务编写好后，一启动就会自动注册到服务中心，调用方只管向服务中心发送服务的名称和参数，根本不关注服务的路径。springboot就是做服务的工具，springcloud是管理服务运行的工具。
