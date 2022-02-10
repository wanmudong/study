

大概目录：
1. 什么是mvcc，简单一句话介绍，然后引入本次文章主题：深入mvcc源码进行解析
2. 相关知识补充：mysql行数据存储，trx_id，roll-ptr，undoLog，readview等（需要比较详细的介绍），可以从mysqldoc上找一些图，或者描述用自己的话说出来
	https://dev.mysql.com/doc/refman/5.7/en/innodb-multi-versioning.html
3. 构建一个场景（可以复杂一点，多开器几个事务），在没有mvcc的情况下，mysql如何实现事务隔离，和下面的一章可以连起来，如果只使用锁，性能问题	
4. 引出mvcc的概念 和工作环境，在mysql中如何开启，如何和其他参数共事
5. 从源码开始+上面的场景案例，进入mvcc，开启readview源码解析，debug进入，看每个参数的的值是什么，源码解析分为两三个小节
6. 小节一下
7. 说说mvcc还有哪些问题不能解决，比如幻读场景等，mvcc会造成什么影响，比如表膨胀，或者参考下网上的文章分析
8. 致谢+总结

 