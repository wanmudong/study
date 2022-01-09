

1、 什么是bean definition



2、 bean definition 元信息



3、命名bean

- id
- name：自己起名，容器自动起名（怎么自动起名的？）



4、bean别名

- xml :alias



5、 将bean 注册到Ioc容器中

- BeanDefinition 注册

  - xml：<bean>

  - 注解：@Bean @Component @Import

  - java api：命名、非命名

- 外部单体对象注册：beanFactory.registerSingleton()

6、将bean 实例化

- 构造器：类本身指定静态构造方法
- 工厂：
- FactoryBean： 



7、bean 初始化

- @PostConstruct
- 实现 InitializingBean 接口
- 自定义初始化方法：XML、java注解，java api

8、bean 延迟初始化



9、bean 销毁

- @PreDistroy
- 实现 DisposableBean 接口
- 自定义销毁方法：XML、java注解，java api



10、bean被GC回收



11、 

- bean如何注册？
- 什么是beandefinition？