

​		我理解的世界由事物与行为组成，事物其本身的性质与行为对事物的影响，构成了我们所存在的世界。目前我的知识不允许我对世界有着更加深层次的思考，只是想通过对事物本身以及事物行为的思考，去认识一个又一个的事物。我认为一个事物的状态至少有三个阶段：产生 -> 功能实现 -> 灭亡，如果一个事务只是存在产生与灭亡，那么其本身对世界没有任何价值，我对其目前不太感兴趣，也不愿去深入了解。基于事物发展的三个阶段，存在这么一类事物，其本身会延迟产生作用，意味着其还需要进行存储，用于维持其状态直到其功能的实现。这也是大部分事物都会面临的一个阶段。

​		所以接下来，会以事物存在四个状态来探究一个事物，在对事物本身认知的了解下，对事物的生命周期进行一个剖析，以求能够对事物有一个较为充分的认知。如标题所言，我们以redo log为例，先明确其事务本质，再探究其在产生、存储、功能实现、灭亡四个阶段事物本身的变化以及对外界的影响。

​		针对事物本身的分析，我青睐借鉴对行为分析的5W1H方法，也就是why，what、where、when、who、how，其原本是用来分析某种行为，这里用来对事物本身进行探究。

**why**：为什么会有redo log？它是为了完成什么功能？

- 解决了InnoDB内部事务的持久性，确保事务中对数据的变更能够高效且可靠的记录下来。

**what**：redo log 是什么？

- 它记录了DML操作对于磁盘数据的物理操作。具有幂等性。

**where**：redo log在哪里产生？又会存储在哪里？

- 在InnoDB缓冲池出现，以环形存储的方式存储在磁盘中的logfile文件中。

**when**：redo log 是什么时间产生的？什么时间会存储到磁盘中？什么时候会产生作用？什么时候又会灭亡？

- 产生：事务中出现DML操作，会将内存中的数据页进行修改成为脏页。在将脏页刷新到磁盘前，会生成redo log，并将redo log 刷新到磁盘中。
- 存储：正常情况下，redo log会默认直接刷新到磁盘中，这项行为可被参数控制。
- 产生作用：当数据库重启时，存储引擎会默认对数据进行恢复，这里的恢复就是通过redo log进行。
- 灭亡：redo log 被覆盖后，就消亡了。

**who**：谁主导的产生？谁主导的存储？谁主导的产生作用？谁主导的灭亡？

- 产生、存储、灭亡：checkpoint机制
- 产生作用：InnoDB恢复机制

**how**：如何产生的？如何存储的？如何产生作用的？如何灭亡的？

- 产生：我们知道，redo log 的产生时间就是脏页刷新的时间，这里就会涉及到checkpoint机制，通过该机制确定脏页刷新的时间。所以需要我们细说checkpoint机制的实现。
- 存储：如何存储分为两个问题？如何存储到磁盘以及在磁盘如何存储？
  - 如何存储到磁盘：当redo log刷新的磁盘时，考虑到磁盘IO性能问题，会通过group commit机制批量调用fsync刷新操作系统文件缓存。在开启binlog 时，为了确保binlog与redolog数据的一致性，会通过MySQL的内部XA事务，在group commit与BLGC机制的共同作用下，确保两者记录的一致性。
  - 如何在磁盘存储？之前的redo log一直是抽象存在，当我们刷新到磁盘时，是通过log block进行保存的，当一个大小为512k的日志块没有存满时，会将其他页面对应的redo log也存放到这个日志块中保存，日志块头部会记录被保存的redo log 的信息。日志块是通过logfile进行组织的，logfile头部会存储最近两次刷新的checkpoint信息。
- 产生作用：在InnoDB进行恢复时，通过logfile 头部的最新一次checkpoint的LSN找到对应开始恢复的redo log，将其应用到对应的各个数据页中，这里为了使用顺序IO，会利用哈希表将同个页面的多个redo log按顺序一起应用。应用前会比较数据页面中包含的LSN的值，过滤掉不必要的刷新。
- 灭亡：redo log是以环形方式进行存储，新日志的写入伴随的就是旧日志的被覆盖。

