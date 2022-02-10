[TOC]



### Spring 配置元信息



这一章的重点：Spring 配置元信息是什么？如何产生？如何使用？



- 配置元信息 
  - Spring Bean 配置元信息 - BeanDefinition 
  - Spring Bean 属性元信息 - PropertyValues 
  -  Spring 容器配置元信息 
  -  Spring 外部化配置元信息 - PropertySource 
  - Spring Profile 元信息 - @Profile



#### Spring Bean 配置元信息

- GenericBeanDefinition 通用 bean definition
- RootBeanDefinition 无 Parent 的 BeanDefinition 或者合并后 BeanDefinition
- AnnotatedBeanDefinition：注解标注的 BeanDefinition

#### Spring Bean 属性元信息

Bean 属性元信息 - PropertyValues 

- 可修改实现 - MutablePropertyValues 

- 元素成员 - PropertyValue 

Bean 属性上下文存储 

- AttributeAccessor 其作用在于bean的生命周期的上下文中传递属性。

Bean 元信息元素 

- BeanMetadataElement 其作用在于知道bean的元数据是来源于哪里的。

#### Spring 容器配置元信息

> 通过BeanDefinitionParserDelegate来解析

 XML 配置元信息 - beans 元素相关 

![image-20220124225922491](/Users/chenjiehao/Library/Mobile Documents/com~apple~CloudDocs/study/image/image-20220124225922491.png)



 Spring Bean 配置元信息 

![image-20220124225937713](/Users/chenjiehao/Library/Mobile Documents/com~apple~CloudDocs/study/image/image-20220124225937713.png)



### Spring Bean 配置元信息加载



#### 基于XML资源装载Spring Bean 配置元信息

> 底层基于XmlBeanDefinitionReader实现，满足W3C协议，底层通过DOM解析文件
>
> 注意，根节点必然是bean标签，bean标签内的bean 的定义被解析成BeanDefinitionHolder，然后进行注册



- Spring XML 资源 BeanDefinition 解析与注册 
  - 核心 API - XmlBeanDefinitionReader 
    - 资源 - Resource 
    - 底层 - BeanDefinitionDocumentReader 
      - XML 解析 - Java DOM Level 3 API 
      - BeanDefinition 解析 - BeanDefinitionParserDelegate 
      - BeanDefinition 注册 - BeanDefinitionRegistry





#### 基于Properties资源装载Spring Bean 配置元信息

> 底层基于PropertiesBeanDefinitionReader实现



- Spring Properties 资源 BeanDefinition 解析与注册 
  - 核心 API - PropertiesBeanDefinitionReader 
    -  资源
      - 字节流 - Resource 
      - 字符流 - EncodedResouce 
    - 底层
      - 存储 - java.util.Properties 
      - BeanDefinition 解析 - API 内部实现 
      - BeanDefinition 注册 - BeanDefinitionRegistry



#### 基于Java 注解装载Spring Bean 配置元信息

> 底层基于AnnotationBeanDefinitionReader实现



- Spring Java 注册 BeanDefinition 解析与注册 
  -  核心 API - AnnotatedBeanDefinitionReader 
    - 资源
      - 类对象 - java.lang.Class 
    - 底层
      - 条件评估，判断bean是否需要注册 - ConditionEvaluator 
      - Bean 范围解析 - ScopeMetadataResolver 
      - BeanDefinition 解析 - 内部 API 实现 
      - BeanDefinition 处理 - AnnotationConfigUtils.processCommonDefinitionAnnotations 
      - BeanDefinition 注册 - BeanDefinitionRegistry



### Spring Ioc容器配置元信息加载

#### 基于XML资源装载Spring IoC 容器 配置元信息

注意spring.handlers 以及 spring.schemas 两个文件



#### 基于Java注解装载Spring IoC 容器 配置元信息

- Spring IoC 容器装配注解
- Spring IoC 配置属性注解



### Spring 外部化配置元信息加载

#### 基于 Properties 资源装载外部化配置

注解驱动 

- @org.springframework.context.annotation.PropertySource 

- @org.springframework.context.annotation.PropertySources 

 API 编程

- org.springframework.core.env.PropertySource 

- org.springframework.core.env.PropertySources



#### 基于 YAML 资源装载外部化配置

API 编程

- org.springframework.beans.factory.config.YamlProcessor 

  - org.springframework.beans.factory.config.YamlMapFactoryBean 

  - org.springframework.beans.factory.config.YamlPropertiesFactoryBean

注意： user.name 是 Java Properties 默认存在，当前用户的名称。