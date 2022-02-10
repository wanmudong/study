

### Spring bean 有哪些作用域

> 这里的spring bean 其实是指 beandefinition。
>
> beandefinition中有字段：isSingleton、isPrototype。

![image-20220116174231792](/Users/chenjiehao/Library/Mobile Documents/com~apple~CloudDocs/study/image/image-20220116174231792-2326153.png)

注：鉴于目前是前后端分离，所以理解singleton与prototype即可。

#### singleton

- 在依赖查找或者依赖注入中，会通过判断bean.isSIngleton来判断bean 是否是单例模式，从而做到注入的是同一个bean。
- 注意是在一个beanFactory中是唯一的。

#### prototype

- 通过@Scope注解来标注当前bean作用域，默认为单例模式。
- 在依赖查找或者依赖注入中，均为新生成的对象

- Spring 容器没有办法管理 prototype Bean 的完整生命周期，也没有办法记录实例的存在。销毁回调方法将不会执行，可以利用 BeanPostProcessor 进行清扫工作。这是为什么？因为此时bean与对象的对应关系是一对多，spring此时无法进行管理。
  - 推荐使用org.springframework.beans.factory.DisposableBean来销毁prototype的对象。

#### 自定义bean作用域

<img src="/Users/chenjiehao/Library/Mobile Documents/com~apple~CloudDocs/study/image/image-20220116181633048.png" alt="image-20220116181633048" style="zoom:50%;" />



小结：

