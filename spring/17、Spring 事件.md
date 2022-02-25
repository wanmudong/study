[TOC]



### Java 事件、监听者模型

- 设计模式 - 观察者模式  java自带的观察者模式的实现

  - 可观者对象（消息发送者） - java.util.Observable 

  - 观察者 - java.util.Observer 

-  标准化接口 

  - 事件对象，无实际意义，只是标记 - java.util.EventObject  

  - 事件监听器，无实际意义，只是标记 - java.util.EventListener



#### java中事件、监听者使用例子

![image-20220210214735568](/Users/chenjiehao/Library/Mobile Documents/com~apple~CloudDocs/study/image/image-20220210214735568.png)

![image-20220210215637831](/Users/chenjiehao/Library/Mobile Documents/com~apple~CloudDocs/study/image/image-20220210215637831.png)

### Spring 标准事件监听器模型

#### Spring 标准事件 ApplicationEvent

Java 标准事件 java.util.EventObject

Spring 标准事件 ApplicationEvent

#### Spring 事件监听器

##### 基于接口的Spring 事件监听器 EventListener

- Java 标准事件监听器 java.util.EventListener 扩展 

  - 扩展接口 - org.springframework.context.ApplicationListener 

  - 设计特点：单一类型事件处理 

  - 处理方法：onApplicationEvent(ApplicationEvent) 

  - 事件类型：org.springframework.context.ApplicationEvent

##### 基于注解的 Spring 事件监听器

Spring 注解 - @org.springframework.context.event.EventListener

注意：异步执行时，需要在监听器的类上加上@EnableAsync 激活异步

![image-20220210220201505](/Users/chenjiehao/Library/Mobile Documents/com~apple~CloudDocs/study/image/image-20220210220201505.png)

### 注册 Spring ApplicationListener

• 方法一：ApplicationListener 作为 Spring Bean 注册 

• 方法二：通过 ConfigurableApplicationContext API 注册



### Spring 事件发布器

- 方法一：通过 ApplicationEventPublisher 发布 Spring 事件 

  - 获取 ApplicationEventPublisher 

  - 依赖注入 

- 方法二：通过 ApplicationEventMulticaster 发布 Spring 事件 

  - 获取 ApplicationEventMulticaster 

  - 依赖注入 

  - 依赖查找



### Spring 层次性上下文事件传播 

- 发生说明 
  - 当 Spring 应用出现多层次 Spring 应用上下文（ApplicationContext）时，如 Spring WebMVC、Spring Boot 或 Spring Cloud 场景下，由子 ApplicationContext 发起 Spring 事件可能会传递到其 Parent ApplicationContext（直到 Root）的过程 

- 如何避免 
  - 定位 Spring 事件源（ApplicationContext）进行过滤处理



### Spring 内建事件

- ApplicationContextEvent 派生事件 

  - ContextRefreshedEvent ：Spring 应用上下文就绪事件 

  - ContextStartedEvent ：Spring 应用上下文启动事件 

  -  ContextStoppedEvent ：Spring 应用上下文停止事件 

  - ContextClosedEvent ：Spring 应用上下文关闭事件



### 使用Spring 的事件机制

#### 自定义Spring 事件

- 扩展 org.springframework.context.ApplicationEvent 

- 实现 org.springframework.context.ApplicationListener 
-  注册 org.springframework.context.ApplicationListener

#### 依赖注入 ApplicationEventPublisher

- 通过 ApplicationEventPublisherAware 回调接口 

- 通过 @Autowired ApplicationEventPublisher

当然，直接注入ApplicationContext也可以



### Spring 如何实现事件的广播



#### 同步和异步 Spring 事件广播

##### 方式一：基于ApplicationEventMulticaster 

- 基于实现类 - org.springframework.context.event.SimpleApplicationEventMulticaster 

  - 模式切换：setTaskExecutor(java.util.concurrent.Executor) 方法 

  - 默认模式：同步 

  - 异步模式：如 java.util.concurrent.ThreadPoolExecutor   这会使所有的事件均为异步

  - 设计缺陷：非基于接口契约编程

注意，需要通过监听springclose事件，关闭线程池

方式二：基于@Async 

- 基于注解 - @org.springframework.context.event.EventListener 

  - 模式切换

  - 默认模式：同步 

  - 异步模式：标注 @org.springframework.scheduling.annotation.Async 

  - 实现限制：无法直接实现同步/异步动态切换



### Spring 事件异常处理

org.springframework.util.ErrorHandler

- 使用场景

  - Spring 事件（Events） 
    - SimpleApplicationEventMulticaster Spring 4.1 开始支持 

  - Spring 本地调度（Scheduling） 

    - org.springframework.scheduling.concurrent.ConcurrentTaskScheduler 

    - org.springframework.scheduling.concurrent.ThreadPoolTaskScheduler

### Spring 事件/监听器实现原理

核心类 - org.springframework.context.event.SimpleApplicationEventMulticaster 

- 设计模式：观察者模式扩展 

  - 被观察者 - org.springframework.context.ApplicationListener 

    - API 添加 

    - 依赖查找 

  - 通知对象 - org.springframework.context.ApplicationEvent 

- 执行模式：同步/异步 

- 异常处理：org.springframework.util.ErrorHandler 

- 泛型处理：org.springframework.core.ResolvableType

注意：SimpleApplicationEventMulticaster  会获取事件对应的Listener，依次执行。如果事件是Listener关心的事件类型的子类时，也会执行该Listener。即泛型与层次性处理。





### Spring 事件核心接口/组件？

- Spring 事件 - org.springframework.context.ApplicationEvent 

- Spring 事件监听器 - org.springframework.context.ApplicationListener 

- Spring 事件发布器 - org.springframework.context.ApplicationEventPublisher 

- Spring 事件广播器 - org.springframework.context.event.ApplicationEventMulticaster



### Spring 同步和异步事件处理的使用场景？

- Spring 同步事件 - 绝大多数 Spring 使用场景，如 ContextRefreshedEvent 

- Spring 异步事件 - 主要 @EventListener 与 @Asyc 配合，实现异步处理，不阻塞主线程，比如长 时间的数据计算任务等。不要轻易调整 SimpleApplicationEventMulticaster 中关联的 taskExecutor 对象，除非使用者非常了解 Spring 事件机制，否则容易出现异常行为。