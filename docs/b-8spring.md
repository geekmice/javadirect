

# Spring #

## 1、什么是spring？ ##

一个轻量级Java开发框架，解决企业级应用开发的复杂性，即简化Java开发

## 2、使用spring框架的优劣是什么 ##

优势

- 方便解耦，简化开发

  Spring就是一个大工厂，可以将所有对象的创建和依赖关系的维护，交给Spring管理。

- AOP编程的支持

  Spring提供面向切面编程，可以方便的实现对程序进行权限拦截、运行监控等功能。

- 声明式事务的支持

  只需要通过配置就可以完成对事务的管理，而无需手动编程。

- 方便程序的测试

  Spring对Junit4支持，可以通过注解方便的测试Spring程序。

- 方便集成各种优秀框架

  Spring不排斥各种优秀的开源框架，其内部提供了对各种优秀框架的直接支持（如：Struts、Hibernate、MyBatis等）。

- 降低JavaEE API的使用难度

  Spring对JavaEE开发中非常难用的一些API（JDBC、JavaMail、远程调用等），都提供了封装，使这些API应用难度大大降低。

缺点

- Spring明明一个很轻量级的框架，却给人感觉大而全
- Spring依赖反射，反射影响性能
- 使用门槛升高，入门Spring需要较长时间



## 3、spring有哪些模块组成 ##

- spring core：提供了框架的基本组成部分，包括控制反转（Inversion of Control，IOC）和依赖注入（Dependency Injection，DI）功能。
- spring beans：提供了BeanFactory，是工厂模式的一个经典实现，Spring将管理对象称为Bean。
- spring context：构建于 core 封装包基础上的 context 封装包，提供了一种框架式的对象访问方法。
- spring jdbc：提供了一个JDBC的抽象层，消除了烦琐的JDBC编码和数据库厂商特有的错误代码解析， 用于简化JDBC。
- spring aop：提供了面向切面的编程实现，让你可以自定义拦截器、切点等。
- spring Web：提供了针对 Web 开发的集成特性，例如文件上传，利用 servlet listeners 进行 ioc 容器初始化和针对 Web 的 ApplicationContext。
- spring test：主要为测试提供支持的，支持使用JUnit或TestNG对Spring组件进行单元测试和集成测试。



## 4、核心容器(应用上下文)模块spring context应用上下文 ##

​	这是基本的Spring模块，提供spring 框架的基础功能，BeanFactory 是 任何以spring为基础的应用的核心。Spring 框架建立在此模块之上，它使Spring成为一个容器。

​	Bean 工厂是工厂模式的一个实现，提供了控制反转功能，用来把应用的配置和依赖从真正的应用代码中分离。最常用的就是org.springframework.beans.factory.xml.XmlBeanFactory ，它根据XML文件中的定义加载beans。该容器从XML 文件读取配置元数据并用它去创建一个完全配置的系统或应用。



## 5、spring框架中使用了哪些设计模式 ##

1. 工厂模式：BeanFactory就是简单工厂模式的体现，用来创建对象的实例；
2. 单例模式：在 spring 配置文件中定义的 bean默认为单例模式。
3. 代理模式：Spring的AOP功能用到了JDK的动态代理和CGLIB字节码生成技术；
4. 模板方法：用来解决代码重复的问题。比如. RestTemplate, JmsTemplate, JpaTemplate。
5. 观察者模式：定义对象键一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都会得到通知被制动更新，如Spring中listener的实现–ApplicationListener。



## 7、如何理解AOP ##

​	OOP(Object-Oriented Programming)面向对象编程，允许开发者定义纵向的关系，但并适用于定义横向的关系，导致了大量代码的重复，而不利于各个模块的重用。

​	AOP(Aspect-Oriented Programming)，一般称为面向切面编程，作为面向对象的一种补充，用于将那些与业务无关，但却对多个对象产生影响的公共行为和逻辑，抽取并封装为一个可重用的模块，这个模块被命名为“切面”（Aspect），减少系统中的重复代码，降低了模块间的耦合度，同时提高了系统的可维护性。可用于权限认证、日志、事务处理等。



## 8、如何理解IOC ##

​	控制反转即IoC (Inversion of Control)，它把传统上由程序代码直接操控的对象的调用权交给容器，通过容器来实现对象组件的装配和管理。所谓的“控制反转”概念就是对组件对象控制权的转移，从程序代码本身转移到了外部容器。

​	Spring IOC 负责创建对象，管理对象（通过依赖注入（DI），装配对象，配置对象，并且管理这些对象的整个生命周期。

​	原理就是工厂模式加反射机制

## 9、IOC优势 ##

- IOC 或 依赖注入把应用的代码量降到最低。
- 它使应用容易测试，单元测试不再需要单例和JNDI查找机制。
- 最小的代价和最小的侵入性使**松散耦合**得以实现。
- IOC容器支持加载服务时的**饿汉式初始化和懒加载**。



## 10、beanFactory和applicationFactory区别 ##

**依赖关系**

BeanFactory：是Spring里面最底层的接口，包含了各种Bean的定义，读取bean配置文档，管理bean的加载、实例化，控制bean的生命周期，维护bean之间的依赖关系。

ApplicationContext接口作为BeanFactory的派生，除了提供BeanFactory所具有的功能外，还提供了更完整的框架功能：

- 继承MessageSource，因此支持国际化。
- 统一的资源文件访问方式。
- 提供在监听器中注册bean的事件。
- **同时加载多个配置文件。**
- **载入多个（有继承关系）上下文 ，使得每一个上下文都专注于一个特定的层次，比如应用的web层。**

**加载方式**

BeanFactroy采用的是**延迟加载形式**来注入Bean的，即只有在使用到某个Bean时(调用getBean())，才对该Bean进行加载实例化。这样，我们就不能发现一些存在的Spring的配置问题。如果Bean的某一个属性没有注入，BeanFacotry加载后，直至第一次使用调用getBean方法才会抛出异常。

ApplicationContext，立即加载，它是在容器启动时，一次性创建了所有的Bean。这样，在容器启动时，我们就可以发现Spring中存在的配置错误，这样有利于检查所依赖属性是否注入。 ApplicationContext启动后预载入所有的单实例Bean，通过预载入单实例bean ,确保当你需要的时候，你就不用等待，因为它们已经创建好了。

相对于基本的BeanFactory，ApplicationContext 唯一的不足是占用内存空间。当应用程序配置Bean较多时，程序启动较慢。



## 11、何为依赖注入（了解） ##

控制反转IoC是一个很大的概念，可以用不同的方式来实现。其主要实现方式有两种：依赖注入和依赖查找

依赖注入：相对于IoC而言，依赖注入(DI)更加准确地描述了IoC的设计理念。所谓依赖注入（Dependency Injection），即组件之间的依赖关系由容器在应用系统运行期来决定，也就是由容器动态地将某种依赖关系的目标对象实例注入到应用系统中的各个关联的组件之中。组件不做定位查询，只提供普通的Java方法让容器去决定依赖关系。



## 12、有哪些不同类型的依赖注入 ##

依赖注入是时下最流行的IoC实现方式，依赖注入分为接口注入（Interface Injection），Setter方法注入（Setter Injection）和构造器注入（Constructor Injection）三种方式。其中接口注入由于在灵活性和易用性比较差，现在从Spring4开始已被废弃。

**构造器依赖注入**：构造器依赖注入通过容器触发一个类的构造器来实现的，该类有一系列参数，每个参数代表一个对其他类的依赖。

**Setter方法注入**：Setter方法注入是容器通过调用无参构造器或无参static工厂 方法实例化bean之后，调用该bean的setter方法，即实现了基于setter的依赖注入。

构造器依赖注入和 Setter方法注入的区别

| **构造函数注入**           | **setter** **注入**        |
| -------------------------- | -------------------------- |
| 没有部分注入               | 有部分注入                 |
| 不会覆盖 setter 属性       | 会覆盖 setter 属性         |
| 任意修改都会创建一个新实例 | 任意修改不会创建一个新实例 |
| 适用于设置很多属性         | 适用于设置少量属性         |

两种依赖方式都可以使用，构造器注入和Setter方法注入。最好的解决方案是用构造器参数实现强制依赖，setter方法实现可选依赖。



## 13、哪种依赖注入方式你建议使用，构造器注入，还是setter方法注入 ##

**你两种依赖方式都可以使用，构造器注入和****Setter****方法注入。最好的解决方案是用构造器参数实现强制依赖，****setter****方法实现可选依赖**



## **14、什么是Spring Bean定义 包含什么** ##

Spring beans 是那些形成Spring应用的主干的java对象。它们被Spring IOC容器初始化，装配，和管理。这些beans通过容器中配置的元数据创建。比如，以XML文件中 的形式定义。

**Spring** **框架定义的beans****都是单件****beans****。在****bean tag****中有个属性****”singleton”****，如果它被赋为****TRUE****，****bean** **就是单件，否则就是一个** **prototype bean****。默认是****TRUE****，所以所有在****Spring****框架中的****beans** **缺省都是单件。**



## **15、如何给spring容器提供配置元数据** ##

这里有三种重要的方法给Spring 容器提供配置元数据。

- XML配置文件。
- 基于注解的配置。
- 基于java的配置。



## **16、你怎样定义类的作用域** ##

当定义一个 在Spring里，我们还能给这个bean声明一个作用域。它可以通过bean 定义中的scope属性来定义。如，当Spring要在需要的时候每次生产一个新的bean实例，bean的scope属性被指定为prototype。另一方面，一个bean每次使用的时候必须返回同一个实例，这个bean的scope 属性 必须设为 singleton。



## **17、解释spring支持的集中bean的作用域** ##

Spring框架支持以下五种bean的作用域：

- **singleton :** bean在每个Spring ioc 容器中只有一个实例。
- **prototype**：一个bean的定义可以有多个实例。
- **request**：每次http请求都会创建一个bean，该作用域仅在基于web的Spring ApplicationContext情形下有效。
- **session**：在一个HTTP Session中，一个bean定义对应一个实例。该作用域仅在基于web的Spring ApplicationContext情形下有效。
- **global-session**：在一个全局的HTTP Session中，一个bean定义对应一个实例。该作用域仅在基于web的Spring ApplicationContext情形下有效。

**注意：** 缺省的Spring bean 的作用域是Singleton。使用 prototype 作用域需要慎重的思考，因为频繁创建和销毁 bean 会带来很大的性能开销。



## **18、spring框架中bean的生命周期** ##

**Spring****容器** **从****XML** **文件中读取****bean****的定义，并实例化****bean****。**

**Spring****根据****bean****的定义填充所有的属性。**

**如果****bean****实现了****BeanNameAware** **接口，****Spring** **传递****bean** **的****ID** **到** **setBeanName****方法。**

**如果****Bean** **实现了** **BeanFactoryAware** **接口，** **Spring****传递****beanfactory** **给****setBeanFactory** **方法。****如果有任何与****bean****相关联的****BeanPostProcessors****，****Spring****会在****postProcesserBeforeInitialization()****方法内调用它们。**

**如果****bean****实现****IntializingBean****了，调用它的****afterPropertySet****方法，如果****bean****声明了初始化方法，调用此初始化方法。**

**如果有****BeanPostProcessors** **和****bean** **关联，这些****bean****的****postProcessAfterInitialization()** **方法将被调用。**

**如果****bean****实现了** **DisposableBean****，它将调用****destroy()****方法。



## **19、spring框架中的单例bean是线程安全的吗** ##

不是，Spring框架中的单例bean不是线程安全的。

spring 中的 bean 默认是单例模式，spring 框架并没有对单例 bean 进行多线程的封装处理。

实际上大部分时候 spring bean 无状态的（比如 dao 类），所有某种程度上来说 bean 也是安全的，但如果 bean 有状态的话（比如 view model 对象），那就要开发者自己去保证线程安全了，最简单的就是改变 bean 的作用域，把“singleton”变更为“prototype”，这样请求 bean 相当于 new Bean()了，所以就可以保证线程安全了。

- 有状态就是有数据存储功能。
- 无状态就是不会保存数据

方案二：

1. Spring 容器中的 Bean 默认是单例的，所有线程都共享一个单实例的 Bean，因 此是存在资源的竞争。如果单例 Bean,是一个无状态 Bean，也就是线程中的操作不 会对 Bean 的成员执行查询以外的操作，那么这个单例 Bean 是线程安全的。比如 Spring mvc 的 Controller、Service、Dao 等，这些 Bean 大多是无状态的，只 关注于方法本身。对于有状态的 bean，是线程不安全的，但是我们可以通过 ThreadLocal 去解决线程安全的方法。

   threadlocal
   	变量值的共享可以使用public static的形式，所有线程都使用同一个变量。
   ThreadLocal类并不是用来解决多线程环境下的共享变量问题，而是用来提供线程内部的共享变量，在多线程环境下，可以保证各个线程之间的变量互相隔离、相互独立。在线程中，可以通过get()/set()方法来访问变量。ThreadLocal实例通常来说都是private static类型的，它们希望将状态与线程进行关联。这种变量在线程的生命周期内起作用，可以减少同一个线程内多个函数或者组件之间一些公共变量的传递的复杂度。



## **20、在spring中如何注入一个java集合** ##

Spring提供以下几种集合的配置元素：

list类型用于注入一列值，允许有相同的值。

set类型用于注入一组值，不允许有相同的值。

map类型用于注入一组键值对，键和值都可以为任意类型。

props类型用于注入一组键值对，键和值都只能为String类型。



## **21、什么是bean装配，bean的自动装配** ##

​	装配，或bean 装配是指在Spring 容器中把bean组装到一起，前提是容器需要知道bean的依赖关系，如何通过依赖注入来把它们装配到一起

​	在Spring框架中，在配置文件中设定bean的依赖关系是一个很好的机制，Spring 容器能够自动装配相互合作的bean，这意味着容器不需要和配置，能通过Bean工厂自动处理bean之间的协作。这意味着 Spring可以通过向Bean Factory中注入的方式自动搞定bean之间的依赖关系。自动装配可以设置在每个bean上，也可以设定在特定的bean上。



## **22、解释不同方式的自动装配，spring 自动装配 bean 有哪些方式** ##

在spring中，对象无需自己查找或创建与其关联的其他对象，由容器负责把需要相互协作的对象引用赋予各个对象，使用autowire来配置自动装载模式。

在Spring框架xml配置中共有5种自动装配：

- no：默认的方式是不进行自动装配的，通过手工设置ref属性来进行装配bean。
- byName：通过bean的名称进行自动装配，如果一个bean的 property 与另一bean 的name 相同，就进行自动装配。
- byType：通过参数的数据类型进行自动装配。
- constructor：利用构造函数进行装配，并且构造函数的参数通过byType进行装配。
- autodetect：自动探测，如果有构造方法，通过 construct的方式自动装配，否则使用 byType的方式自动装配。



## **23、你可以在spring中注入一个null和一个空字符串吗** ##

可以



## **24、什么是基于Java的spring注解配置？给一些注解的例子** ##

**基于****Java****的配置，允许你在少量的****Java****注解的帮助下，进行你的大部分****Spring****配置而非通过****XML****文件。**

**以****@Configuration** **注解为例，它用来标记类可以当做一个****bean****的定义，被****Spring IOC****容器使用。另一个例子是****@Bean****注解，它表示此方法将要返回一个对象，作为一个****bean****注册进****Spring****应用上下文。**



## **25、怎样开启注解装配** ##

**注解装配在默认情况下是不开启的，为了使用注解装配，我们必须在****Spring****配置文件中配置** **<context:annotation-config/>元素。



## 26、@Component, @Controller, @Repository, @Service 有何区别？ ##

@Component：这将 java 类标记为 bean。它是任何 Spring 管理组件的通用构造型。spring 的组件扫描机制现在可以将其拾取并将其拉入应用程序环境中。

@Controller：这将一个类标记为 Spring Web MVC 控制器。标有它的 Bean 会自动导入到 IoC 容器中。

@Service：此注解是组件注解的特化。它不会对 @Component 注解提供任何其他行为。您可以在服务层类中使用 @Service 而不是 @Component，因为它以更好的方式指定了意图。

@Repository：这个注解是具有类似用途和功能的 @Component 注解的特化。它为 DAO 提供了额外的好处。它将 DAO 导入 IoC 容器，并使未经检查的异常有资格转换为 Spring DataAccessException。



## 27、@Required 注解有什么作用

这个注解表明bean的属性必须在配置的时候设置，通过一个bean定义的显式的属性值或通过自动装配，若@Required注解的bean属性未被设置，容器将抛出BeanInitializationException。示例：

```java
public class Employee {
    private String name;
    @Required
    public void setName(String name){
        this.name=name;
    }
    public string getName(){
        return name;
    }
}
12345678910
```

## 28、@Autowired 注解有什么作用 ##

@Autowired默认是按照类型装配注入的，默认情况下它要求依赖对象必须存在（可以设置它required属性为false）。@Autowired 注解提供了更细粒度的控制，包括在何处以及如何完成自动装配。它的用法和@Required一样，修饰setter方法、构造器、属性或者具有任意名称和/或多个参数的PN方法。

```java
public class Employee {
    private String name;
    @Autowired
    public void setName(String name) {
        this.name=name;
    }
    public string getName(){
        return name;
    }
}
12345678910
```

## 29、@Autowired和@Resource之间的区别 ##

@Autowired可用于：构造函数、成员变量、Setter方法

@Autowired和@Resource之间的区别

- @Autowired默认是按照**类型**装配注入的，默认情况下它要求依赖对象必须存在（可以设置它required属性为false）。
- @Resource默认是按照**名称**来装配注入的，只有当找不到与名称匹配的bean才会按照类型来装配注入。

## 30、@Qualifier 注解有什么作用 ##

当您创建多个相同类型的 bean 并希望仅使用属性装配其中一个 bean 时，您可以使用@Qualifier 注解和 @Autowired 通过指定应该装配哪个确切的 bean 来消除歧义。



## **31、Spring支持的事务管理类型**

Spring支持两种类型的事务管理：

**编程式事务管理**：这意味你通过编程的方式管理事务，给你带来极大的灵活性，但是难维护。

**声明式事务管理**：这意味着你可以将业务代码和事务管理分离，你只需用注解和XML配置来管理事务。

## 32、说一下Spring的事务传播行为

spring事务的传播行为说的是，当多个事务同时存在的时候，spring如何处理这些事务的行为。

> ① PROPAGATION_REQUIRED：如果当前没有事务，就创建一个新事务，如果当前存在事务，就加入该事务，该设置是最常用的设置。
>
> ② PROPAGATION_SUPPORTS：支持当前事务，如果当前存在事务，就加入该事务，如果当前不存在事务，就以非事务执行。
>
> ③ PROPAGATION_MANDATORY：支持当前事务，如果当前存在事务，就加入该事务，如果当前不存在事务，就抛出异常。
>
> ④ PROPAGATION_REQUIRES_NEW：创建新事务，无论当前存不存在事务，都创建新事务。
>
> ⑤ PROPAGATION_NOT_SUPPORTED：以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。
>
> ⑥ PROPAGATION_NEVER：以非事务方式执行，如果当前存在事务，则抛出异常。
>
> ⑦ PROPAGATION_NESTED：如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则按REQUIRED属性执行。



## **33、Spring框架的事务管理有哪些优点** ##

- 为不同的事务API 如 JTA，JDBC，Hibernate，JPA 和JDO，提供一个不变的编程模式。
- 为编程式事务管理提供了一套简单的API而不是一些复杂的事务API
- 支持声明式事务管理。
- 和Spring各种数据访问抽象层很好得集成。



## **34、在Spring AOP中，关注点和横切关注的区别是什么** ##

关注点（concern）是应用中一个模块的行为，一个关注点可能会被定义成一个我们想实现的一个功能。

横切关注点（cross-cutting concern）是一个关注点，此关注点是整个应用都会使用的功能，并影响整个应用，比如日志，安全和数据传输，几乎应用的每个模块都需要的功能。因此这些都属于横切关注点。



## 35、Spring通知有哪些类型？ ##

在AOP术语中，切面的工作被称为通知，实际上是程序执行时要通过SpringAOP框架触发的代码段。

Spring切面可以应用5种类型的通知：

1. 前置通知（Before）：在目标方法被调用之前调用通知功能；
2. 后置通知（After）：在目标方法完成之后调用通知，此时不会关心方法的输出是什么；
3. 返回通知（After-returning ）：在目标方法成功执行之后调用通知；
4. 异常通知（After-throwing）：在目标方法抛出异常后调用通知；
5. 环绕通知（Around）：通知包裹了被通知的方法，在被通知的方法调用之前和调用之后执行自定义的行为。

> 同一个aspect，不同advice的执行顺序：
>
> ①没有异常情况下的执行顺序：
>
> around before advice
> before advice
> target method 执行
> around after advice
> after advice
> afterReturning
>
> ②有异常情况下的执行顺序：
>
> around before advice
> before advice
> target method 执行
> around after advice
> after advice
> afterThrowing:异常发生
> java.lang.RuntimeException: 异常发生



## 36、解释一下Spring AOP里面的几个名词 ##

（1）切面（Aspect）：切面是通知和切点的结合。通知和切点共同定义了切面的全部内容。 在Spring AOP中，切面可以使用通用类（基于模式的风格） 或者在普通类中以 @AspectJ 注解来实现。

（2）连接点（Join point）：指方法，在Spring AOP中，一个连接点 总是 代表一个方法的执行。 应用可能有数以千计的时机应用通知。这些时机被称为连接点。连接点是在应用执行过程中能够插入切面的一个点。这个点可以是调用方法时、抛出异常时、甚至修改一个字段时。切面代码可以利用这些点插入到应用的正常流程之中，并添加新的行为。

（3）通知（Advice）：在AOP术语中，切面的工作被称为通知。

（4）切入点（Pointcut）：切点的定义会匹配通知所要织入的一个或多个连接点。我们通常使用明确的类和方法名称，或是利用正则表达式定义所匹配的类和方法名称来指定这些切点。

（5）引入（Introduction）：引入允许我们向现有类添加新方法或属性。

（6）目标对象（Target Object）： 被一个或者多个切面（aspect）所通知（advise）的对象。它通常是一个代理对象。也有人把它叫做 被通知（adviced） 对象。 既然Spring AOP是通过运行时代理实现的，这个对象永远是一个 被代理（proxied） 对象。

（7）织入（Weaving）：织入是把切面应用到目标对象并创建新的代理对象的过程。在目标对象的生命周期里有多少个点可以进行织入：

- 编译期：切面在目标类编译时被织入。AspectJ的织入编译器是以这种方式织入切面的。
- 类加载期：切面在目标类加载到JVM时被织入。需要特殊的类加载器，它可以在目标类被引入应用之前增强该目标类的字节码。AspectJ5的加载时织入就支持以这种方式织入切面。
- 运行期：切面在应用运行的某个时刻被织入。一般情况下，在织入切面时，AOP容器会为目标对象动态地创建一个代理对象。SpringAOP就是以这种方式织入切面。



## **37、什么是Spring的MVC框架** ##

​	Spring MVC是一个基于Java的实现了MVC设计模式的请求驱动类型的轻量级Web框架，通过把模型-视图-控制器分离，将web层进行职责解耦，把复杂的web应用分成逻辑清晰的几部分，简化开发，减少出错，方便组内开发人员之间的配合。



## 38、springmvc常见组件 ##

（1）前端控制器 DispatcherServlet（不需要程序员开发）

作用：接收请求、响应结果，相当于转发器，有了DispatcherServlet 就减少了其它组件之间的耦合度。

（2）处理器映射器HandlerMapping（不需要程序员开发）

作用：根据请求的URL来查找Handler

（3）处理器适配器HandlerAdapter

注意：在编写Handler的时候要按照HandlerAdapter要求的规则去编写，这样适配器HandlerAdapter才可以正确的去执行Handler。

（4）处理器Handler（需要程序员开发）

（5）视图解析器 ViewResolver（不需要程序员开发）

作用：进行视图的解析，根据视图逻辑名解析成真正的视图（view）

（6）视图View（需要程序员开发jsp）

View是一个接口， 它的实现类支持不同的视图类型（jsp，freemarker，pdf等等）



## 39、springmvc常用注解 ##

@RequestMapping：用于处理请求 url 映射的注解，可用于类或方法上。用于类上，则表示类中的所有响应请求的方法都是以该地址作为父路径。

@RequestBody：注解实现接收http请求的json数据，将json转换为java对象。

@ResponseBody：注解实现将conreoller方法返回对象转化为json对象响应给客户。

`@RequestParam` 是从request里面拿取值模板：http://localhost:8080/springmvc/hello/101?param1=10&param2=20

类似于request.getParameter("name")

`@PathVariable` 将请求URL中的模板变量映射到功能处理方法的参数上

http://localhost:8080/springmvc/hello/1

## 40、springmvc流程图 ##

（1）用户发送请求至前端控制器DispatcherServlet；
（2） DispatcherServlet收到请求后，调用HandlerMapping处理器映射器，请求获取Handle；
（3）处理器映射器根据请求url找到具体的处理器，生成处理器对象及处理器拦截器(如果有则生成)一并返回给DispatcherServlet；
（4）DispatcherServlet 调用 HandlerAdapter处理器适配器；
（5）HandlerAdapter 经过适配调用 具体处理器(Handler，也叫后端控制器)；
（6）Handler执行完成返回ModelAndView；
（7）HandlerAdapter将Handler执行结果ModelAndView返回给DispatcherServlet；
（8）DispatcherServlet将ModelAndView传给ViewResolver视图解析器进行解析；
（9）ViewResolver解析后返回具体View；
（10）DispatcherServlet对View进行渲染视图（即将模型数据填充至视图中）
（11）DispatcherServlet响应用户。





## 41、@PathVariable和@RequestParam的区别 ##

@RequestParam 处理的是 http://localhost/paper/editpage?paperId=1

@PathVariable 处理的是 http://localhost/paper/editpage/1

@RequestParam：是从请求里面获取参数

@PathVariable：从路径里面去获取变量



## 42、Beanfactory与factorybean区别 ##

BeanFactory是提供了OC容器最基本的形式，给具体的IOC容器的实现提供了规范，FactoryBean可以说为IOC容器中Bean的实现提供了更加灵活的方式，FactoryBean在IOC容器的基础上给Bean的实现加上了一个简单工厂模式和装饰模式，我们可以在getObject()方法中灵活配置



## 43、Spring解决循环依赖 ##

1-Java中的循环依赖分两种，一种是构造器的循环依赖，另一种是属性的循环依赖。  构造器的循环依赖就是在构造器中有属性循环依赖，这种循环依赖没有什么解决办法，因为JVM虚拟机在对类进行实例化的时候，需先实例化构造器的参数，而由于循环引用这个参数无法提前实例化，故只能抛出错误。



2-Spring通过将实例化后的对象提前暴露给Spring容器中的singletonFactories，解决了循环依赖的问题。

3-简单了解三级缓存

singletonObjects 它是我们最熟悉的朋友，俗称“单例池”“容器”，缓存创建完成单例Bean的地方。
singletonFactories 映射创建Bean的原始工厂
earlySingletonObjects 映射Bean的早期引用，也就是说在这个Map里的Bean不是完整的，甚至还不能称之为“Bean”，只是一个Instance.
后两个Map其实是“垫脚石”级别的，只是创建Bean的时候，用来借助了一下，创建完成就清掉了。



4-流程

开始初始化a
调用A的构造，把A放入singletonFactories
开始注入A的依赖，发现A依赖对象B
开始初始化对象B
调用B的构造，把B放入singletonFactories
开始注入B的依赖，发现B依赖对象A
开始初始化对象A，发现A在singletonFactories里有，则直接获取A，
把A放入earlySingletonObjects，把A从singletonfactories删除
对象B的依赖注入完成
对象B创建完成，把B放入singletonObjects，
把B从earlySingletonObjects和singletonFactories中删除
对象B注入给A，继续注入A的其他依赖，直到A注入完成
对象A创建完成，把A放入singletonObjects，
把A从earlySingletonObjects和singletonFactories中删除
循环依赖处理结束，A和B都初始化和注入完成



## 45、springmvc启动流程 ##

\> 在 web.xml 文件中给 Spring MVC 的 Servlet 配置了 load-on-startup,所以程序启动的 > 时候会初始化 Spring MVC，在 HttpServletBean 中将配置的 contextConfigLocation > 属性设置到 Servlet 中，然后在 FrameworkServlet 中创建了 WebApplicationContext, > DispatcherServlet 根据 contextConfigLocation 配置的 classpath 下的 xml 文件初始化 了 > Spring MVC 总的组件。



## 46、springmvc运行流程 ##

\> 1.spring mvc 将所有的请求都提交给 DispatcherServlet,它会委托应用系统的其他模块 负责对请求 进行真正的处理工作。 

> 2.DispatcherServlet 查询一个或多个 HandlerMapping,找到处理请求的 Controller. 
>
> > 3.DispatcherServlet 请请求提交到目标 Controller 
> >
> > > 4.Controller 进行业务逻辑处理后，会返回一个 ModelAndView 
> > >
> > > > 5.Dispathcher 查询一个或多个 ViewResolver 视图解析器,找到 ModelAndView 对象指定 的视图对象 
> > > >
> > > > > 6.视图对象负责渲染返回给客户端。



## 47、spring事务实现方式 ##

\> 1、编码方式 > 所谓编程式事务指的是通过编码方式实现事务，即类似于 JDBC 编程实现事务管理。 > 2、声明式事务管理方式 > 声明式事务管理又有两种实现方式：基于 xml 配置文件的方式；另一个实在业务方法上 进行@Transaction 注解，将事务规则应用到业务逻辑中



## 48、spring aop实现原理 ##

\> Spring AOP 中的动态代理主要有两种方式，JDK 动态代理和 CGLIB 动态代理。JDK 动态代 理通过反射来接收被代理的类，并且要求被代理的类必须实现一个接口。JDK 动态代理的 核心是 InvocationHandler 接口和 Proxy 类。 

> 如果目标类没有实现接口，那么 Spring AOP 会选择使用 CGLIB 来动态代理目标类。CGLIB （Code Generation Library），是一个代码生成的类库，可以在运行时动态的生成某个类 的子类，注意，CGLIB 是通过继承的方式做的动态代理，因此如果某个类被标记为 final， 那么它是无法使用 CGLIB 做动态代理的。
>
> 





## 53、spring事务注解 何时出现回滚 ##

b. 事务注解@transactional； 

c. 默认情况下，如果在事务中抛出了未检查异常（继承⾃ RuntimeException 的异常）或者 Error，则 Spring 将回滚事务。 

d. @Transactional 只能应⽤到 public ⽅法才有效：只有@Transactional 注解应⽤到 public ⽅法，才能进⾏事务管理。这是因为 在使⽤ Spring AOP 代理时，Spring 在调⽤在图 1 中的 TransactionInterceptor 在⽬标⽅法执⾏前后进⾏拦截之前， DynamicAdvisedInterceptor（CglibAopProxy 的内部类）的的 intercept ⽅法或 JdkDynamicAopProxy 的 invoke ⽅法会间接调 ⽤ AbstractFallbackTransactionAttributeSource的 computeTransactionAttribute ⽅法。



## 55、事务出现问题解决方案 ##

1、脏读（新增或删除）：脏读就是指当⼀个事务正在访问数据，并且对数据进⾏了修改，⽽这种修改还没有提交到数据库 中，这时，另外⼀个事务也访问这个数据，然后使⽤了这个数据；



2、不可重复读（修改）：是指在⼀个事务内，多次读同⼀数据。在这个事务还没有结束时，另外⼀个事务也访问该同⼀数 据。那么，在第⼀个事务中的两次读数据之间，由于第⼆个事务的修改，那么第⼀个事务两次读到的的数据可能是不⼀样 的。这样在⼀个事务内两次读到的数据是不⼀样的，因此称为是不可重复读。（解决：只有在修改事务完全提交之后才可以 读取数据，则可以避免该问题）；



3、幻读（新增或删除）：是指当事务不是独⽴执⾏时发⽣的⼀种现象，例如第⼀个事务对⼀个表中的数据进⾏了修改，这 种修改涉及到表中的全部数据⾏。同时，第⼆个事务也修改这个表中的数据，这种修改是向表中插⼊⼀⾏新数据。那么，以 后就会发⽣操作第⼀个事务的⽤户发现表中还有没有修改的数据⾏，就好象发⽣了幻觉⼀样（解决：如果在操作事务完成数 据处理之前，任何其他事务都不可以添加新数据，则可避免该问题）。

案例说明

 ⽬前⼯资为1000的员⼯有10⼈。 

 1.事务1,读取所有⼯资为1000的员⼯。

2.这时事务2向employee表插⼊了⼀条员⼯记录，⼯资也为1000  

3.事务1再次读取所有⼯资为1000的员⼯ 共读取到了11条记录。





## 58、Spring单例实现原理 ##

 Spring 对 Bean 实例的创建是采用单例注册表的方式进行实现的，而这个注册表的缓存是 ConcurrentHashMap 对象。

## 59、Spring中bean为什么是单例的？

分析

如果一个bean被声明为单例的时候，在处理多次请求的时候在Spring容器里只实例化出一个bean，后续的请求都公用这个对象，这个对象会保存在一个map里面。当有请求来的时候会先从缓存(map)里查看有没有，有的话直接使用这个对象，没有的话才实例化一个新的对象，所以这是个单例的。但是对于原型(prototype)bean来说当每次请求来的时候直接实例化新的bean，没有缓存以及从缓存查的过程

结论

为了提高性能！！！从几个方面：1.少创建实例2.垃圾回收3.缓存快速获取

*优缺点*

*优点*

（1）减少了新生成实例的消耗 新生成实例消耗包括两方面，首先，Spring会通过反射或者cglib来生成bean实例这都是耗性能的操作，其次给对象分配内存也会涉及复杂算法
（2）减少jvm垃圾回收 由于不会给每个请求都新生成bean实例，所以自然回收的对象少了
（3）可以快速获取到bean 因为单例的获取bean操作除了第一次生成之外其余的都是从缓存里获取的所以很快

*缺点*

单例的bean一个很大的劣势就是他不能做到线程安全！！！，由于所有请求都共享一个bean实例，所以这个bean要是有状态的一个bean的话可能在并发场景下出现问题，而原型的bean则不会有这样问题（但也有例外，比如他被单例bean依赖），因为给每个请求都新创建实例

