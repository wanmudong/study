### Spring bean 生命周期

[TOC]



#### Spring Bean 元信息配置阶段

这里的元信息其实也就是BeanDefinitionde 的配置，有三类配置：

- 面向资源：如XML配置（XMLBeanDefinitionReader）、Proterties配置（PropertiesBeanDefinitionReader）
- 面向注解：@Bean
- 面向APi：BeanDefinitionBuilder等

#### Spring Bean 元信息解析阶段

- 面向资源BeanDefinition解析：BeanDefinitionReader
- 面向注解BeanDefinition解析：AnnotatedBeanDefinitionReader.register
  - bean名称生成于BeanNameGenerator。


#### Spring Bean 注册阶段

beandefinition 的注册接口：BeanDefinitionRegistry

- 实现类：DefaultLIstableBeanFactory.registerBeanDefinition

通过map维护bean ，通过list维护beanNames的顺序

代码思考：新建操作相较于覆盖操作关系线程安全，需要锁确保唯一性。

#### Spring BeanDefinition合并阶段

org.springframework.beans.factory.support.AbstractBeanFactory#getMergedBeanDefinition

过程中如果是有parent，会递归查询parent，并且以parent为基础，用child的属性来覆盖parent，得到最终的beandefinition。

结论：beandefinition最终呈现的是一个RootBeanDefinition。不论最开始定义的就是RootBeandefinition还是一个有Parent的ChildBeanDefinition。

#### Spring Bean 实例化前置操作

阶段所处问题：resolveBeforeInstantiation（创建对象阶段）

阶段目标：可以手动对bean进行实例化，如果扩展点返回bean则作为bean的实例化结果进行返回，返回null则进行默认后续流程。

扩展点：InstantiationAwareBeanPostProcessor#postProcessBeforeInstantiation

#### Spring Bean 实例化阶段

- 实例化方式 
  -  传统实例化方式 
  -  实例化策略 - InstantiationStrategy 

- 构造器依赖注入：构造器注入按照类型注入，内部通过resolveDependency来查找依赖的bean。

#### Spring Bean 实例化后阶段

阶段所处位置：doCreateBean中的populateBean（填充对象）

阶段目标：bean已经实例化，此时属性还未填充。在此时可以扩展点可以填充对象，并且选择是否执行默认的属性填充流程。

扩展点：InstantiationAwareBeanPostProcessor#postProcessAfterInstantiation（返回false则跳过默认的属性填充，true则不跳过）

#### Spring Bean 属性赋值前阶段

阶段所处问题：doCreateBean中的populateBean（填充对象）

阶段目标：此时bean正在在进行属性填充(AbstractAutowireCapableBeanFactory#populateBean)，在此阶段可以选择是否将bean 的属性在赋值的时候进行修改

扩展点：

- Bean 属性值元信息 ：PropertyValues 

- Bean 属性赋值前回调 

  - Spring 1.2 - 5.0：InstantiationAwareBeanPostProcessor#postProcessPropertyValues 

  - Spring 5.1：InstantiationAwareBeanPostProcessor#postProcessProperties（默认返回null，表示不修改赋值）

    

注：我们常常回写一些复制属性的代码，就可以参考这里：AbstractAutowireCapableBeanFactory#applyPropertyValues

#### Spring Bean Ware接口回调阶段

阶段所处位置：doCreateBean中的initializeBean

阶段目标：此时bean正在进行初始化（AbstractAutowireCapableBeanFactory#initializeBean()）

阶段实现方式：

1. invokeAwareMethods(BeanFactory)
2. applyBeanPostProcessorsBeforeInitialization(ApplicationContext)



Spring Aware 接口 

- 在BeanFactory所拥有的的回调

  - BeanNameAware 

  - BeanClassLoaderAware 

  - BeanFactoryAware 

- 在ApplicationContext所用于的回调（prepareBeanFactory阶段较之BeanFactory多出的行为）

  - EnvironmentAware 
  - EmbeddedValueResolverAware 
  -  ResourceLoaderAware 
  - ApplicationEventPublisherAware 
  - MessageSourceAware 
  - ApplicationContextAware

  

#### Spring Bean 初始化前阶段

阶段所处位置：doCreateBean中的initializeBean

阶段具体位置：applyBeanPostProcessorsBeforeInitialization

阶段目标：这是bean初始化前的最后一个操作，执行bean初始化的前置操作。

阶段实现方式：可以通过注册BeanPostProcessor的形式，执行BeanPostProcessor#postProcessBeforeInitialization方法

注意：上一阶段中的ApplicationContext所用于的回调，就是基于该操作实现的。

#### Spring Bean 初始化阶段

阶段所处位置：doCreateBean中的initializeBean

阶段具体位置：invokeInitMethods

阶段目标：初始化bean，执行bean初始化时的一些扩展点

扩展点：执行顺序自上而下

- @PostConstruct 标注方法 
  - 注意：依赖于注解驱动，需要CommonAnnotationBeanPostProcessor(CommonAnnotationBeanPostProcessor#CommonAnnotationBeanPostProcessor)。其本身是在org.springframework.beans.factory.annotation.InitDestroyAnnotationBeanPostProcessor#postProcessBeforeInitialization进行回调的。
  - 所以，它其实是在bean初始化前就回调了。所以它真正其实处于applyBeanPostProcessorsBeforeInitialization阶段
- 实现 InitializingBean 接口的 afterPropertiesSet() 方法 
- 自定义初始化方法

#### Spring Bean 初始化后置阶段

阶段所处位置：doCreateBean中的initializeBean

阶段具体位置：applyBeanPostProcessorsAfterInitialization

阶段目标：这是bean初始化后的一个操作，执行bean初始化的后置操作。

扩展点：BeanPostProcessor#postProcessAfterInitialization

#### Spring Bean 初始化完成阶段

阶段所处位置：AbstractApplicationContext#finishBeanFactoryInitialization

阶段目的：此时bean是可以确保是一个完整的，可以进行bean初始化后的行为操作。

SmartInitializingSingleton#afterSingletonsInstantiated

注意：

- 一般用于applicationContext场景。
- 其他场景均需要手动调用beanFactory的方法preInstantiateSingletons。此时会先将bean初始化后，再调用每个bean 的afterSingletonsInstantiated方法。

#### Spring Bean 销毁前阶段

注册销毁前方法：doCreateBean中的registerDisposableBeanIfNecessary

阶段所处位置：DisposableBeanAdapter#destroy以及AbstractApplicationContext#close

阶段目标：在bean销毁前进行以下前置操作。可以用在例如应用优雅下线的场景

扩展点：

- DestructionAwareBeanPostProcessor#postProcessBeforeDestruction
- @PreDestroy



#### Spring Bean 销毁阶段 

阶段所处位置：：DisposableBeanAdapter#destroy以及AbstractApplicationContext#close

扩展点：

- @PreDestroy 标注方法 
  - 注意：依赖于注解驱动，需要CommonAnnotationBeanPostProcessor(CommonAnnotationBeanPostProcessor#CommonAnnotationBeanPostProcessor)，其本质是在销毁前阶段进行的。
- 实现 DisposableBean 接口的 destroy() 方法 
- 自定义销毁方法



#### Spring Bean 垃圾收集





- 关闭 Spring 容器（应用上下文） 
- 执行 GC 
- Spring Bean 覆盖的 finalize() 方法被回调





### 小结：

##### BeanPostProcessor 的使用场景有哪些？

- 答：BeanPostProcessor 提供 Spring Bean 初始化前和初始化后的 生命周期回调，分别对应 postProcessBeforeInitialization 以及 postProcessAfterInitialization 方法，允许对关心的 Bean 进行 扩展，甚至是替换。 

- 加分项：其中，ApplicationContext 相关的 Aware 回调也是基于 BeanPostProcessor 实现，即 ApplicationContextAwareProcessor 

##### BeanFactoryPostProcessor 与 BeanPostProcessor 的区别

- 答：BeanFactoryPostProcessor 是 Spring BeanFactory（实际为 ConfigurableListableBeanFactory） 的后置处理器，用于扩展 BeanFactory，或通过 BeanFactory 进行依赖查找和依赖注入。 
- 加分项：BeanFactoryPostProcessor 必须有 Spring ApplicationContext 执行，BeanFactory 无法与其直接交互。 而 BeanPostProcessor 则直接与BeanFactory 关联，属于 N 对 1 的关系 。

##### BeanFactory 是怎样处理 Bean 生命周期？

BeanFactory 的默认实现为 DefaultListableBeanFactory，其中 Bean生命周期与方法映射如下： 

- BeanDefinition 注册阶段 - registerBeanDefinition 
- BeanDefinition 合并阶段 - getMergedBeanDefinition 
- Bean 实例化前阶段 - resolveBeforeInstantiation 
-  Bean 实例化阶段 - createBeanInstance 
- Bean 实例化后阶段 - populateBean 
- Bean 属性赋值前阶段 - populateBean 
- Bean 属性赋值阶段 - populateBean 
- Bean Aware 接口回调阶段 - initializeBean 
- Bean 初始化前阶段 - initializeBean 
- Bean 初始化阶段 - initializeBean 
- Bean 初始化后阶段 - initializeBean 
- Bean 初始化完成阶段 - preInstantiateSingletons 
- Bean 销毁前阶段 - destroyBean 
- Bean 销毁阶段 - destroyBean