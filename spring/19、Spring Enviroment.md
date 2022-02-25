[TOC]

PropertyResolver 属性处理器

ConfigurationPropertyResource  可配置接口



Spring Enviroment 有哪些具体的应用场景，怎么用

职责：

- 管理 Spring 配置属性源 

- 管理 Profiles



底层逻辑是什么

@Value 不支持动态更新能力

@Value 的实现方式：

- AutowiredAnnotationBeanPostProcessor
- QualifierAnnotationAutowireCandidateResolver

Enviroment 的外部配置源的类型转换的实现方式：ConversionService



### Spring 的配置属性源

源有很多，源之间也有层次性之分，获取属性值时会取优先级最高的



#### 内置的配置属性源

命令行配置、property配置、环境变量配置等等



#### 基于注解扩展 Spring 配置属性源 

使用：

 @org.springframework.context.annotation.PropertySource  指定要注入spring的配置属性源（properties文件）

实现原理： 

- 入口 - org.springframework.context.annotation.ConfigurationClassParser#doProcessConfigurationClass 
  - org.springframework.context.annotation.ConfigurationClassParser#processPropertySource 

- 4.3 新增语义 
  -  配置属性字符编码 - encoding 
  - org.springframework.core.io.support.PropertySourceFactory 

- 适配对象 - org.springframework.core.env.CompositePropertySource

扩展：

如果想要支持properties以外的文件，如yaml文件，需要自定义实现PropertySourceFactory



#### 基于 API 扩展 Spring 配置属性源