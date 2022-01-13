对于注入（一段关联）本身： 有哪些注入的方法，每一种怎么用，原理是什么，为什么要用这个方法，彼此的差异性是什么。

对于被注入的数据：有哪数据些可以被注入，对象、基础类型、集合类型、限定注入（指定注入对象）

- 自动
- 手动
  - 构造器
  - setter方法注入
  - 字段注入
  - 方法注入



自动绑定

1、spring 为什么有自动绑定？autowiring

无需知道字段对应的依赖，spring自动绑定，简化代码

2、自动绑定模式有哪些？autowiring mode

- no
- byName
- byType（出现冲突，需要指定primary bean）
- constructure

3、自动绑定的不足

- 不够精确
- 存在bean不存在，而无法注入的情况



Setter方法注入

4、Setter方法注入的方式

- 手动注入：手动指定setter方法对应的ref
- 自动注入：指定自动注入方式

5、 构造器注入

6、 字段注入

7、 方法注入

8、 结构回调注入

- Aware系列接口回调

9、注入方式之间的差异



10、基础类型注入，除了常说的bean 注入，还有哪些数据可以注入



11、集合类型注入



12、限定注入 @Qualifier

两种用法

- 限定注入的依赖
- 对bean 进行分组 通过@Qualifier或者扩展该注解使用，思考策略模式是否可以使用这个注解来区分不同策略的实现方式呢？



13、 延迟依赖注入

- 对非延迟bean进行延迟注入操作
  - ObjectProvider（可以获取单一类型以及集合类型）
  - ObjectFactory（可以获取单一类型以及集合类型）





14、依赖处理过程：在使用@Autowire时发送了什么事情？

DefaultListableBeanFactory#resolveDependency 看这个方法，可以打断点测试。

在该方法中会处理依赖注入，内部会区分懒加载、集合类型注入、bean注入等。

本质上是将@Autowire转换为DependencyDescriptor（依赖描述符），即先确认我需要什么样的依赖，然后找到需要的依赖，进行注入。

而AutowireCandidateResolver则是待注入的候选对象。



15、@Autowire 怎么将bean进行注入的？

AutowiredAnnotationBeanPostProcessor

- postProcessMergedBeanDefinition:先找到bean的元数据，进行构建元数据的操作。（merged？合并双亲bean）
- postProcessProperties:依据构建的元数据，来进行依赖的注入，其中注入过程中就有（14）提到的依赖处理过程



16、@Inject 注解用法与@Autowire一致。

过程也在AutowiredAnnotationBeanPostProcessor



17、Java 通用注解注入原理

CommonAnnotationBeanPostProcessor 与AutowiredAnnotationBeanPostProcessor的实现基本一致。相较于后者多了bean 的生命周期的处理（基于PostConstruct以及PreDestroy），实现生命周期的回调。

- javax.xml.ws.WebServiceRef 
-  javax.ejb.EJB 
- javax.annotation.Resource

 生命周期注解

- javax.annotation.PostConstruct 
- javax.annotation.PreDestroy





18、自定义依赖注入注解

实现方式

- 在自定义的注解上标注@Autowire
- 使用AutowiredAnnotationBeanPostProcessor



注意：通过@Bean 声明statis bean时，bean会提早初始化。





小结：

- spring 依赖注入的方式：四种

