[TOC]



##  1.1 定义数据库和实例

数据库：物理操作系统文件或其他形式文件类型的集合。

实例：MySQL数据库由后台线程和一个共享内存组成。MySQL数据库实例在系统上的表现就是一个进程。



## 1.2 MySQL体系结构



![image-20210428091910881](/Users/chenjiehao/Library/Mobile Documents/com~apple~CloudDocs/study/image/image-20210428091910881.png)



MySQL 组成部分： 连接池组件、管理服务和工具组件、SQL接口组件、查询分析器组件、优化器组件、缓存组件、插件式存储引擎、物理文件。



> 注意：存储引擎基于表而不是基于物理文件。



## 1.3 MySQL存储引擎



InnoDB存储引擎支持事务，其设计目标主要是面向在线事务处理（OLTP）的应用。

InnoDB存储引擎将数据存储在逻辑的表空间，这个表空间像黑盒一样由InnoDB存储引擎自身管理。

InnoDB通过多版本并发控制（MVCC）获取高并发性。同时提供插入缓存、二次写、自适应哈希索引、预读等高性能和高可用的功能。

对于表中的数据，InnoDB存储引擎采用了聚集（clustered）的方式，因此每张表的存储都是按主键的顺序进行存放。

```mysql

mysql> show engines\G;
*************************** 1. row ***************************
      Engine: InnoDB
     Support: DEFAULT
     Comment: Supports transactions, row-level locking, and foreign keys
Transactions: YES
          XA: YES
  Savepoints: YES
*************************** 2. row ***************************
      Engine: MRG_MYISAM
     Support: YES
     Comment: Collection of identical MyISAM tables
Transactions: NO
          XA: NO
  Savepoints: NO
*************************** 3. row ***************************
      Engine: PERFORMANCE_SCHEMA
     Support: YES
     Comment: Performance Schema
Transactions: NO
          XA: NO
  Savepoints: NO
*************************** 4. row ***************************
      Engine: BLACKHOLE
     Support: YES
     Comment: /dev/null storage engine (anything you write to it disappears)
Transactions: NO
          XA: NO
  Savepoints: NO
*************************** 5. row ***************************
      Engine: CSV
     Support: YES
     Comment: CSV storage engine
Transactions: NO
          XA: NO
  Savepoints: NO
*************************** 6. row ***************************
      Engine: MyISAM
     Support: YES
     Comment: MyISAM storage engine
Transactions: NO
          XA: NO
  Savepoints: NO
*************************** 7. row ***************************
      Engine: ARCHIVE
     Support: YES
     Comment: Archive storage engine
Transactions: NO
          XA: NO
  Savepoints: NO
*************************** 8. row ***************************
      Engine: MEMORY
     Support: YES
     Comment: Hash based, stored in memory, useful for temporary tables
Transactions: NO
          XA: NO
  Savepoints: NO
*************************** 9. row ***************************
      Engine: FEDERATED
     Support: NO
     Comment: Federated MySQL storage engine
Transactions: NULL
          XA: NULL
  Savepoints: NULL
9 rows in set (0.00 sec)
```



![image-20210428092906547](/Users/chenjiehao/Library/Mobile Documents/com~apple~CloudDocs/study/image/image-20210428092906547.png)



## 1.5 连接MySQL

进程间通用的通信方式有：管道、命名管道、命名字、TCP/IP套接字、UNIX域套接字。



#### TCP/IP：

```
mysql -h<ip> -u username -p
```

需要注意的是，进行连接时，MySQL会先检查一张权限视图，判断IP是否允许连接。该视图在mysql库中，表名为user。

```sql
USE mysql;
select host,user,password fron user;
```



#### UNIX域套接字：



UNIX域套接字不是一个网络协议，所以只能在MySQL 客户端和数据库实例在一台服务器上的情况下使用。使用方式如下：1、获取套接字文件路径；2、进行连接时指定套接字文件的路径。

```mysql

mysql> show variables like 'socket';
+---------------+-----------------+
| Variable_name | Value           |
+---------------+-----------------+
| socket        | /tmp/mysql.sock |
+---------------+-----------------+
1 row in set (0.01 sec)
```

``` mysql

```



## 1.6 安装MySQL

[安装MySQL](https://www.cnblogs.com/yinzhengjie/p/10125609.html)

[初始密码忘记了怎么办](https://blog.csdn.net/WangQingLei0307/article/details/109148778?utm_medium=distribute.pc_relevant.none-task-blog-baidujs_title-0&spm=1001.2101.3001.4242)





