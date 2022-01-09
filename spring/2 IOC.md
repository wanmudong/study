什么是Ioc？

- 为什么是控制反转？控制为什么要反转
- 好莱坞原则：不要打电话给我，我会打电话给你



Java 自省  

spring中依赖查找与依赖注入的来源是相同的吗？通过依赖查找与依赖注入获取到的bean 是相同的吗



依赖来源：

- bean：业务自定义
- 容器内建依赖：依赖注入（谁注入的？容器注入的）
- 容器内建bean：spring初始化生成的bean



1、BeanFactory 和 ApplicationContext 谁才是Ioc容器?

两者皆是Ioc容器，但容器的级别不同。BeanFactory是基础Ioc容器，真正提供了bean的获取能力。ApplicationContext在实现中内部组合了一个BeanFactory，通过组合的BeanFactory来实现BeanFactory的功能（因为ApplicationContext继承了BeanFactory）。



2、ApplicationContext除了Ioc容器外，还是提供了什么功能？

> https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#spring-core

<img src="/Users/chenjiehao/Library/Mobile Documents/com~apple~CloudDocs/study/image/image-20211229225231704.png" alt="image-20211229225231704" style="zoom:67%;" />

3、Ioc容器的生命周期、Ioc容器在停止过程中发生了什么





问题：

- 为什么需要applicationContext.refesh()