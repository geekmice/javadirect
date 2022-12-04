

![图片](https://mmbiz.qpic.cn/mmbiz_png/1J6IbIcPCLZMkG4ELsMmbSHDHwfFGic4CA7GmsCW8v9loCtGyHjPUa142lvp8aV4ViaFAsXDp6nGC9Z3ZryRKniaA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## 什么是springcloud

Spring Cloud是一系列框架的有序集合。它利用Spring Boot的开发便利性巧妙地简化了分布式系统基础设施的开发，如服务发现注册、配置中心、智能路由、消息总线、负载均衡、断路器、数据监控等，都可以用Spring Boot的开发风格做到一键启动和部署。Spring Cloud并没有重复制造轮子，它只是将各家公司开发的比较成熟、经得起实际考验的服务框架组合起来，通过Spring Boot风格进行再封装屏蔽掉了复杂的配置和实现原理，最终给开发者留出了一套简单易懂、易部署和易维护的分布式系统开发工具包。

## Spring Cloud Gateway 网关（重点）

### predicate简介

#### 应用场景

- 协议转换，路由转发
- 流量聚合，对流量进行监控，日志输出
- 作为整个系统的前端工程，对流量进行控制，有限流的作用
- 作为系统的前端边界，外部流量只能通过网关才能访问系统
- 可以在网关层做权限的判断
- 可以在网关层做缓存
- 跨域处理

![image-20211215104651730](http://oss.geekmice.cn/blog/image-20211215104651730.png)

在上图中，有很多类型的Predicate,比如说时间类型的Predicated（AfterRoutePredicateFactory BeforeRoutePredicateFactory BetweenRoutePredicateFactory），当只有满足特定时间要求的请求会进入到此predicate中，并交由router处理；cookie类型的CookieRoutePredicateFactory，指定的cookie满足正则匹配，才会进入此router;以及host、method、path、querparam、remoteaddr类型的predicate，每一种predicate都会对当前的客户端请求进行判断，是否满足当前的要求，如果满足则交给当前请求处理。如果有很多个Predicate，并且一个请求满足多个Predicate，则按照配置的顺序第一个生效

#### 实战操作predicate

##### After Route Predicate Factory

AfterRoutePredicateFactory，可配置一个时间，当请求的时间在配置时间之后，才交给 router去处理。否则则报错，不通过路由。

在工程的application.yml配置如下：

```yml
server:
  port: 8081
spring:
  profiles:
    active: after_route

---
spring:
  cloud:
    gateway:
      routes:
        - id: after_route
          uri: http://localhost:8080/get
          predicates:
            - After= 2017-01-20T17:42:47.789-07:00[America/Denver]
  profiles: after_route
```

在上面的配置文件中，配置了服务的端口为8081，配置spring.profiles.active:afterroute指定了程序的spring的启动文件为after_route文件。在application.yml再建一个配置文件，语法是三个横线，在此配置文件中通过spring.profiles来配置文件名，和spring.profiles.active一致，然后配置spring cloud gateway 相关的配置，id标签配置的是router的id，每个router都需要一个唯一的id，uri配置的是将请求路由到哪里，本案例全部路由到http://httpbin.org:80/get

predicates： After=2017-01-20T17:42:47.789-07:00[America/Denver] 会被解析成PredicateDefinition对象 （name =After ，args= 2017-01-20T17:42:47.789-07:00[America/Denver]）。在这里需要注意的是predicates的After这个配置，遵循的契约大于配置的思想，它实际被AfterRoutePredicateFactory这个类所处理，这个After就是指定了它的Gateway web handler类为AfterRoutePredicateFactory，同理，其他类型的predicate也遵循这个规则

当请求的时间在这个配置的时间之后，请求会被路由到http://httpbin.org:80/get

启动工程，在浏览器上访问http://localhost:8081/，会显示http://httpbin.org:80/get返回的结果，此时gateway路由到了配置的uri。如果我们将配置的时间设置到当前时之后，浏览器会显示404，此时证明没有路由到配置的uri

跟时间相关的predicates还有Before Route Predicate Factory、Between Route Predicate Factory

##### Header Route Predicate Factory

Header Route Predicate Factory需要2个参数，一个是header名，另外一个header值，该值可以是一个正则表达式。当此断言匹配了请求的header名和值时，断言通过，进入到router的规则中去。

在工程的配置文件加上以下的配置：

```yml
spring:
  profiles:
    active: header_route

---
spring:
  cloud:
    gateway:
      routes:
        - id: header_route
          uri: http://localhost:8080/get
          predicates:
            - Header=X-Request-Id,\d+
  profiles: header_route
```

在上面的配置中，当请求的Header中有X-Request-Id的header名，且header值为数字时，请求会被路由到配置的 uri. 使用curl执行以下命令:

```sh
$ curl -H 'X-Request-Id:1' localhost:8081
```

执行命令后，会正确的返回请求结果，结果省略。如果在请求中没有带上X-Request-Id的header名，并且值不为数字时，请求就会报404，路由没有被正确转发。



##### Path Route Predicate Factory

Path Route Predicate Factory 需要一个参数: 一个spel表达式，应用匹配路径。

在工程的配置文件application.yml文件中，做以下的配置：

```yml
spring:
  profiles:
    active: path_route

---
spring:
  cloud:
    gateway:
      routes:
        - id: path_route
          uri: http://localhost:8080/get
          predicates:
            - Path=/foo/{segment}
  profiles: path_route
```

在上面的配置中，所有的请求路径满足/foo/{segment}的请求将会匹配并被路由，比如/foo/1 、/foo/bar的请求，将会命中匹配，并成功转发。

使用curl模拟一个请求localhost:8081/foo/dew，执行之后会返回正确的请求结果。

```sh
$ curl localhost:8081/foo/de
```

Predict作为断言，它决定了请求会被路由到哪个router 中。在断言之后，请求会被进入到filter过滤器的逻辑

Predict决定了请求由哪一个路由处理，在路由处理之前，需要经过“pre”类型的过滤器处理，处理返回响应之后，可以由“post”类型的过滤器处理

### filter作用和生命周期

由filter工作流程点，可以知道filter有着非常重要的作用，在“pre”类型的过滤器可以做参数校验、权限校验、流量监控、日志输出、协议转换等，在“post”类型的过滤器中可以做响应内容、响应头的修改，日志的输出，流量监控等。首先需要弄清一点为什么需要网关这一层，这就不得不说下filter的作用了

#### 作用

当我们有很多个服务时，比如中的审批服务，分发服务，下达精确指令等服务，客户端请求各个服务的Api时，每个服务都需要做相同的事情，比如鉴权、限流、日志输出等

对于这样重复的工作，有没有办法做的更好，答案是肯定的。在微服务的上一层加一个全局的权限控制、限流、日志输出的Api Gateway服务，然后再将请求转发到具体的业务服务层。这个Api Gateway服务就是起到一个服务边界的作用，外接的请求访问系统，必须先通过网关层

#### 生命周期

Spring Cloud Gateway同zuul类似，有“pre”和“post”两种方式的filter。客户端的请求先经过“pre”类型的filter，然后将请求转发到具体的业务服务，比如上图中的user-service，收到业务服务的响应之后，再经过“post”类型的filter处理，最后返回响应到客户端

与zuul不同的是，filter除了分为“pre”和“post”两种方式的filter外，在Spring Cloud Gateway中，filter从作用范围可分为另外两种，一种是针对于单个路由的gateway filter，它在配置文件中的写法同predicate类似；另外一种是针对于所有路由的global gateway filer。现在从作用范围划分的维度来讲解这两种filter

##### AddRequestHeader GatewayFilter Factory

在工程的配置文件中，加入以下的配置

```yml
server:
  port: 8081
spring:
  profiles:
    active: add_request_header_route
    
---
spring:
  cloud:
    gateway:
      routes:
      - id: add_request_header_route
        uri: http://httpbin.org:80/get
        filters:
        - AddRequestHeader=X-Request-Foo,Bar
        predicates:
        - After: After=2017-01-20T17:42:47.789-07:00[America/Denver]
  profiles: add_request_header_route
```

在上述的配置中，工程的启动端口为8081，配置文件为addrequestheaderroute，在addrequestheaderroute配置中，配置了roter的id为addrequestheader_route，路由地址为http://httpbin.org:80/get，该router有AfterPredictFactory，有一个filter为AddRequestHeaderGatewayFilterFactory(约定写成AddRequestHeader)，AddRequestHeader过滤器工厂会在请求头加上一对请求头，名称为X-Request-Foo，值为Bar。

启动工程，通过curl命令来模拟请求：

```
curl localhost:8081
```

最终显示了从 http://httpbin.org:80/get得到了请求，响应如下：

**![image-20211215112159035](http://oss.geekmice.cn/blog/image-20211215112159035.png)**


可以上面的响应可知，确实在请求头中加入了X-Request-Foo这样的一个请求头，在配置文件中配置的AddRequestHeader过滤器工厂生效。

跟AddRequestHeader过滤器工厂类似的还有AddResponseHeader过滤器工厂，在此就不再重复。

##### global filter

每一个GlobalFilter都作用在每一个router上，能够满足大多数的需求。但是如果遇到业务上的定制，可能需要编写满足自己需求的GlobalFilter。在下面的案例中将讲述如何编写自己GlobalFilter，该GlobalFilter会校验请求中是否包含了请求参数“token”，如何不包含请求参数“token”则不转发路由，否则执行正常的逻辑

```java
public class TokenFilter implements GlobalFilter,Ordered{
    Logger logger=LoggerFactory.getLogger(TokenFilter.class)
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain){
        String token = exchange.getRequest().getQueryParams().getFirst("token");
        if(token == null || token.isEmpty()){
            logger.info("token is empty");
            exchange.getResponse().setStatusCode(HttpStatus.UNAUTHORIZED);
            return exchange.getResponse().setComplete();
        }
    }
    @Overrize
    public int getOrder(){
        return -100;
    }
}
```

在上面的TokenFilter需要实现GlobalFilter和Ordered接口，这和实现GatewayFilter很类似。然后根据ServerWebExchange获取ServerHttpRequest，然后根据ServerHttpRequest中是否含有参数token，如果没有则完成请求，终止转发，否则执行正常的逻辑。

然后需要将TokenFilter在工程的启动类中注入到Spring Ioc容器中，代码如下：

```java
@Beanpublic TokenFilter tokenFilter(){
    return new TokenFilter();
}
```

启动工程，使用curl命令请求：

```sh
 curl localhost:8081/customer/123
```

可以看到请没有被转发，请求被终止，并在控制台打印了如下日志：

```sh
2018-11-16 15:30:13.543  INFO 19372 --- [ctor-http-nio-2] gateway.TokenFilter                      : token is empty...
```

上面的日志显示了请求进入了没有传“token”的逻辑。

##### GatewayFilter和GlobalFilter二者区别

- GatewayFilter : 需要通过spring.cloud.routes.filters 配置在具体路由下，只作用在当前路由上或通过spring.cloud.default-filters配置在全局，作用在所有路由上
- GlobalFilter : 全局过滤器，不需要在配置文件中配置，作用在所有的路由上，最终通过GatewayFilterAdapter包装成GatewayFilterChain可识别的过滤器，它为请求业务以及路由的URI转换为真实业务服务的请求地址的核心过滤器，不需要配置，系统初始化时加载，并作用在每个路由上。

- `Route（路由）`：路由是网关的
- 基本单元，由ID、URI、一组Predicate、一组Filter组成，根据Predicate进行匹配转发。
- `Predicate（谓语、断言）`：路由转发的判断条件，目前`SpringCloud Gateway`支持多种方式，常见如：`Path`、`Query`、`Method`、`Header`等。
- `Filter（过滤器）`：过滤器是路由转发请求时所经过的过滤逻辑，可用于修改请求、响应内容。

API网关组件，对请求提供路由及过滤功能。

**先来解释下`route`的组成部分：**

- `id`：路由的ID
- `uri`：匹配路由的转发地址
- `predicates`：配置该路由的断言，通过`PredicateDefinition`类进行接收配置。

## Spring Cloud OpenFeign 服务调用（重点）

可以动态创建基于Spring MVC注解的接口实现用于服务调用

主启动类加上注解@EnableFeignClients注解

EnableFeignClients申明该项目是Feign客户端，扫描对应的feign client,并添加@FeignClient(name="springcloud-product-provider")注解，其中name就是我们要访问的微服务的名称

没有底层的建立连接、构造请求、解析响应的代码，直接就是用注解定义一个 FeignClient接口，然后调用那个接口就可以了。人家Feign Client会在底层根据你的注解，跟你指定的服务建立连接、构造请求、发起靕求、获取响应、解析响应，等等。这一系列脏活累活，人家Feign全给你干了 

那么问题来了，Feign是如何做到这么神奇的呢？很简单，**Feign的一个关键机制就是使用了动态代理**。咱们一起来看看下面的图，结合图来分析：

- 首先，如果你对某个接口定义了@FeignClient注解，Feign就会针对这个接口创建一个动态代理
- 接着你要是调用那个接口，本质就是会调用 Feign创建的动态代理，这是核心中的核心
- Feign的动态代理会根据你在接口上的@RequestMapping等注解，来动态构造出你要请求的服务的地址
- 最后针对这个地址，发起请求、解析响应

 

## Spring Cloud Eureka 注册中心（重点）

Eureka是微服务架构中的注册中心，专门负责服务的注册与发现

现券买卖服务，库存服务，订单服务服务中都有一个Eureka Client组件，这个组件专门负责将这个服务的信息注册到Eureka Server中。说白了，就是告诉Eureka Server，自己在哪台机器上，监听着哪个端口。而Eureka Server是一个注册中心，里面有一个注册表，保存了各服务所在的机器和端口号。

订单服务里也有一个Eureka Client组件，这个Eureka Client组件会找Eureka Server问一下：库存服务在哪台机器啊？监听着哪个端口啊？库存服务呢？质押式回购服务呢？然后就可以把这些相关信息从Eureka Server的注册表中拉取到自己本地缓存起来。

这时如果订单服务想要调用库存服务，不就可以找自己本地的Eureka Client问一下库存服务在哪台机器？监听哪个端口吗？收到响应后，紧接着就可以发送一个请求过去，调用库存服务扣减库存的那个接口！同理，如果订单服务要调用仓储服务、积分服务，也是如法炮制。

总结

- **Eureka** **Client：**负责将这个服务的信息注册到Eureka Server中
- **Eureka Server：**注册中心，里面有一个注册表，保存了各个服务所在的机器和端口号

## Spring Cloud Ribbon 负载均衡（重点）

说完了Feign，还没完。现在新的问题又来了，如果人家库存服务部署在了5台机器上

```sh
192.168.169:9000

192.168.170:9000

192.168.171:9000

192.168.172:9000

192.168.173:9000
```

Feign怎么知道该请求哪台机器呢

这时Spring Cloud Ribbon就派上用场了。Ribbon就是专门解决这个问题的。它的作用是负载均衡，会帮你在每次请求时选择一台机器，均匀的把请求分发到各个机器上

Ribbon的负载均衡默认使用的最经典的Round Robin轮询算法。这是啥？简单来说，就是如果订单服务对库存服务发起10次请求，那就先让你请求第1台机器、然后是第2台机器、第3台机器、第4台机器、第5台机器，接着再来—个循环，第1台机器、第2台机器。。。以此类推

**Ribbon是和Feign以及Eureka紧密协作，完成工作的**

- 首先Ribbon会从 Eureka Client里获取到对应的服务注册表，也就知道了所有的服务都部署在了哪些机器上，在监听哪些端口号。
- 然后Ribbon就可以使用默认的Round Robin算法，从中选择一台机器
- Feign就会针对这台机器，构造并发起请求。

![图片](https://mmbiz.qpic.cn/mmbiz_png/1J6IbIcPCLZMkG4ELsMmbSHDHwfFGic4CtD7kdmviciaXUubWd69erjZn8Hgw5ZZrDWYDBYCAiaBchGDxxwQj7Lic0w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## Spring Cloud Hystrix 断路器（重点）

启动类中加入@EnableCircuitBreaker注解
另因为原启动类中，常需要加入Eureka的注册注解@EnableDiscoveryClient以及SpringBoot启动注解@SpringBootApplication

在微服务架构里，一个系统会有很多的服务。以本文的业务场景为例：现券买卖服务需要下达指令在一个业务流程里需要调用三个服务。现在假设订单服务自己最多只有100个线程可以处理请求，然后呢，指令审批服务不幸的挂了，每次订单服务调用指令审批服务的时候，都会卡住几秒钟，然后抛出—个超时异常。

咱们一起来分析一下，这样会导致什么问题？

- 如果系统处于高并发的场景下，大量请求涌过来的时候，订单服务的100个线程都会卡在请求指令审批服务这块。导致订单服务没有一个线程可以处理请求
- 然后就会导致别人请求订单服务的时候，发现订单服务也挂了，不响应任何请求了

 

上面这个，就是**微服务架构中恐怖的服务雪崩问题**，如下图所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/1J6IbIcPCLZMkG4ELsMmbSHDHwfFGic4CMNiazSoZtYictsye5m2UbKUeJP8mOcXpgSBsMWhlqEqScb5B25NkdLqQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

 

如上图，这么多服务互相调用，要是不做任何保护的话，某一个服务挂了，就会引起连锁反应，导致别的服务也挂。比如指令审批服务挂了，会导致订单服务的线程全部卡在请求指令审批服务这里，没有一个线程可以工作，瞬间导致订单服务也挂了，别人请求订单服务全部会卡住，无法响应。

 

**现在问题分析完了，如何解决？**

这时就轮到Hystrix闪亮登场了。Hystrix是隔离、熔断以及降级的一个框架。啥意思呢？说白了，Hystrix会搞很多个小小的线程池，比如订单服务请求库存服务是一个线程池，请求指令审批服务是一个线程池。每个线程池里的线程就仅仅用于请求那个服务。

 

**打个比方：现在很不幸，指令审批挂了，会咋样？**

当然会导致订单服务里的那个用来调用指令审批服务的线程都卡死不能工作了啊！但是由于订单服务调用库存服务，指令分发服务的这两个线程池都是正常工作的，所以这两个服务不会受到任何影响



这个时候如果别人请求订单服务，订单服务还是可以正常调用库存服务扣减库存，调用仓储服务通知发货。只不过调用指令审批服务的时候，每次都会报错。**但是如果指令审批都挂了，每次调用都要去卡住几秒钟干啥呢？****有意义吗？当然没有！`所以我们直接对积分服务熔断不就得了，比如在5分钟内请求积分服务直接就返回了，不要去走网络请求卡住几秒钟，这个过程，就是所谓的熔断！`

 

**那人家又说，兄弟，积分服务挂了你就熔断，好歹你干点儿什么啊！别啥都不干就直接返回啊？**没问题，咱们就来个降级：每次调用指令审批服务，你就在数据库里记录一条消息，说给某某用户增加了多少审批情况如何，因为该服务挂了，导致没成功！这样等指令审批服务恢复了，你可以根据这些记录手工加一下积分。这个过程，就是所谓的降级。

 

为帮助大家更直观的理解，接下来用一张图，梳理一下Hystrix隔离、熔断和降级的全流程：

![图片](https://mmbiz.qpic.cn/mmbiz_png/1J6IbIcPCLZMkG4ELsMmbSHDHwfFGic4CZsj0neCTAwgfBVClSwTarrZ7gMRKAQlNviabrsOJ0DA9GcSmuZ2BTcQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

正确的方法上加上注解@HystrixCommand(defaultFallback = “”)参数为降级方法名。
降级方法的返回值与参数类型必须与原方法一致

类上可以写一个注解，用于所有方法的降级处理
@DefaultProperties（defaultFallback = “”） 在方法上就可以只用写@HystrixCommand用来启用降级处理，不用再写默认方法了
通用方法就不要写参数了
Hystix服务降级默认超时是1秒，不同业务可能耗时不同，所以超时时间要手动配置



[拜托！面试请不要再问我Spring Cloud底层原理 ](https://mp.weixin.qq.com/s?__biz=MzAxNjk4ODE4OQ==&mid=2247484647&idx=1&sn=f6439e38b3c89a0c5f184db08bf2157c&chksm=9bed2595ac9aac831f4b88c65d8d18df1cc6888ad310681e7d0187750fe808213c7ef282e691&mpshare=1&scene=2&srcid=11139YgAAWb8mht1bfPJlEEZ&from=timeline#rd)
[filter的作用和生命周期 ](https://mp.weixin.qq.com/s/CZtOz_qBh9XcHd8RRc1bkw)
