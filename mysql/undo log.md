

[TOC]



## 事务回滚

为什么会有undo log  为了回滚事务。

### roll_point

这占用7个字节的字段本质上是一个指向当前记录上一个undo log的指针。

### trx_id

InnoDB行记录中有一个trx_id隐藏列，用于表示对该记录进行改动时对应的事务id。

#### 事务id 

##### 分配事务id的时机

- 只读事务：在第一次对临时表进行增删改操作时分配。
- 读写事务：第一次对表（包含临时表）进行增删改操作时分配。

##### 事务id如何生成的

- 数据库本身维护了一个最大事务的事务id属性，需要分配事务id时，将其分给对应的事务，且自身加一。
- 内存中id变为256倍数时刷新到磁盘中。
- 数据库重启加载最大事务id时，将磁盘中的值加上256作为当前最大事务id。



## undo日志格式

undo log中undo no在一个事务中是从0开始递增的。

### Insert操作对应的undo 日志

- trx_undo_insert_rec
- 记录主键每个列占用的存储空间大小和真实值。

### Delete操作对应的undo 日志

InnoDB中删除操作分为两步：

1. 先将记录中的delete_mask标志置为1。
2. 事务提交后，通过purge线程将记录真正的删除，加入垃圾链表（数据页的page_header中有字段指向垃圾链表头结点）。

delete对应的undo  log type ：trx_undo_del_mark_rec

- 记录了old trx_id 与roll_point
- 记录了索引列各列信息的内容。

### update 操作对应的undo 日志

不更新主键的情况：

- 就地更新：当更新前后列占用的存储空间相同时，在原纪录的基础上直接修改对应的值。
- 先删除旧记录，再插入新纪录：更新前后列大小不一致时，先删除旧记录（直接加入垃圾链表），后新增新纪录。

此时对于的undo log type = trx_undo_upd_exist_rec

- 记录更新列的数量。
- 记录被更新前后列的值。

更新主键的情况：

更新主键意味着更新后该条记录在聚簇索引中的位置会发生变化。会分两步进行处理。

1. 将旧记录进行delete mark
2. 插入一条新纪录到聚簇索引中。

对于此时会分别记录两条undo log ，分别是删除以及新增对应的undo log。

>  List node：记录上一节点、记录下一节点
>
> List base node：记录头结点，记录尾结点、记录节点的数量![image_1d79nq0ge1gn7pmcaa7gma1uh62q.png-71kB](https://user-gold-cdn.xitu.io/2019/4/16/16a24a7fcc3c65f1?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

## undo log 页面

用于记录undo log的页面

页面头结点属性：

- trx_undo_type：当前页面存放undo log的类型，分为：insert以及update
- trx_undo_page_start：页面存储undo log 开始的位置。
- trx_undo_page_free：页面可以写入新undo log 的位置。
- trx_undo_page_node：用来与前后undo log 页面进行连接。

## undo 页面链表

单个事务中的undo log 页面链表：通过每个页面的trx_undo_page_node进行链接。

由于我们将undo log页面分为两个大类insert与update，且对于临时表的undo log与普通表跟别记录，所以每个事务中均存在四个undo 页面链表，分别为两个insert undo 链表与两个update undo 链表。在出现undo log 后才出现链表，不是必存的。

多个事务中的undo log 页面链表：事务间的链表相互隔离。

### undo 日志具体写入的过程

#### undo log segment header

InnoDB中每一个undo log 页面链表对应着一个段。所以每一个undo log 页面链表中的第一个节点都会记录着当前链表对应的段的信息。对应的属性如下：

- trx_undo_state：当前链表对应的段的状态
  - active：活跃
  - cached：被缓存，等待着被其他事物重用
  - to_free：insert 链表等待着被释放
  - to_purge:update链表等待着被purge
  - prepared：当前事务处于prepare状态
- trx_undo_last_log：undo log 链表中最后一个undo log header 的位置。
- trx_undo_fseg_header：当前链表对应着的段的header值。
- trx_undo_page_list：undo log 页面链表的基节点。

#### undo log header

InnoDB中一个事务在一个undo log页面链表中写入的undo log 是一组，所以存在了undo log header来记录当前组内的相关信息。而undo log header只会写在当前组中的第一个页面中（一个链表可能被其他事务复用，所以一个链表中可能存在多个组，多个undo log header，也就存在可能一个undo log 页有两个不同事物的undo log 的可能性）。

对应的属性如下：

- trx_undo_trx_id：事务id。
- trx_undo_trx_no：事务提交的序号。
- trx_undo_del_marks：本组中是否有deletemark 产生的undo log。
- trx_undo_log_start：本组中第一条undo log的偏移量
- trx_undo_xid_exists：本组中undo日志是否包含xid信息
- trx_undo_dict_trans：本组undo log是否由ddl产生。
- trx_undo_table_id：ddl语句操作的表的id。
- trx_undo_next_log：下一组undo log在页面中开始的偏移量
- trx_undo_prev_log：下一组undoog在页面中开始的偏移量
- trx_undo_history_node：代表history链表的节点。

所以对于没有被重用的undo log 页面链表来说，其结构如下：

![image_1d7cocjrk1dvm1ehg16da1di4mfb16.png-87kB](https://user-gold-cdn.xitu.io/2019/4/16/16a24a80f3cc070f?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

## 重用undo 页面

为了确保资源的合理利用，InnoDB中的undo 链表是可以被重用的，只需要满足以下规则：

- 链表中只存在一个undo 页面：如果存在多个undo 页面，那么当被重用后，也要维护上一个组对应的页面，存在很大的成本损耗。
- 该undo 页面使用空间小于页面大小的四分之三。

链表被重用时的策略：

- insert undo 链表：由于事务提交后insert undo 链表就无用了，所以重用时直接覆盖原有的undo log即可。
- update undo 链表：在又有undo log后写入。

注意，重用时会对页面中的各个header的属性进行调整。

## 回滚段

### 回滚段的概念

InnoDB中存在一个回滚段，该段中只存在一个header 页面，其中存放了各个undo log页面链表的头结点对应的位置。

header中的属性如下：

- trx_rseg_max_size：回滚段中管理的undo log链表中所以页面数量和的最大值。也就是InnoDB支持最多有多少个undolog 页。
- trx_rseg_history_size：history链表占用的页面数量
- trx_rseg_history：bistory链表的基节点。
- trx_rseg_fseg_header：当前回滚段对应的段的header 属性。
- trx_rseg_undo_slots：每个undo log 页面链表的first undo page 的页号集合，也就是 undo slot 集合。

### 从回滚段中申请undo 页面链表

初始情况下，每一个undo slot都被设置为fil_null，表示当前undo slot不指向任何页面。

当有事务需要分配undo log页面时，会向回滚段进行申请，此时会判断第一个undo slot是否为fil_null：

- 是，则在表空间新建一个undo log 段，将段中第一个页面的页号设置在undo slot中。
- 否，判断下一个undo slot。

目前只支持1024个slot。

当事务提交时，会对undo slot进行操作：

- 如果undo slot 对应的undo log 链表可以被重用，则将该undo log 页面链表状态设置为cached。将其undo slot 加入到缓存链表中。
  - 缓存链表有两类，分别为insert 缓存链表以及update 缓存链表。一个回滚段就对应着上述两个cached链表，如果有新事务要分配undo slot时，先从对应的cached链表中找。如果没有被缓存undo slot，才会到回滚段的Rollback Segment Header页面中再去找。
- 如果可以被重用：
  - 如果是insert链表，会将该undo log 页面链表状态设置为to_free，然后将该段进行释放。对应的undo slot设置为fil_null。
  - 如果是update链表，会将该undo log 页面链表状态设置为to_purge，对应的undo slot设置为fil_null。然后将原有的undo log 页面链表加入history链表中。

### 多个回滚段

由于一个回滚段最多只支持1024个事务并发，所以后续定义了128个回滚段，最大支持128 * 1024 = 131072个事务并发。

在系统表空间的5号页面中记录着这128个回滚段的位置。

![image_1d7h72gvlin31far16h11elj16tu1m.png-97kB](https://user-gold-cdn.xitu.io/2019/4/16/16a24a8116df4474?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

### 回滚段的分类

由于undo log 本质是为了事务的回滚，会通过redo log 进行持久化，但对于临时表的undo log，是不需要进行持久化的，所以回滚段存在两类，分别是：

- 普通表对应的回滚段：0以及33-127回滚段
- 临时表对应的回滚段：1-32号回滚段

### 为事务分配回滚段的具体过程

- 事务在执行过程中对普通表的记录首次做改动之前，首先会到系统表空间的第`5`号页面中分配一个回滚段（其实就是获取一个`Rollback Segment Header`页面的地址）。一旦某个回滚段被分配给了这个事务，那么之后该事务中再对普通表的记录做改动时，就不会重复分配了。

  使用传说中的`round-robin`（循环使用）方式来分配回滚段。比如当前事务分配了第`0`号回滚段，那么下一个事务就要分配第`33`号回滚段，下下个事务就要分配第`34`号回滚段，简单一点的说就是这些回滚段被轮着分配给不同的事务（就是这么简单粗暴，没啥好说的）。

- 在分配到回滚段后，首先看一下这个回滚段的两个`cached链表`有没有已经缓存了的`undo slot`，比如如果事务做的是`INSERT`操作，就去回滚段对应的`insert undo cached链表`中看看有没有缓存的`undo slot`；如果事务做的是`DELETE`操作，就去回滚段对应的`update undo cached链表`中看看有没有缓存的`undo slot`。如果有缓存的`undo slot`，那么就把这个缓存的`undo slot`分配给该事务。

- 如果没有缓存的`undo slot`可供分配，那么就要到`Rollback Segment Header`页面中找一个可用的`undo slot`分配给当前事务。

  从`Rollback Segment Header`页面中分配可用的`undo slot`的方式我们上边也说过了，就是从第`0`个`undo slot`开始，如果该`undo slot`的值为`FIL_NULL`，意味着这个`undo slot`是空闲的，就把这个`undo slot`分配给当前事务，否则查看第`1`个`undo slot`是否满足条件，依次类推，直到最后一个`undo slot`。如果这`1024`个`undo slot`都没有值为`FIL_NULL`的情况，就直接报错喽（一般不会出现这种情况）～

- 找到可用的`undo slot`后，如果该`undo slot`是从`cached链表`中获取的，那么它对应的`Undo Log Segment`已经分配了，否则的话需要重新分配一个`Undo Log Segment`，然后从该`Undo Log Segment`中申请一个页面作为`Undo页面`链表的`first undo page`。

- 然后事务就可以把`undo日志`写入到上边申请的`Undo页面`链表了！



## undo log 

> http://mysql.taobao.org/monthly/2015/04/01/

why，what、where、when、who、how。

why：为什么要有undo log？为了事务的回滚。为什么可以用来实现mvcc？因为undo log记录了每个事务对记录的操作行为，可以通过找到对应版本的记录。

what：undo log 是什么？事务中对表进行增删改操作都会生成undo log，用来记录本次操作对记录进行了什么变更。分为insert、delete、update 三类undo log。

where：undo log 在哪里产生？存储在哪里？在哪里产生作用？什么时候灭完？

- 在回滚段对应的undo log segment中对应的undo log 页面链表中对应的undo log 页面记录。
- 默认存储在共享表空间中。可以自定义存储位置。
- 分为对事务进行回滚以及实现mvcc机制。
- purge线程会通过history list对redo log 进行清理

when：什么时候产生？什么时候存储？什么时候产生作用？什么时候灭亡？

- 事务中对表进行增删改操作时产生。
- 事务提交时写入磁盘。
- 在对事务进行回滚，要么用户手动执行，要么事务的redo log 写入失败通过undo log进行回滚。
- purge线程的清理时间

who：

how：

- 如何产生：
  - 为事务分配回滚段的具体过程
- 如何存储：
  - 事务prepare：先写 undo log 对应的redo log 再写undo log，wal机制。
  - 事务commit：对回滚段中的undo slot进行处理，选择是否加入history listcached以及链表。
- 如何产生作用：
  - 回滚：析取老版本记录，做逆向操作即可。
  - 崩溃恢复：当InnoDB从崩溃中恢复时，需要将活跃的事务从undo log中提取出来，对于active的事务直接回滚，对于Prepare状态的事务，通过xid判断该事务对应的binlog是否已经记录，是则提交，否则回滚事务。
  - mvcc：由于在修改聚集索引记录时，总是存储了回滚段指针和事务id，可以通过该指针找到对应的undo 记录，通过事务Id来判断记录的可见性。当旧版本记录中的事务id对当前事务而言是不可见时，则继续向前构建，直到找到一个可见的记录或者到达版本链尾部。

