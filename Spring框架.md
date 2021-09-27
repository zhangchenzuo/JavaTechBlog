- [什么是 Spring 框架?](#什么是-spring-框架)
  - [列举一些重要的Spring模块？](#列举一些重要的spring模块)
      - [3. @RestController vs @Controller](#3-restcontroller-vs-controller)
- [Spring IOC & AOP](#spring-ioc--aop)
  - [IoC](#ioc)
    - [Q: @Autowired存在多个同一类型的bean，怎么选择哪一个来注入？](#q-autowired存在多个同一类型的bean怎么选择哪一个来注入)
  - [AOP](#aop)
    - [Spring AOP 和 AspectJ AOP 有什么区别？](#spring-aop-和-aspectj-aop-有什么区别)
    - [具体的原理，实现了什么接口](#具体的原理实现了什么接口)
    - [AOP的概念](#aop的概念)
- [Spring bean](#spring-bean)
  - [说一下beanFactory吧](#说一下beanfactory吧)
  - [Spring 中的 bean 的作用域有哪些?](#spring-中的-bean-的作用域有哪些)
    - [Spring 中的单例 bean 的线程安全问题了解吗？](#spring-中的单例-bean-的线程安全问题了解吗)
    - [@Component 和 @Bean 的区别是什么？](#component-和-bean-的区别是什么)
    - [将一个类声明为Spring的 bean 的注解有哪些?](#将一个类声明为spring的-bean-的注解有哪些)
    - [Spring 中的 bean 生命周期?](#spring-中的-bean-生命周期)
    - [Q:如何解决bean的循环依赖问题？](#q如何解决bean的循环依赖问题)
  - [6. Spring MVC](#6-spring-mvc)
    - [6.1 说说自己对于 Spring MVC 了解?](#61-说说自己对于-spring-mvc-了解)
    - [SpringMVC 工作原理了解吗?](#springmvc-工作原理了解吗)
- [Spring 框架中用到了哪些设计模式？](#spring-框架中用到了哪些设计模式)
- [Spring 事务](#spring-事务)
    - [Spring 管理事务的方式有几种？](#spring-管理事务的方式有几种)
    - [8.2 Spring 事务中的隔离级别有哪几种?](#82-spring-事务中的隔离级别有哪几种)
    - [8.3 Spring 事务中哪几种事务传播行为?](#83-spring-事务中哪几种事务传播行为)
    - [8.4 @Transactional(rollbackFor = Exception.class)注解了解吗？](#84-transactionalrollbackfor--exceptionclass注解了解吗)
  - [spring事务的实现原理](#spring事务的实现原理)
- [Spring Boot](#spring-boot)
  - [简单介绍一下 Spring?有啥缺点? 为什么要有 SpringBoot?](#简单介绍一下-spring有啥缺点-为什么要有-springboot)
  - [说出使用 Spring Boot 的主要优点](#说出使用-spring-boot-的主要优点)
  - [什么是 Spring Boot Starters?](#什么是-spring-boot-starters)
  - [如何在 Spring Boot 应用程序中使用 Jetty 而不是 Tomcat?](#如何在-spring-boot-应用程序中使用-jetty-而不是-tomcat)
    - [介绍一下@SpringBootApplication 注解](#介绍一下springbootapplication-注解)
  - [Spring Boot 的自动配置是如何实现的?](#spring-boot-的自动配置是如何实现的)
  - [常用的注解有哪些？](#常用的注解有哪些)
    - [@Autowired、@Resource和@Inject注解的作用？](#autowiredresource和inject注解的作用)
  - [Spring Boot 常用的两种配置文件](#spring-boot-常用的两种配置文件)
  - [Spring Boot 常用的读取配置文件的方法有哪些？](#spring-boot-常用的读取配置文件的方法有哪些)
  - [Spring Boot 加载配置文件的优先级了解么？](#spring-boot-加载配置文件的优先级了解么)
    - [16. Spring Boot 如何做请求参数校验？](#16-spring-boot-如何做请求参数校验)
      - [16.1. 校验注解](#161-校验注解)
      - [16.2. 验证请求体(RequestBody)](#162-验证请求体requestbody)
      - [16.3. 验证请求参数(Path Variables 和 Request Parameters)](#163-验证请求参数path-variables-和-request-parameters)
  - [如何使用 Spring Boot 实现全局异常处理？](#如何使用-spring-boot-实现全局异常处理)
  - [说一下 Spring Boot 自动装配原理？](#说一下-spring-boot-自动装配原理)
    - [`EnableAutoConfiguration`：实现自动装配的核心](#enableautoconfiguration实现自动装配的核心)
      - [AutoConfigurationImportSelector.class 加载自动装配类](#autoconfigurationimportselectorclass-加载自动装配类)
    - [如何写一个自己的starter](#如何写一个自己的starter)
  - [spring 中IoC容器源码](#spring-中ioc容器源码)
  - [bean的生命周期](#bean的生命周期)
- [拦截器与过滤器](#拦截器与过滤器)
  - [什么是serverlet](#什么是serverlet)

>spring与spring boot问答
这部分内容没有很系统的学习，因此打算直接从面经学习。

# 什么是 Spring 框架?

Spring 是一种轻量级开发框架，提高开发效率以及系统的可维护性。。

Spring 官网列出的 Spring 的 6 个特征:

- **核心技术** ：**依赖注入(DI)，AO**P，事件(events)，资源，i18n，验证，数据绑定，类型转换，SpEL。
- **测试** ：模拟对象，TestContext框架，Spring MVC 测试，WebTestClient。
- **数据访问** ：事务，DAO支持，JDBC，ORM，编组XML。
- **Web支持** : Spring MVC和Spring WebFlux Web框架。
- **集成** ：远程处理，JMS，JCA，JMX，电子邮件，任务，调度，缓存。
- **语言** ：Kotlin，Groovy，动态语言。


* Q: spring和spirng boot的区别
A：spring虽然是一个轻量的开发，但是需要配置xml等依赖关系。spring boot 通过加入了自动配置，帮我们省了这个麻烦。


## 列举一些重要的Spring模块？

![Spring主要模块](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-6/Spring主要模块.png)

- **Spring Core：** 基础,可以说 Spring 其他所有的功能都需要依赖于该类库。主要提供 IoC 依赖注入功能。
- **Spring AOP** ：提供了面向切面的编程实现。
- **Spring JDBC** : Java数据库连接。
- Spring JMS ：Java消息服务。
- Spring ORM : 用于支持Hibernate等ORM工具。
- Spring Web : 为创建Web应用程序提供支持。
- Spring Test : 提供了对 JUnit 和 TestNG 测试的支持。
- Spring Aspects ： 该模块为与AspectJ的集成提供支持。
  
#### 3. @RestController vs @Controller

**`Controller` 返回一个页面**

**`@RestController` 返回JSON 或 XML 形式数据**。等于`@Controller +@ResponseBody`

单独使用 `@Controller` 不加 `@ResponseBody`的话一般使用在要返回一个视图的情况，这种情况属于比较传统的Spring MVC 的应用，对应于前后端不分离的情况。

但`@RestController`只返回对象，对象数据直接以 JSON 或 XML 形式写入 HTTP 响应(Response)中，这种情况属于 RESTful Web服务，这也是目前日常开发所接触的最常用的情况（前后端分离）。

# Spring IOC & AOP

* 谈谈自己对于 Spring IoC 和 AOP 的理解

## IoC

IoC（控制反转）是一种**设计思想**，就是 **将原本在程序中手动创建对象的控制权，交由Spring框架来管理。**  IoC 在其他语言中也有应用，并非 Spring 特有。 **IoC 容器是 Spring 用来实现 IoC 的载体,IoC 容器实际上就是个Map（key，value）,Map 中存放的是各种对象。**

**控制反转**就是**依赖倒置原则**的一种代码设计的思路。具体采用的方法就是所谓的**依赖注入**。**所谓依赖注入，就是把底层类作为参数传入上层类，实现上层类对下层类的“控制”**。**依赖注入的方法还包括构造参数传入和setter传递以及接口传递。**

将对象之间的相互依赖关系交给 IoC 容器来管理，并由 IoC 容器完成对象的注入。这样可以很大程度上简化应用的开发，把应用从复杂的依赖关系中解放出来。  **IoC 容器就像是一个工厂一样，当我们需要创建一个对象的时候，只需要配置好配置文件/注解即可，完全不用考虑对象是如何被创建出来的。** 如果利用 IoC 的话，你只需要配置好，然后在需要的地方引用就行了，这大大增加了项目的可维护性且降低了开发难度。

Spring 我们一般通过 XML 文件来配置 Bean，SpringBoot 中使用注解配置。

推荐阅读：https://www.zhihu.com/question/23277575/answer/169698662

**Spring IoC的初始化过程：** 

![Spring IoC的初始化过程](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-7/SpringIOC初始化过程.png)

IoC源码阅读

- https://javadoop.com/post/spring-ioc
  

### Q: @Autowired存在多个同一类型的bean，怎么选择哪一个来注入？

当容器中的bean为单例时，@Autowire为ByType的方式注入，被注入的成员的名称可以任意取名。

当容器中的bean存在多个的情况下，@Autowire为ByName的方式注入，ByName是将bean单例池中的key（bean的名字）与被注入的成员变量的名称匹配，而不是与被注入的成员变量的类型匹配


## AOP

AOP(面向切面编程)能够将那些与业务无关，**却为业务模块所共同调用的逻辑或责任（例如事务处理、日志管理、权限控制等）封装起来**，便于**减少系统的重复代码**，**降低模块间的耦合度**，并**有利于未来的可拓展性和可维护性**。

**Spring AOP就是基于动态代理的**，如果要代理的对象，实现了某个接口，那么Spring AOP会使用**JDK Proxy**，去创建代理对象，而对于没有实现接口的对象，就无法使用 JDK Proxy 去进行代理了，这时候Spring AOP会使用**Cglib** ，这时候Spring AOP会使用 **Cglib** 生成一个被代理对象的子类来作为代理，如下图所示：

![SpringAOPProcess](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-6/SpringAOPProcess.jpg)

当然你也可以使用 AspectJ ,Spring AOP 已经集成了AspectJ  ，AspectJ  应该算的上是 Java 生态系统中最完整的 AOP 框架了。

使用 AOP 之后我们可以把一些通用功能抽象出来，在需要用到的地方直接使用即可，这样大大简化了代码量。我们需要增加新功能时也方便，这样也提高了系统扩展性。日志功能、事务管理等等场景都用到了 AOP 。

### Spring AOP 和 AspectJ AOP 有什么区别？

**Spring AOP 属于运行时增强，而 AspectJ 是编译时增强。** Spring AOP 基于代理(Proxying)，而 AspectJ 基于字节码操作(Bytecode Manipulation)。AspectJ是静态代理。

 Spring AOP 已经集成了 AspectJ  ，AspectJ  应该算的上是 Java 生态系统中最完整的 AOP 框架了。AspectJ  相比于 Spring AOP 功能更加强大，但是 Spring AOP 相对来说更简单，

如果我们的切面比较少，那么两者性能差异不大。但是，当切面太多的话，最好选择 AspectJ ，它比Spring AOP 快很多。


### 具体的原理，实现了什么接口

实现了**BeanPostProcessor**接口，那就意味着这个类在spring加载实例化前会调用**postProcessAfterInitialization**方法。每个bean的实例化之前都会先经过AbstractAutoProxyCreator类的postProcessAfterInitialization（）这个方法

### AOP的概念
- Aspect（切面）： Aspect 声明类似于 Java 中的类声明，在 Aspect 中会包含着一些 Pointcut 以及相应的 Advice。

- Joint point（连接点）：表示在程序中明确定义的点，典型的包括方法调用，对类成员的访问以及异常处理程序块的执行等等，它自身还可以嵌套其它 joint point。

- Pointcut（切点）：表示一组 joint point，这些 joint point 或是通过逻辑关系组合起来，或是通过通配、正则表达式等方式集中起来，它定义了相应的 Advice 将要发生的地方。

- Advice（增强）：Advice 定义了在 Pointcut 里面定义的程序点具体要做的操作，它通过 before、after 和 around 来区别是在每个 joint point 之前、之后还是代替执行的代码。

- Target（目标对象）：织入 Advice 的目标对象。

- Weaving（织入）：将 Aspect 和其他对象连接起来, 并创建 Adviced object 的过程


# Spring bean

## 说一下beanFactory吧
BeanFactory和ApplicationContext就是spring框架的两个IOC容器。

Spring 通过一个配置文件来描述 Bean 及 Bean 之间的依赖关系，**利用 Java 的反射功能实例化 Bean 并建立 Bean 之间的依赖关系** 。**BeanFactory 是类的通用工厂，它可以创建并管理各种类的对象**，这些类就是 POJO，Spring 称这些被创建并管理的类对象为 Bean。ApplicationContext 由 BeanFactory 派生而来，提供了很多实际应用的功能 。 在 BeanFactory 中，很多功能需要以编程的方式实现，而在 A**pplicationContext 中则可以通过配置的方式来实现**。初始化 ApplicationContext 时，根据配置文件的所在路径，来选择 ApplicationContext 的实现类。

ApplicationContext 的主要实现类是：
* ClassPathXmlApplicationContext - 从类路径加载配置文件。 

factoryBean是一个可以创建工厂的bean。

## Spring 中的 bean 的作用域有哪些?

- singleton : 唯一 bean 实例，Spring 中的 bean 默认都是单例的。
- prototype : 每次请求都会创建一个新的 bean 实例。
- request : 每一次HTTP请求都会产生一个新的bean，该bean仅在当前HTTP request内有效。
- session : 每一次HTTP请求都会产生一个新的 bean，该bean仅在当前 HTTP session 内有效。
- global-session： 全局session作用域，仅仅在基于portlet的web应用中才有意义，Spring5已经没有了。Portlet是能够生成语义代码(例如：HTML)片段的小型Java Web插件。它们基于portlet容器，可以像servlet一样处理HTTP请求。但是，与 servlet 不同，每个 portlet 都有不同的会话

[多例也未必是安全的](https://blog.csdn.net/shmily_lsl/article/details/81207080?utm_medium=distribute.pc_relevant_bbs_down.none-task-blog-baidujs-1.nonecase&depth_1-utm_source=distribute.pc_relevant_bbs_down.none-task-blog-baidujs-1.nonecase)
* **Q: 既然有单例存在安全问题，为什么默认是单例？**
> 1. 减少了新生成实例的消耗
新生成实例消耗包括两方面，第一，spring会通过反射或者cglib来生成bean实例这都是耗性能的操作，其次给对象分配内存也会涉及复杂算法。

> 2. 减少jvm垃圾回收
由于不会给每个请求都新生成bean实例，所以自然回收的对象少了。

> 3. 可以快速获取到bean
因为单例的获取bean操作除了第一次生成之外其余的都是从缓存里获取的所以很快。


### Spring 中的单例 bean 的线程安全问题了解吗？

的确是存在安全问题的。因为，当多个线程操作同一个对象的时候，对这个对象的成员变量的写操作会存在线程安全问题。

但是，一般情况下，我们常用的 `Controller`、`Service`、`Dao` 这些 Bean 是无状态的。无状态的 Bean 不能保存数据，因此是线程安全的。(有状态对象：有实例变量可以标志其对象所处的状态。（有实例变量的对象，有存储数据能力）- 白话：有属性的对象)

常见的有 2 种解决办法：

1. 在类中定义一个 `ThreadLocal` 成员变量，将需要的可变成员变量保存在  `ThreadLocal`  中（推荐的一种方式）。
2. 改变 Bean 的作用域为 “prototype”：每次请求都会创建一个新的 bean 实例，自然不会存在线程安全问题。


### @Component 和 @Bean 的区别是什么？

1. 作用对象不同: `@Component` 注解作用于类，而`@Bean`注解作用于方法。
2. `@Component`通常是通过类路径扫描来自动侦测以及自动装配到Spring容器中（我们可以使用 `@ComponentScan` 注解定义要扫描的路径从中找出标识了需要装配的类自动装配到 Spring 的 bean 容器中）。`@Bean` 注解通常是我们在标有该注解的方法中定义产生这个 bean,`@Bean`告诉了Spring这是某个类的示例，当我需要用它的时候还给我。
3. `@Bean` 注解比 `Component` 注解的自定义性更强，而且很多地方我们只能通过 `@Bean` 注解来注册bean。**比如当我们引用第三方库中的类需要装配到 `Spring`容器时，则只能通过 `@Bean`来实现。**

`@Bean`注解使用示例：

```java
@Configuration
public class AppConfig {
    @Bean
    public TransferService transferService() {
        return new TransferServiceImpl();
    }

}
```
###  将一个类声明为Spring的 bean 的注解有哪些?

我们一般使用 `@Autowired` 注解自动装配 bean，要想把**类**标识成可用于 `@Autowired` 注解自动装配的 bean 的类,采用以下注解可实现：

- `@Component` ：通用的注解，可标注任意类为 `Spring` 组件。如果一个Bean不知道属于哪个层，可以使用`@Component` 注解标注。
- `@Repository` : 对应持久层即 Dao 层，主要用于数据库相关操作。
- `@Service` : 对应服务层，主要涉及一些复杂的逻辑，需要用到 Dao层。
- `@Controller` : 对应 Spring MVC 控制层，主要用于接受用户请求并调用 Service 层返回数据给前端页面。

### Spring 中的 bean 生命周期?

这部分网上有很多文章都讲到了，下面的内容整理自：<https://yemengying.com/2016/07/14/spring-bean-life-cycle/> ，除了这篇文章，再推荐一篇很不错的文章 ：<https://www.cnblogs.com/zrtqsk/p/3735273.html> 。

- Bean 容器找到配置文件中 Spring Bean 的定义。
- Bean 容器利用 Java Reflection API 创建一个Bean的实例。
- 如果涉及到一些属性值 利用 `set()`方法设置一些属性值。
- 如果 Bean 实现了 `BeanNameAware` 接口，调用 `setBeanName()`方法，传入Bean的名字。
- 如果 Bean 实现了 `BeanClassLoaderAware` 接口，调用 `setBeanClassLoader()`方法，传入 `ClassLoader`对象的实例。
- 与上面的类似，如果实现了其他 `*.Aware`接口，就调用相应的方法。
- 如果有和加载这个 Bean 的 Spring 容器相关的 `BeanPostProcessor` 对象，执行`postProcessBeforeInitialization()` 方法
- 如果Bean实现了`InitializingBean`接口，执行`afterPropertiesSet()`方法。
- 如果 Bean 在配置文件中的定义包含  init-method 属性，执行指定的方法。
- 如果有和加载这个 Bean的 Spring 容器相关的 `BeanPostProcessor` 对象，执行`postProcessAfterInitialization()` 方法
- 当要销毁 Bean 的时候，如果 Bean 实现了 `DisposableBean` 接口，执行 `destroy()` 方法。
- 当要销毁 Bean 的时候，如果 Bean 在配置文件中的定义包含 destroy-method 属性，执行指定的方法。


这里面BeanPostProcessor这个类是SpringAOP的关键，可以用于创建代理。 

![Spring Bean 生命周期](http://my-blog-to-use.oss-cn-beijing.aliyuncs.com/18-9-17/5496407.jpg)


### Q:如何解决bean的循环依赖问题？
bean的循环依赖是因为，实例化时候可以分为三个大的步骤，首先是实例化bean，然后是填充bean的属性，最后是bean的初始化。在一二步之间会出现循环依赖问题。

为了解决（单例）的循环依赖问题，采用了**三级缓存**策略。简单来说就是，先让最底层对象完成初始化，通过三级缓存与二级缓存提前曝光创建中的 Bean，让其他 Bean 率先完成初始化。
```
/** 一级缓存：用于存放完全初始化好的 bean **/
private final Map<String, Object> singletonObjects = new ConcurrentHashMap<String, Object>(256);

/** 二级缓存：存放原始的 bean 对象（尚未填充属性），用于解决循环依赖 */
private final Map<String, Object> earlySingletonObjects = new HashMap<String, Object>(16);

/** 三级级缓存：存放 bean 工厂对象，用于解决循环依赖 */
private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<String, ObjectFactory<?>>(16);

/**
bean 的获取过程：先从一级获取，失败再从二级、三级里面获取

创建中状态：是指对象已经 new 出来了但是所有的属性均为 null 等待被 init
*/
```
检测循环依赖的过程如下：

    * A 创建过程中需要 B，于是 A 将自己放到三级缓里面 ，去实例化 B (synchronized (this.singletonObjects)这里对A加锁了，在线程安全的环境下去创建B）。
    * B 实例化的时候发现需要 A，于是 B 先查一级缓存，没有，再查二级缓存，还是没有，再查三级缓存，找到了！
      * 然后把三级缓存里面的这个 A 放到二级缓存里面，并删除三级缓存里面的 A 
      * B 顺利初始化完毕，将自己放到一级缓存里面（此时B里面的A依然是创建中状态）
    * 然后回来接着创建 A，此时 B 已经创建结束，直接从一级缓存里面拿到 B ，然后完成创建，并将自己放到一级缓存里面


使用构造器注入其他 Bean 的实例是无法解决的。需要使用`@Autowire`。

## 6. Spring MVC

### 6.1 说说自己对于 Spring MVC 了解?

MVC 是一种设计模式,Spring MVC 是一款很优秀的 MVC 框架。Spring MVC 可以帮助我们进行更简洁的Web层的开发，并且它天生与 Spring 框架集成。Spring MVC 下我们一般把**后端项目分为 Service层（处理业务）、Dao层（数据库操作）、Model层（实体类）、Controller层(控制层，返回数据给前台页面)。**

**Spring MVC 的简单原理图如下：**

![](http://my-blog-to-use.oss-cn-beijing.aliyuncs.com/18-10-11/60679444.jpg)

### SpringMVC 工作原理了解吗?

**原理如下图所示：**
![SpringMVC运行原理](http://my-blog-to-use.oss-cn-beijing.aliyuncs.com/18-10-11/49790288.jpg)

上图的一个笔误的小问题：Spring MVC 的入口函数也就是前端控制器 `DispatcherServlet` 的作用是接收请求，响应结果。

**流程说明（重要）：**

1. 客户端（浏览器）发送请求，直接请求到 `DispatcherServlet`。
2. `DispatcherServlet` 根据请求信息调用 `HandlerMapping`，解析请求对应的 `Handler`。
3. 解析到对应的 `Handler`（也就是我们平常说的 `Controller` 控制器）后，开始由 `HandlerAdapter` 适配器处理。
4. `HandlerAdapter` 会根据 `Handler `来调用真正的处理器来处理请求，并处理相应的业务逻辑。
5. 处理器处理完业务后，会返回一个 `ModelAndView` 对象，`Model` 是返回的数据对象，`View` 是个逻辑上的 `View`。
6. `ViewResolver` 会根据逻辑 `View` 查找实际的 `View`。
7. `DispaterServlet` 把返回的 `Model` 传给 `View`（视图渲染）。
8. 把 `View` 返回给请求者（浏览器）

说点人话就是：首先用户向服务器发送的请求需要先在前端被捕获（DispatcherServlet），然后对请求的URL进行解析，得到URI。并判断URI的映射，返回一个HandlerAdapter。提取request中参数，填入Handler，执行具体的方法。完成以后，向DispatcherServlet返回一个ModelandView对象，视图解析器在进行解析等到真正的对象，进行渲染。

各个组件的具体作用：
1. 前端控制器`DispatcherServlet`,由框架提供

作用：接收请求，响应结果，相当于转发器，中央处理器。有了dispatcherServlet减少了其它组件之间的耦合度。

用户请求到达前端控制器，它就相当于mvc模式中的c，dispatcherServlet是整个流程控制的中心，由它调用其它组件处理用户的请求，dispatcherServlet的存在降低了组件之间的耦合性。

2. 处理器映射器HandlerMapping,由框架提供

作用：根据请求的url查找Handler

HandlerMapping负责根据用户请求找到Handler即处理器，springmvc提供了不同的映射器实现不同的映射方式，例如：配置文件方式，实现接口方式，注解方式等。

3. 处理器适配器HandlerAdapter

作用：按照特定规则（HandlerAdapter要求的规则）去执行Handler。通过HandlerAdapter对处理器进行执行，这是适配器模式的应用，通过扩展适配器可以对更多类型的处理器进行执行。

4、处理器Handler(需要工程师开发)

注意：编写Handler时按照HandlerAdapter的要求去做，这样适配器才可以去正确执行Handler。

Handler 是继DispatcherServlet前端控制器的后端控制器，在DispatcherServlet的控制下Handler对具体的用户请求进行处理。

由于Handler涉及到具体的用户业务请求，所以一般情况需要工程师根据业务需求开发Handler。

5、视图解析器View resolver,由框架提供

作用：进行视图解析，根据逻辑视图名解析成真正的视图（view）

View Resolver负责将处理结果生成View视图

6、视图View(需要工程师开发页面渲染)

View是一个接口，实现类支持不同的View类型（jsp、freemarker、pdf...）

简单了解下MVC：常说的 MVC 是一种设计模式，并不是SpringMVC框架（只是该框架支持这种MVC模式）

# Spring 框架中用到了哪些设计模式？

关于下面一些设计模式的详细介绍，可以看笔主前段时间的原创文章[《面试官:“谈谈Spring中都用到了那些设计模式?”。》](https://mp.weixin.qq.com/s?__biz=Mzg2OTA0Njk0OA==&mid=2247485303&idx=1&sn=9e4626a1e3f001f9b0d84a6fa0cff04a&chksm=cea248bcf9d5c1aaf48b67cc52bac74eb29d6037848d6cf213b0e5466f2d1fda970db700ba41&token=255050878&lang=zh_CN#rd) 。

- **工厂设计模式** : Spring使用工厂模式通过 `BeanFactory`、`ApplicationContext` 创建 bean 对象。
- **代理设计模式** : Spring AOP 功能的实现。
- **单例设计模式** : Spring 中的 Bean 默认都是单例的。
- **模板方法模式** : Spring 中 `jdbcTemplate`、`hibernateTemplate` 等以 Template 结尾的对数据库操作的类，它们就使用到了模板模式。
- **包装器设计模式** : 我们的项目需要连接多个数据库，而且不同的客户在每次访问中根据需要会去访问不同的数据库。这种模式让我们可以根据客户的需求能够动态切换不同的数据源。
- **观察者模式:** Spring 事件驱动模型就是观察者模式很经典的一个应用。
- **适配器模式** :Spring AOP 的增强或通知(Advice)使用到了适配器模式、spring MVC 中也是用到了适配器模式适配`Controller`。
- ......

# Spring 事务

### Spring 管理事务的方式有几种？

1. 编程式事务，在代码中硬编码。(不推荐使用)
2. 声明式事务，在配置文件中配置（推荐使用）

**声明式事务又分为两种：**

1. 基于XML的声明式事务
2. 基于注解的声明式事务

### 8.2 Spring 事务中的隔离级别有哪几种?

**TransactionDefinition 接口中定义了五个表示隔离级别的常量：**

- **TransactionDefinition.ISOLATION_DEFAULT:**  使用后端数据库默认的隔离级别，Mysql 默认采用的 REPEATABLE_READ隔离级别 Oracle 默认采用的 READ_COMMITTED隔离级别.
- **TransactionDefinition.ISOLATION_READ_UNCOMMITTED:** 最低的隔离级别，允许读取尚未提交的数据变更，**可能会导致脏读、幻读或不可重复读**
- **TransactionDefinition.ISOLATION_READ_COMMITTED:**   允许读取并发事务已经提交的数据，**可以阻止脏读，但是幻读或不可重复读仍有可能发生**
- **TransactionDefinition.ISOLATION_REPEATABLE_READ:**  对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，**可以阻止脏读和不可重复读，但幻读仍有可能发生。**
- **TransactionDefinition.ISOLATION_SERIALIZABLE:**   最高的隔离级别，完全服从ACID的隔离级别。所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，**该级别可以防止脏读、不可重复读以及幻读**。但是这将严重影响程序的性能。通常情况下也不会用到该级别。

### 8.3 Spring 事务中哪几种事务传播行为?

**支持当前事务的情况：**

- **TransactionDefinition.PROPAGATION_REQUIRED：** 如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务。
- **TransactionDefinition.PROPAGATION_SUPPORTS：** 如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。
- **TransactionDefinition.PROPAGATION_MANDATORY：** 如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常。（mandatory：强制性）

**不支持当前事务的情况：**

- **TransactionDefinition.PROPAGATION_REQUIRES_NEW：** 创建一个新的事务，如果当前存在事务，则把当前事务挂起。
- **TransactionDefinition.PROPAGATION_NOT_SUPPORTED：** 以非事务方式运行，如果当前存在事务，则把当前事务挂起。
- **TransactionDefinition.PROPAGATION_NEVER：** 以非事务方式运行，如果当前存在事务，则抛出异常。

**其他情况：**

- **TransactionDefinition.PROPAGATION_NESTED：** 如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行；如果当前没有事务，则该取值等价于TransactionDefinition.PROPAGATION_REQUIRED。

### 8.4 @Transactional(rollbackFor = Exception.class)注解了解吗？

当`@Transactional`注解作用于类上时，该类的所有 public 方法将都具有该类型的事务属性。

在`@Transactional`注解中如果不配置`rollbackFor`属性,那么事务只会在遇到`RuntimeException`的时候才会回滚,加上`rollbackFor=Exception.class`,可以让事务在遇到非运行时异常时也回滚。

## spring事务的实现原理
https://www.cnblogs.com/insaneXs/p/13638034.html

本质是通过Aop实现的。spring本身也提供了事务的API，包括事务管理器——PlatformTransactionManager，事务状态——TransactionStatus，事务属性的定义——TransactionDefinition

# Spring Boot
## 简单介绍一下 Spring?有啥缺点? 为什么要有 SpringBoot?

Spring 为企业级 Java 开发提供了一种相对简单的方法，通过 **依赖注入**实现了**控制反转** 和 **面向切面编程**。但是spring的配置比较麻烦，需要借助xml进行配置。并且相关库的依赖也需要自己管理，不同库之间的版本冲突也非常常见。

Spring Boot 旨在简化 Spring 开发，减少配置文件。

羊与牧羊人的关系。
## 说出使用 Spring Boot 的主要优点

Spring Boot 不需要编写大量样板代码、XML 配置和注释。spring boot约定大于配置，以减少开发工作（默认配置可以修改）。


## 什么是 Spring Boot Starters?

Spring Boot Starters 是一系列依赖关系的集合，因为它的存在，项目的依赖之间的关系对我们来说变的更加简单了。

但是，有了 Spring Boot Starters 我们只需要一个只需添加一个**spring-boot-starter-web**一个依赖就可以了，这个依赖包含的字依赖中包含了我们开发 REST 服务需要的所有依赖。

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```
## 如何在 Spring Boot 应用程序中使用 Jetty 而不是 Tomcat?

Spring Boot （`spring-boot-starter-web`）使用 Tomcat 作为默认的嵌入式 servlet 容器, 如果你想使用 Jetty 的话只需要修改`pom.xml`(Maven)或者`build.gradle`(Gradle)就可以了。

**Maven:**

```xml
<!--从Web启动器依赖中排除Tomcat-->
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
	<exclusions>
		<exclusion>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-tomcat</artifactId>
		</exclusion>
	</exclusions>
</dependency>
<!--添加Jetty依赖-->
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-jetty</artifactId>
</dependency>
```


### 介绍一下@SpringBootApplication 注解

```java
package org.springframework.boot.autoconfigure;
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = {
		@Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
   ......
}
```

```java
package org.springframework.boot;
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Configuration
public @interface SpringBootConfiguration {

}
```

可以看出大概可以把 `@SpringBootApplication`看作是 `@Configuration`、`@EnableAutoConfiguration`、`@ComponentScan` 注解的集合。根据 SpringBoot 官网，这三个注解的作用分别是：

- `@EnableAutoConfiguration`：启用 SpringBoot 的自动配置机制
- `@ComponentScan`： 扫描被`@Component` (`@Service`,`@Controller`)注解的 bean，注解默认会扫描该类所在的包下所有的类。
- `@Configuration`：允许在上下文中注册额外的 bean 或导入其他配置类

## Spring Boot 的自动配置是如何实现的?

这个是因为`@SpringBootApplication`注解的原因，在上一个问题中已经提到了这个注解。我们知道 `@SpringBootApplication`看作是 `@Configuration`、`@EnableAutoConfiguration`、`@ComponentScan` 注解的集合。

- `@EnableAutoConfiguration`：启用 SpringBoot 的自动配置机制
- `@ComponentScan`： 扫描被`@Component` (`@Service`,`@Controller`)注解的 bean，注解默认会扫描该类所在的包下所有的类。
- `@Configuration`：允许在上下文中注册额外的 bean 或导入其他配置类

`@EnableAutoConfiguration`是启动自动配置的关键，源码如下(建议自己打断点调试，走一遍基本的流程)：

```java
import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Inherited;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
import org.springframework.context.annotation.Import;

@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import({AutoConfigurationImportSelector.class})
public @interface EnableAutoConfiguration {
    String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";

    Class<?>[] exclude() default {};

    String[] excludeName() default {};
}
```

`@EnableAutoConfiguration` 注解通过 Spring 提供的 `@Import` 注解导入了`AutoConfigurationImportSelector`类（`@Import` 注解可以导入配置类或者 Bean 到当前类中）。

`AutoConfigurationImportSelector`类中`getCandidateConfigurations`方法会将所有自动配置类的信息以 List 的形式返回。这些配置信息会被 Spring 容器作 bean 来管理。

```java
	protected List<String> getCandidateConfigurations(AnnotationMetadata metadata, AnnotationAttributes attributes) {
		List<String> configurations = SpringFactoriesLoader.loadFactoryNames(getSpringFactoriesLoaderFactoryClass(),
				getBeanClassLoader());
		Assert.notEmpty(configurations, "No auto configuration classes found in META-INF/spring.factories. If you "
				+ "are using a custom packaging, make sure that file is correct.");
		return configurations;
	}
```

自动配置信息有了，那么自动配置还差什么呢？

`@Conditional` 注解。`@ConditionalOnClass`(指定的类必须存在于类路径下),`@ConditionalOnBean`(容器中是否有指定的 Bean)等等都是对`@Conditional`注解的扩展。

拿 Spring Security 的自动配置举个例子:`SecurityAutoConfiguration`中导入了`WebSecurityEnablerConfiguration`类，`WebSecurityEnablerConfiguration`源代码如下：

```java
@Configuration
@ConditionalOnBean(WebSecurityConfigurerAdapter.class)
@ConditionalOnMissingBean(name = BeanIds.SPRING_SECURITY_FILTER_CHAIN)
@ConditionalOnWebApplication(type = ConditionalOnWebApplication.Type.SERVLET)
@EnableWebSecurity
public class WebSecurityEnablerConfiguration {

}
```

`WebSecurityEnablerConfiguration`类中使用`@ConditionalOnBean`指定了容器中必须还有`WebSecurityConfigurerAdapter` 类或其实现类。所以，一般情况下 Spring Security 配置类都会去实现 `WebSecurityConfigurerAdapter`，这样自动将配置就完成了。

## 常用的注解有哪些？

**Spring Bean 相关：**

- `@Autowired` : 自动导入对象到类中，被注入进的类同样要被 Spring 容器管理。
- `@RestController` : `@RestController`注解是`@Controller和`@`ResponseBody`的合集,表示这是个控制器 bean,并且是将函数的返回值直 接填入 HTTP 响应体中,是 REST 风格的控制器。
- `@Component` ：通用的注解，可标注任意类为 `Spring` 组件。如果一个 Bean 不知道属于哪个层，可以使用`@Component` 注解标注。
- `@Repository` : 对应持久层即 Dao 层，主要用于数据库相关操作。
- `@Service` : 对应服务层，主要涉及一些复杂的逻辑，需要用到 Dao 层。
- `@Controller` : 对应 Spring MVC 控制层，主要用于接受用户请求并调用 Service 层返回数据给前端页面。

### @Autowired、@Resource和@Inject注解的作用？ 
@Resource和@Autowired注解都是用来实现依赖注入的。只是@AutoWried按by type自动注入，而@Resource默认按byName自动注入。@Inject和jsr330规范提供的，而@Autowired是spring提供的。@Resource则是jsr250的实现，这是多年前的规范。

**处理常见的 HTTP 请求类型：**

- `@GetMapping` : GET 请求、
- `@PostMapping` : POST 请求。
- `@PutMapping` : PUT 请求。
- `@DeleteMapping` : DELETE 请求。

**前后端传值：**

- `@RequestParam`以及`@Pathvairable ：@PathVariable用于获取路径参数，@RequestParam用于获取查询参数。`
- `@RequestBody` ：用于读取 Request 请求（可能是 POST,PUT,DELETE,GET 请求）的 body 部分并且 Content-Type 为 `application/json` 格式的数据，接收到数据之后会自动将数据绑定到 Java 对象上去。系统会使用`HttpMessageConverter`或者自定义的`HttpMessageConverter`将请求的 body 中的 json 字符串转换为 java 对象。

详细介绍可以查看这篇文章：[《Spring/Spring Boot 常用注解总结》](https://snailclimb.gitee.io/javaguide/#/./docs/system-design/framework/spring/SpringBoot+Spring%E5%B8%B8%E7%94%A8%E6%B3%A8%E8%A7%A3%E6%80%BB%E7%BB%93?id=_21-autowired) 。

## Spring Boot 常用的两种配置文件

我们可以通过 `application.properties`或者 `application.yml` 对 Spring Boot 程序进行简单的配置。如果，你不进行配置的话，就是使用的默认配置。

## Spring Boot 常用的读取配置文件的方法有哪些？

我们要读取的配置文件`application.yml` 内容如下：

```yaml
mq:
  nameserver:
    addr: 123.56.52.77:9876
  topicname: stock
```

1. ### 通过 `@value` 读取比较简单的配置信息

使用 `@Value("${property}")` 读取比较简单的配置信息：

```java
    @Value("${mq.nameserver.addr}")
    private String nameAddr;
    @Value("${mq.topicname}")
    private String topicName;
```

> **需要注意的是 `@value`这种方式是不被推荐的，Spring 比较建议的是下面几种读取配置信息的方式。**

2. 通过`@ConfigurationProperties`读取并与 bean 绑定

```java
@Component
@ConfigurationProperties(prefix = "library")
class LibraryProperties {
    private String location;
    private List<Book> books;
    static class Book {
        String name;
        String description;
    }
}
```


## Spring Boot 加载配置文件的优先级了解么？

Spring 读取配置文件也是有优先级的，直接上图：

![](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-11/read-config-properties-priority.jpg)

> 常用的 Bean 映射工具有哪些？

MapStruct、ModelMapper、Dozer、Orika、JMapper 是 5 种常用的 Bean 映射工具。

综合日常使用情况和相关测试数据，个人感觉 MapStruct、ModelMapper 这两个 Bean 映射框架是最佳选择。

> Spring Boot 如何监控系统实际运行状况？

我们可以使用 Spring Boot Actuator 来对 Spring Boot 项目进行简单的监控。

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

集成了这个模块之后，你的 Spring Boot 应用程序就自带了一些开箱即用的获取程序运行时的内部状态信息的 API。

比如通过 GET 方法访问 `/health` 接口，你就可以获取应用程序的健康指标。

### 16. Spring Boot 如何做请求参数校验？

数据的校验的重要性就不用说了，即使在前端对数据进行校验的情况下，我们还是要对传入后端的数据再进行一遍校验，避免用户绕过浏览器直接通过一些 HTTP 工具直接向后端请求一些违法数据。

Spring Boot 程序做请求参数校验的话只需要`spring-boot-starter-web` 依赖就够了，它的子依赖包含了我们所需要的东西。

#### 16.1. 校验注解

**Hibernate Validator 提供的校验注解**：

- `@NotBlank(message =)` 验证字符串非 null，且长度必须大于 0
- `@Email` 被注释的元素必须是电子邮箱地址
- `@Length(min=,max=)` 被注释的字符串的大小必须在指定的范围内
- `@NotEmpty` 被注释的字符串的必须非空
- `@Range(min=,max=,message=)` 被注释的元素必须在合适的范围内

#### 16.2. 验证请求体(RequestBody)

我们在需要验证的参数上加上了`@Valid` 注解，如果验证失败，它将抛出`MethodArgumentNotValidException`。默认情况下，Spring 会将此异常转换为 HTTP Status 400（错误请求）。

```java
@RestController
@RequestMapping("/api")
public class PersonController {

    @PostMapping("/person")
    public ResponseEntity<Person> getPerson(@RequestBody @Valid Person person) {
        return ResponseEntity.ok().body(person);
    }
}
```

#### 16.3. 验证请求参数(Path Variables 和 Request Parameters)

一定一定不要忘记在类上加上 Validated 注解了，这个参数可以告诉 Spring 去校验方法参数。

```java
@RestController
@RequestMapping("/api")
@Validated
public class PersonController {

    @GetMapping("/person/{id}")
    public ResponseEntity<Integer> getPersonByID(@Valid @PathVariable("id") 
    @Max(value = 5,message = "超过 id 的范围了") Integer id) {
        return ResponseEntity.ok().body(id);
    }

    @PutMapping("/person")
    public ResponseEntity<String> getPersonByName(@Valid @RequestParam("name") 
    @Size(max = 6,message = "超过 name 的范围了") String name) {
        return ResponseEntity.ok().body(name);
    }
}
```
## 如何使用 Spring Boot 实现全局异常处理？

可以使用 `@ControllerAdvice` 和 `@ExceptionHandler` 处理全局异常。


## 说一下 Spring Boot 自动装配原理？
springBoot定义了一套接口规范，在springboot启动时会扫描外部引入jar包得`META_INF/spring.factories`文件，将文件中配置的类型信息加载到spring容器中，并且执行类中定义的各种操作。

核心注解是`@SpringBootApplication`，里面有三个核心的注解`SpringBootConfiguration`, `ComponetScan`, `EnableAutoConfiguration`。

- `springbootConfiguration` :点进去其实就是`Configuration`。允许在上下文注册额外的bean或者导入其他配置类。
- `ComponentScan`: 扫描被`Component`注解的类，注解会默认扫描启动类所在包下的所有类。当然也可以自定义某些不扫描的bean。
- `EnableAutoConfiguration`: 是实现自动装配的重要注解。也是核心。

![s](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bcc73490afbe4c6ba62acde6a94ffdfd~tplv-k3u1fbpfcp-watermark.image)

### `EnableAutoConfiguration`：实现自动装配的核心

这个注解下面有两个核心注解：
- `AutoConfigurationPackage`，作用是将main包下的全部组件注册到容器中。
- `Import({AutoConfigurationImportSelector.class})`: 加载自动装配类 xxxAutoConfiguration

接下来我们需要看下这个类到底是什么。

#### AutoConfigurationImportSelector.class 加载自动装配类

这个类继承了一些类，一些类又实现了一些接口，但是这个并不重要。重要的是这个类实现了一个叫`selectImports`的方法，**这个方法的作用是获得所有符合条件的类的全限定名，这些类会被加载到IoC容器中**。

```java
	@Override
	public String[] selectImports(AnnotationMetadata annotationMetadata) {
        // 判断开启了自动装配开关
		if (!isEnabled(annotationMetadata)) {
			return NO_IMPORTS;
		}
        // 获取所有需要装配的bean
		AutoConfigurationEntry autoConfigurationEntry = getAutoConfigurationEntry(annotationMetadata);
		return StringUtils.toStringArray(autoConfigurationEntry.getConfigurations());
	}
```

这里我们重点关注下 `getAutoConfigurationEntry()`这个方法，这个方法是负载加载自动配类的。

方法的调用如下：
![ss](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3c1200712655443ca4b38500d615bb70~tplv-k3u1fbpfcp-watermark.image)


再来仔细回味下这个`getAutoConfigurationEntry()`的源码：
```java 
	protected AutoConfigurationEntry getAutoConfigurationEntry(AnnotationMetadata annotationMetadata) {
        // 判断是否打开了自动装配开关
		if (!isEnabled(annotationMetadata)) {
			return EMPTY_ENTRY;
		}
        // 获取`EnableAutoConfiguration`注解中排除的类型
		AnnotationAttributes attributes = getAttributes(annotationMetadata);
        /* 获取所有需要自动装配的类，`META-INF/spring.factories`
        全部spring boot start下的/spring.factories都会被读入。
        */
		List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes);
		// 进行筛选，满足条件的类才被加载。
        configurations = removeDuplicates(configurations);
		Set<String> exclusions = getExclusions(annotationMetadata, attributes);
		checkExcludedClasses(configurations, exclusions);
		configurations.removeAll(exclusions);
        // 下面这个函数是筛选函数。
		configurations = getConfigurationClassFilter().filter(configurations);
		fireAutoConfigurationImportEvents(configurations, exclusions);
		return new AutoConfigurationEntry(configurations, exclusions);
	}
```

在过滤加载的时候，都是采用的条件注解进行的。`@ConditionOnBean`当容器里有指定的bean，`@ConditionOnClass`当类路径下有指定类的条件下。

在getCandidateConfigurations这个方法内部有`List<String> configurations = SpringFactoriesLoader.loadFactoryNames(getSpringFactoriesLoaderFactoryClass(),getBeanClassLoader());`这个方法。其实是加载了关键的依赖信息。从META-INF文件夹下面去逐级扫描遍历遍历，加载所以的url资源。
* springFactoriesLoader
java的双亲委派模型无法加载一些实现了java核心库提供的接口，但是第三方实现的库。这就需要打破双亲委派模型。线程上下文类加载器(ContextClassLoader)可以解决这个问题。实际上，它仅仅是Thread类的一个变量而已。

### 如何写一个自己的starter
首先新建一个springboot的项目工程，一般叫`xxxx-spring-boot-starter`。然后构造一个类，叫做`xxxxAutoConfiguration`。这类内使用核心注解`Configuration`,`bean`，`@ConditionalOnClass(ThreadPoolExecutor.class)`,给充分加载条件：
```java
package org.example;

import org.springframework.boot.autoconfigure.condition.ConditionalOnClass;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.util.concurrent.LinkedBlockingDeque;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;

@Configuration
public class ThreadPoolAutoConfiguration {

    @Bean
    @ConditionalOnClass(ThreadPoolExecutor.class)
    public ThreadPoolExecutor MyThread(){
        return new ThreadPoolExecutor(10,10,0, TimeUnit.SECONDS,new LinkedBlockingDeque<>());
    }
}

```


然后在resource文件夹中新建`META-INF`的`spring-factories`文件，指定`org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
  org.example.ThreadPoolAutoConfiguration` 告诉去自动装配这个类。

然后只需要在其他项目中导入这个依赖就可以。
```xml
<dependency>
    <groupId>org.example</groupId>
    <artifactId>threadpool-spring-boot-starter</artifactId>
    <version>0.0.1-SNAPSHOT</version>
</dependency>
```

## spring 中IoC容器源码

首先最简单启动Spring的方法。
```java
public static void main(String[] args) {
    ApplicationContext context = new ClassPathXmlApplicationContext("classpath:applicationfile.xml");
}
```

这里的Application其实就是一个IoC的容器。ApplicationContext 启动过程中，会负责**创建实例 Bean，往各个 Bean 中注入依赖等。**

* BeanFactory 生产bean的工厂
实际上ApplicationContext也是一个高级的beanFactory。有一些更高级的方法，获取多个bean等。
BeanFactory只是Spring IoC容器的一种实现，如果没有特殊指定，它采用采用延迟初始化策略；而在实际场景下，我们更多的使用另外一种类型的容器：ApplicationContext，除了具有BeanFactory的所有能力之外，还提供对事件监听机制以及国际化的支持等。它管理的bean，在容器启动时全部完成初始化和依赖注入操作。

在容器的启动阶段，`BeanFactoryPostProcessor`允许我们在容器实例化相应对象之前，对注册到容器的BeanDefinition所保存的信息做一些额外的操作，比如修改bean定义的某些属性或者增加其他信息等。

## bean的生命周期

首先是实例化beanFactory的。

![](https://image.51cto.com/files/uploadimg/20110419/0930070.png)

1.由BeanFactory读取Bean定义文件，并生成各个实例。2.然后依赖注入。3. 如果实现了`BeanNameAware`接口，工厂调用`setBeanName()`方法，传入ID；4. 如果实现了`BeanFactoryAware()`接口，调用`setBeanFactory()`；5. 如果这个Bean关联了BeanPostProcessor接口,调用`ProcessBeforeInitialization`.6. 执行initializingBean接口的`afterPropertiesSet()` 7.、如果Bean在Spring配置文件中配置了`init-method`属性会自动调用其配置的初始化方法。8. BeanPostProcessors的`ProcessAfterInitialization()`

其中这个步骤四也可以是 *如果这个Bean已经实现了ApplicationContextAware接口，会调用setApplicationContext(ApplicationContext)方法，传入Spring上下文*

至此，已经可以使用了。在容器关闭以后，9.如果Bean实现了DisposableBean这个接口，会调用`destroy()`方法；10.如果这个Bean的Spring配置中配置了`destroy-method`属性，会自动调用其配置的销毁方法。



笼统说是四个。
1. 实例化
2. 属性赋值 
3. 初始化 
4. 销毁 Destruction


# 拦截器与过滤器
过滤器和拦截器的区别：
1. 拦截器是基于java的反射机制的，而过滤器是基于函数回调。
2. 拦截器不依赖与servlet容器，过滤器依赖与servlet容器。
3. 拦截器只能对action请求起作用，而过滤器则可以对几乎所有的请求起作用。
4. 拦截器可以访问action上下文、值栈里的对象，而过滤器不能访问。
5. 在action的生命周期中，拦截器可以多次被调用，而过滤器只能在容器初始化时被调用一次。
6. 拦截器可以获取IOC容器中的各个bean，而过滤器就不行，这点很重要，在拦截器里注入一个service，可以调用业务逻辑。
7. 过滤器和拦截器触发时机不一样:　过滤器是在请求进入容器后，但请求进入servlet之前进行预处理的。请求结束返回也是，是在servlet处理完后，返回给前端之前。
![在这里插入图片描述](https://img-blog.csdnimg.cn/202103112359562.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pjejU1NjY3MTk=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210311235702720.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pjejU1NjY3MTk=,size_16,color_FFFFFF,t_70)

 ![在这里插入图片描述](https://img-blog.csdnimg.cn/202103112357313.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pjejU1NjY3MTk=,size_16,color_FFFFFF,t_70)


 ## 什么是serverlet

 事实上，servlet就是一个Java接口。servlet接口定义的是一套处理网络请求的规范，所有实现servlet的类，都需要实现它那五个方法，其中最主要的是两个生命周期方法 init()和destroy()，还有一个处理请求的service()，也就是说，所有实现servlet接口的类，或者说，所有想要处理网络请求的类，都需要回答这三个问题：
 - 你初始化时要做什么
 - 你销毁时要做什么
 - 你接受到请求时要做什么

那请求怎么来到servlet呢？答案是**servlet容器**，比如我们最常用的tomcat，同样，你可以随便谷歌一个servlet的hello world教程，里面肯定会让你把servlet部署到一个容器中，不然你的servlet压根不会起作用。tomcat才是与客户端直接打交道的家伙，**他监听了端口，请求过来后，根据url等信息，确定要将请求交给哪个servlet去处理，然后调用那个servlet的service方法，service方法返回一个response对象，tomcat再把这个response返回给客户端**。

* 如何实现一个servelet呢？

首先继承HttpServelet并且重写doGet()/doPost();父类已经逻辑都写好了，子类只需要简单的重写就可以盘活整个逻辑。
