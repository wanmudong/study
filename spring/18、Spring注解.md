[TOC]



### Spring 注解场景

#### Spring 模式注解

![image-20220213162334317](/Users/chenjiehao/Library/Mobile Documents/com~apple~CloudDocs/study/image/image-20220213162334317.png)

#### Spring 装配注解

![image-20220213162345002](/Users/chenjiehao/Library/Mobile Documents/com~apple~CloudDocs/study/image/image-20220213162345002.png)

#### Spring 依赖注入注解

![image-20220213162354243](/Users/chenjiehao/Library/Mobile Documents/com~apple~CloudDocs/study/image/image-20220213162354243.png)



### Spring 注解编程模型

https://github.com/spring-projects/spring-framework/wiki/Spring-Annotation-Programming-Model

编程模型 

- 元注解（Meta-Annotations） 

- Spring 模式注解（Stereotype Annotations） 

- Spring 组合注解（Composed Annotations） 

- Spring 注解属性别名和覆盖（Attribute Aliases and Overrides）



#### Spring 元注解

用于解释注解的一类注解

• java.lang.annotation.Documented 

• java.lang.annotation.Inherited 

• java.lang.annotation.Repeatable



#### Spring 模式注解

通过注解来描述一个组件的身份。

- 理解@Component 的派生性

  - 元标注 @Component 的注解在 XML 元素 <context:component-scan> 或注解 @ComponentScan 扫描中“派 生”了 @Component 的特性，并且从 Spring Framework 4.0 开始支持多层次“派⽣性”。

- 例如：

  - • @Repository 

    • @Service 

    • @Controller 

    • @Configuration 

    • @SpringBootConfiguration（Spring Boot）

- @Component 的派生性的原理

  - 核心组件 （如何通过@ComponentSacn 注解扫描到标注了@Component 的bean）
    - org.springframework.context.annotation.ClassPathBeanDefinitionScanner 
    - org.springframework.context.annotation.ClassPathScanningCandidateComponentProvider 

  - 资源处理 - org.springframework.core.io.support.ResourcePatternResolver 

  - 资源-类元信息 
    - org.springframework.core.type.classreading.MetadataReaderFactory 

  - 类元信息 - org.springframework.core.type.ClassMetadata 
    - ASM 实现 - org.springframework.core.type.classreading.ClassMetadataReadingVisitor 
    - 反射实现 - org.springframework.core.type.StandardAnnotationMetadata 

  - 注解元信息 - org.springframework.core.type.AnnotationMetadata 

    - ASM 实现 - org.springframework.core.type.classreading.AnnotationMetadataReadingVisitor 

    - 反射实现 - org.springframework.core.type.StandardAnnotationMetadata



#### Spring 组合注解

Spring 组合注解（Composed Annotations）中的元注允许是 Spring 模式注解（Stereotype Annotation）与其 他 Spring 功能性注解的任意组合。即组合模式，将不同的功能的注解进行组合，成为一个多功能的注解。

可以参考：@SpringBootApplication



### Spring 注解属性别名（Attribute Aliases）

@AliasFor

- 单个注解内属性的相互别名
- 指定注解内的属性为另一个注解的属性的别名



### Spring 注解属性覆盖（Attribute Overrides）

注解与元注解之间存在同名属性时，会覆盖元注解的属性。

隐性覆盖：注解与元注解之间存在同名属性

显性覆盖：通过@AliasFor指定要覆盖的注解的属性名



### Spring @Enable 模块驱动

- @Enable 模块驱动 

  - @Enable 模块驱动是以 @Enable 为前缀的注解驱动编程模型。所谓“模块”是指具备相同领域的功能组件集合 ，组合所形成⼀个独⽴的单元。⽐如 Web MVC 模块、AspectJ代理模块、Caching（缓存）模块、JMX（Java 管 理扩展）模块、Async（异步处理）模块等。

  - 举例说明 

    • @EnableWebMvc 

    • @EnableTransactionManagement 

    • @EnableCaching 

    • @EnableMBeanExport 

    • @EnableAsync

- @Enable 模块驱动编程模式 

  - 驱动注解：@EnableXXX 

  - 导入注解：@Import 具体实现 

  - 具体实现

    - 基于 Configuration Class 

    - 基于 ImportSelector 接口实现：

    - 基于 ImportBeanDefinitionRegistrar 接口实现 ：要注入的具体的bean

### Spring 条件注解

- 基于配置条件注解：仅判断环境 

  - 场景：指定bean 激活的Profile，在spring容器启动给的时候指定Environment 中的 Profiles ，此时该bean 才会生效
  -  @org.springframework.context.annotation.Profile 
  - 关联对象 org.springframework.core.env.Environment 中的 Profiles 

  - 实现变化：从 Spring 4.0 开始，@Profile 基于 @Conditional 实现 

- 基于编程条件注解：手动编写条件代码

  - @org.springframework.context.annotation.Conditional 

  - 关联对象 - org.springframework.context.annotation.Condition 具体实现



#### @Conditional 实现原理 

- 上下文对象 - org.springframework.context.annotation.ConditionContext 

- 条件判断 
  - org.springframework.context.annotation.ConditionEvaluator   当所有Condition均满足的情况下，才不跳过bean的注入

- 配置阶段 
  - org.springframework.context.annotation.ConfigurationCondition.ConfigurationPhase 

- 判断入口

  - org.springframework.context.annotation.ConfigurationClassPostProcessor 

  - org.springframework.context.annotation.ConfigurationClassParser