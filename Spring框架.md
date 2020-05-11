# 一、Spring Framework

[官方文档](https://spring.io/projects/spring-framework#overview)

[扩展阅读：极客学院Spring教程](https://wiki.jikexueyuan.com/project/spring/)

<img src=".\images\7bUxDv1xrBJq3ZPw.png" alt="image-20200504091713877" style="zoom: 60%;" />

## Core Container

  - `Core`模块是框架的基本组成部分，通过依赖注入(Dependency Injection)来解耦应用组件
  - `Beans`模块提供 BeanFactory，是一个工厂模式的复杂实现
  - `Context`模块以`Core`和 Bean 模块为基础，是访问定义和配置的任何对象的媒介，核心接口是`ApplicationContext `

## 依赖倒置原则、控制反转、依赖注入的关系

<img src="E:\Files\Notes\images\F4mceiTvTKRGt6aj.png" alt="image-20200504093411587" style="zoom:50%;" />

# 二、Spring IoC

[扩展阅读：Spring IoC原理](https://www.zhihu.com/question/23277575/answer/169698662)

## 容器
### IoC容器的启动过程

<img src=".\images\Bdul0Otz2AvSI8Lu.png" alt="image-20200504143401235" style="zoom:67%;" />

### 依赖注入的方式

- 构造函数 Constructor
- 设值函数 Setter
- 注解 Annotation
- 接口 Interface

### 使用IoC容器的好处

- 避免在各处使用new来创建实例，可以做到统一维护
- 创建实例的时候不需要了解其中的细节

## Bean

Bean 是一个由配置元数据创建、组装、并通过 Spring IoC 容器所管理的对象。

### 作用域

| 作用域         | 描述                                                         |
| :------------- | :----------------------------------------------------------- |
| singleton      | 默认作用域，容器里拥有唯一的Bean实例                         |
| prototype      | 针对每个getBean请求，容器都会创建一个Bean实例                |
| request        | 该作用域将 Bean 的定义限制为 HTTP 请求，为每个HTTP请求创建一个Bean实例 |
| session        | 该作用域将 Bean 的定义限制为 HTTP 会话，为每个session创建一个Bean实例 |
| global-session | 该作用域仅对Portlet有效，会为每个全局HTTP Session创建一个Bean实例 |

### 生命周期

1. 创建过程

- 实例化Bean
- 调用ApplicationContextAware的setApplicationContext()方法，注入BeanID、BeanFactory、AppCtx
- 调用`BeanPostProcessor.postProcessBeforeInitialization()`
- 调用`InitializingBean.afterPropertiesSet()`
- 调用`init-method`
- 调用`BeanPostProcessor.postProcessAfterInitialization()`

2. 销毁过程

- 若实现了DisposableBean接口，则调用destory()方法
- 若配置了destory-method属性，则调用配置的销毁方法

### getBean()过程

- 转换beanName
- 尝试从缓存中加载实例
- 实例化Bean
- 检测parentBeanFactory
- 初始化依赖的Bean
- 创建Bean并返回

## BeanFactory

BeanFactory是Spring框架的核心接口

### 功能

- 提供IoC的配置机制
- 包含Bean的各种定义，便于实例化Bean
- 建立Bean之间的依赖关系
- 控制Bean的生命周期

### 体系结构

<img src="E:\Files\Notes\images\x860uRyXpvDOziw3.png" alt="image-20200504150947479" style="zoom: 67%;" />

## ApplicationContext

BeanFactory是Spring框架的基础设置，面向Spring，而ApplicationContext面向使用Spring框架的开发者。

### 功能

ApplicationContext继承了多个接口，具有以下功能

- BeanFactory：能够装配、管理Bean
- ApplicationEvenetPublisher：能够注册监听器，实现监听机制
- ResourcePatternResolver：能够加载资源文件
- MessageResource：管理Message，能够实现国际化功能

### 事件处理

加载 Bean 时，ApplicationContext 会发布某些类型的事件。如果一个 Bean 实现了 ApplicationListener接口，那么每次 ApplicationEvent 被发布到 ApplicationContext 时，那个 Bean 会被通知。

| 事件                    | 描述                                                         |
| :---------------------- | :----------------------------------------------------------- |
| `ContextRefreshedEvent` | `ApplicationContext` 被初始化或刷新时，该事件被发布，也可以在 `ConfigurableApplicationContext` 接口中使用 refresh() 来发生 |
| `ContextStartedEvent`   | 使用 `ConfigurableApplicationContext` 接口中的 start() 会发布该事件，可以在接受到这个事件后重启任何停止的应用程序 |
| `ContextStoppedEvent`   | 使用 `ConfigurableApplicationContext` 接口中的 stop() 会发布这个事件，可以在接受到这个事件后做必要的清理的工作 |
| `ContextClosedEvent`    | 使用 `ConfigurableApplicationContext` 接口中的 close() 关闭时会发布这个事件，`ApplicationContext`不能被刷新或重启 |
| `RequestHandledEvent`   | 这是一个 web-specific 事件，告诉所有 Bean HTTP 请求已经被服务 |

# 三、Spring AOP

[扩展阅读：Spring AOP](https://www.jianshu.com/p/994027425b44)

AOP (Aspect-Oriented Programming)，即面向切面编程。AOP可以将那些与业务逻辑无关、被业务模块共同使用的代码（事务处理、日志管理、权限控制等）封装起来，有利于降低模块间的耦合度，减少系统的冗余代码。

## 核心概念

- 切入点 (Pointcut)：在哪些类和方法上切入 (Where)
- 通知 (Advice)：在方法执行的哪个时间点织入 (When)
- 切面 (Aspect)：在这里实现哪些通用功能 (What)
- 织入 (Weaving)：AOP的实现过程

## 通知 (Advice)


| 通知              | 描述                                                         |
| ----------------- | ------------------------------------------------------------ |
| `@Before`         | 前置通知，在连接点方法前调用                                 |
| `@Around`         | 环绕通知，它将覆盖原有方法，允许使用反射调用原有方法         |
| `@After`          | 后置通知，在连接点方法后调用                                 |
| `@AfterReturning` | 在连接点方法执行并正常返回后调用，要求连接点方法在执行过程中没有发生异常 |
| `@AfterThrowing`  | 异常通知，当连接点方法异常时调用                             |

```java
// Spring AOP 示例
@Component
@Aspect
class LogAspect {
    @Pointcut("execution(public aspect.web..*.*(..))")
    public void log() {}
    
    @Before("log()")
    public void before(){
        // 在执行方法前记录日志
    }
}
```

## 织入方式 (Weaving)

- 编译时织入：需要特殊的Java编译器，如AspectJ
- 类加载时织入：需要特殊的Java类加载器，如AspectJ和AspectWerkz
- 运行时织入：使用动态代理（Spring AOP使用的就是这种方式）

## AOP 实现原理

由AopProxyFactory根据AdvisedSupport对象的配置来决定。默认的策略是如果目标类是接口，则使用JDK动态代理技术(JDK Dynamic Proxy)，否则使用Cglib (Code Generation Library)来生成代理。

- JDK动态代理：通过Java的反射机制实现，核心是InvocationHandler接口和Proxy类
- Cglib：以继承的方式动态生成目标类的代理，通过修改字节码实现

# 四、Spring Transactions

## 隔离级别

| 名称                       | 描述                                             |
| -------------------------- | ------------------------------------------------ |
| ISOLATION_DEFAULT          | 默认选项。使用数据库默认的事务隔离级别           |
| ISOLATION_READ_UNCOMMITTED | 事务中的修改，即使没有提交，对其它事务也是可见的 |
| ISOLATION_READ_COMMITTED   | 保证一个事务只能读取已经提交的事务所做的修改     |
| ISOLATION_REPEATABLE_READ  | 保证在同一个事务中多次读取同一数据的结果是一样的 |
| ISOLATION_SERIALIZABLE     | 保证事务串行执行                                 |

## 传播机制

Spring事务引入了事务传播的概念，提供多个事务的合并和隔离功能。

| 名称                      | 描述                                                         |
| ------------------------- | ------------------------------------------------------------ |
| PROPAGATION_REQUIRED      | 默认选项。支持当前事务，如果当前没有事务，就新建一个事务     |
| PROPAGATION_SUPPORTS      | 支持当前事务，如果当前没有事务，就以非事务方式执行           |
| PROPAGATION_MANDATORY     | 支持当前事务，如果当前没有事务，就抛出异常                   |
| PROPAGATION_REQUIRES_NEW  | 新建事务，如果当前存在事务，把当前事务挂起                   |
| PROPAGATION_NOT_SUPPORTED | 以非事务方式执行操作，如果当前存在事务，就把当前事务挂起     |
| PROPAGATION_NEVER         | 以非事务方式执行，如果当前存在事务，则抛出异常               |
| PROPAGATION_NESTED        | 如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则进行与PROPAGATION_REQUIRED类似的操作 |

## 事务管理

### 编程式

需要在代码中调用beginTransaction()、commit()、rollback()等事务管理相关的方法，比较繁琐

### 声明式

- 基于TransactionProxyFactoryBean，需要为每个进行事务管理的类做相应配置

- 基于AspectJ AOP，在XML文件中进行配置，不需要改动类

- 基于注解@Transactional，配置简单，需要在业务层类中添加注解

# 五、Spring MVC

Spring Web MVC 框架提供了`模型-视图-控制器`的体系结构和可以用来开发灵活、松散耦合的 web 应用程序的组件。

- 模型 (Model) 封装了应用程序数据，通常由 POJO 组成
- 视图 (View) 用于呈现模型数据，生成客户端的浏览器可以解释的HTML文档
- 控制器 (Controller) 用于处理用户请求，构建合适的模型并将其传递到视图层进行渲染

<img src="E:\Files\Notes\images\gqpxm0MjThXrBB81.png" alt="image-20200510210857631" style="zoom: 67%;" />

## 运行流程

1. 接收到HTTP请求后， `DispatcherServlet` 根据 `HandlerMapping`来选择对应的 `Handler`
2. `DispatcherServlet`根据获得的`Handler`，选择一个合适的 `HandlerAdapter` 
3.  `HandlerAdapter`执行完preHandler方法后，会调用真正的处理器开处理请求，并处理相应的业务逻辑
4. `Handler`处理完业务后，向`DispatcherServlet`返回一个`ModelAndView`对象
5.  `ViewResolver` 会根据逻辑 `View` 查找实际的 `View`
6.  `DispaterServlet` 将返回的 `Model` 传给 `View`进行渲染
7. `DispaterServlet`将渲染好的 `View` 返回给请求者（浏览器）

## @Autowired和@Resource的区别

- @Resource默认是**按照名称**来装配注入的，只有当找不到与名称匹配的bean才会按照类型来装配注入
- @Autowired默认是**按照类型**装配注入的，如果想按照名称来转配注入，则需要结合@Qualifier一起使用
- @Resource注解是由JDK提供，而@Autowired是由Spring提供

# 六、MyBatis

