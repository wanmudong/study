

[TOC]

## Explain 详解

一条查询语句在经过`MySQL`查询优化器的各种基于成本和规则的优化会后生成一个执行计划。我们通过在具体的查询语句前边加一个`EXPLAIN`，可以看见具体的执行计划。

```sql
mysql> EXPLAIN SELECT 1;
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+----------------+
| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra          |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+----------------+
|  1 | SIMPLE      | NULL  | NULL       | NULL | NULL          | NULL | NULL    | NULL | NULL |     NULL | No tables used |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+----------------+
1 row in set, 1 warning (0.01 sec)
```

#### 执行计划输出中各列详解

##### table：

表名。EXPLAIN语句输出的每条记录都对应着某个单表的访问方法，该条记录的table列代表着该表的表名。

##### id：

- 查询语句中每出现一个`SELECT`关键字，mysql就会为它分配一个唯一的`id`值。
- 连接查询中记录的id值相同。可能存在子查询被优化成连接查询，从而执行计划中两条记录id相同
- union的查询语句中，会使用内部的临时表进行去重，此时临时表的执行计划中id为null。而union all由于不需要去重，所以不会使用临时表。

##### select_type：

每一个select代表的查询都存在select_type，用来定义该查询再整个查询语句的角色。

- simple：查询语句中不包含union或者子查询的查询都是simple类型。
- primary：查询语句中包含union或者子查询时，查询语句由多个小查询组成，此时最左侧的查询我们认为是最主要的查询，其查询类型为primary。
- union：对于包含union的查询，除了primary，其余均为查询类型均为union。
- union result：union查询使用临时表解决去重问题时，该临时表的查询类型为union result。
- subquery：对于不相关子查询，无法通过半连接方式优化，而是通过子查询物化的方式进行查询时，子查询的第一个select对应的查询类型即为subquery。物化表只会执行一遍。
- dependent subquery：对于相关子查询，无法通过半连接方式优化时，子查询的第一个select对应的查询类型即为dependent subquery。
- dependent union：对于查询语句中，子查询语句包含union，且内部小查询均依赖外层查询时，处理最左边的小查询，其余小查询的查询类型均为dependent union。
- driven： 对于采用物化的方式执行的包含派生表的查询，该派生表对应的子查询的查询类型就是driven。
- materialized：当查询优化器在执行包含子查询的语句时，选择将子查询物化之后与外层查询进行连接查询时，该子查询对应的查询类型为materialized。

##### partitions：

无分区的情况下，值为null。

##### type：

mysql对某个表的执行查询时的访问方法。

- system：当表中只有一条记录并且该表使用的存储引擎的统计数据是精确的，比如MyISAM、Memory，那么对该表的访问方法就是system。
- const：根据主键或者唯一二级索引列与常数进行等值匹配时，访问方法为system。
- eq_ref：在连接查询时，如果被驱动表是通过主键或者唯一二级索引列等值匹配（联合索引所有列等值比较）的方式进行访问的，则对该被驱动表的访问方法就是eq_ref。
- ref：通过二级索引进行等值匹配时，访问方法即为ref。
- fulltext：全文索引。由于Innodb不支持中文进行全文索引，基本可以忽略。
- ref_or_null：通过二级索引进行等值匹配此时该列的值也可以为null时，访问方法未ref_or_null。
- index_merge：通过intersection或union或sort-union三种索引进行索引合并时，访问方法为index_merge。
- unique_subquery：类似eq_ref，当in子查询语句被优化为exists子查询，且子查询可以使用主键进行等值匹配时，访问方法为unique_subquery。

- index_subquery：与unique_subquery类似，不过走的是普通索引。
- range：通过索引获取范围区间的记录时，访问方法可能为range。
- index：出现索引覆盖时，访问方法为index。
- all：全表扫描。

##### possible_key：

执行计划可能用到的索引。并非越多越好，越多可能mysql优化器选择的时间越长。

##### key：

实际使用的索引。

##### key_len：

当优化器决定使用索引时，该索引记录的最大长度由三部分组成：

- 对于使用固定长度类型的索引列来说，它实际占用的存储空间的最大长度就是该固定值
- 如果该索引列可以存储`NULL`值，则`key_len`比不可以存储`NULL`值时多1个字节。
- 对于变长字段来说，都会有2个字节的空间来存储该变长列的实际长度。

##### ref：

当使用索引列等值匹配的条件去执行查询时，也就是在访问方法是`const`、`eq_ref`、`ref`、`ref_or_null`、`unique_subquery`、`index_subquery`其中之一时，`ref`列展示的就是与索引列作等值匹配的数据来源，比如只是一个常数或者一个函数或者是某个列。

##### rows：

当优化器决定使用全表扫描时，rows代表预计需要扫描的行数。

如果使用索引来执行查询时，执行计划的`rows`列就代表预计扫描的索引记录行数。

##### filtered：

之前在分析连接查询的成本时提出过一个`condition filtering`的概念，就是`MySQL`在计算驱动表扇出时采用的一个策略：

- 如果使用的是全表扫描的方式执行的单表查询，那么计算驱动表扇出时需要估计出满足搜索条件的记录到底有多少条。
- 如果使用的是索引执行的单表扫描，那么计算驱动表扇出的时候需要估计出满足除使用到对应索引的搜索条件外的其他搜索条件的记录有多少条。

执行计划的`filtered`列代表查询优化器预测的扇出记录数是占rows的多少。当rows =9879 、filteed = 10%时，意味着扇出值为987.9，预计还要对被驱动表执行987次查询。

##### extra：

- no table used ：查询语句没有from子句。
- impossible where：where子句永远为false。
- no matching min/max row：查询列表出有聚集函数，但没有where子句匹配的记录。
- using index：出现索引覆盖时，会提示该信息。
- using index condition：查询语句在执行过程中使用索引条件下推（using index condition ），即ICP。
- using where：出现全表扫描，且where子句中有对该表的搜索条件时。
- using join buffer（block nested loop）：无法优化查询时，在内存中分配一个名为join buffer的内存块来加快查询速度，将驱动表符合条件的记录放入join buffer中，也就是基于块的嵌套循环算法。
- not exists：使用左外链接时，如果`WHERE`子句中包含要求被驱动表的某个列等于`NULL`值的搜索条件，而且那个列又是不允许存储`NULL`值的，那么在该表的执行计划的`Extra`列就会提示`Not exists`额外信息，这意味着必定是驱动表的记录在被驱动表中找不到匹配`ON`子句条件的记录才会把该驱动表的记录加入到最终的结果集。所以就相当于可以优化为如果不存在，就符合查询条件。
- using intersect、using union、using sort_union：出现了索引合并的情况。
- zero limit：limit子句参数为0。
- using filesort：当查询语句需要排序，但没有通过索引进行排序，而是在内存或者磁盘进行排序的方式称为文件排序。即filesort。注意group by会默认进行文件排序。
- using temporary：通过建立临时表进行查询。
- start temporary，end temporary：当执行策略为`DuplicateWeedout`时，也就是通过建立临时表来实现为外层查询中的记录进行去重操作时，驱动表查询执行计划的`Extra`列将显示`Start temporary`提示，被驱动表查询执行计划的`Extra`列将显示`End temporary`提示
- looseScan：在将`In`子查询转为`semi-join`时，如果采用的是`LooseScan`执行策略，则在驱动表执行计划的`Extra`列就是显示`LooseScan`提示。
- firstMatch：在将`In`子查询转为`semi-join`时，如果采用的是`FirstMatch`执行策略，则在被驱动表执行计划的`Extra`列就是显示`FirstMatch(tbl_name)`提示。



#### JSON格式的执行计划

在`EXPLAIN`单词和真正的查询语句中间加上`FORMAT=JSON`。

``` sql
mysql> EXPLAIN FORMAT=JSON SELECT * FROM s1 INNER JOIN s2 ON s1.key1 = s2.key2 WHERE s1.common_field = 'a'\G
*************************** 1. row ***************************

EXPLAIN: {
  "query_block": {
    "select_id": 1,     # 整个查询语句只有1个SELECT关键字，该关键字对应的id号为1
    "cost_info": {
      "query_cost": "3197.16"   # 整个查询的执行成本预计为3197.16
    },
    "nested_loop": [    # 几个表之间采用嵌套循环连接算法执行
    
    # 以下是参与嵌套循环连接算法的各个表的信息
      {
        "table": {
          "table_name": "s1",   # s1表是驱动表
          "access_type": "ALL",     # 访问方法为ALL，意味着使用全表扫描访问
          "possible_keys": [    # 可能使用的索引
            "idx_key1"
          ],
          "rows_examined_per_scan": 9688,   # 查询一次s1表大致需要扫描9688条记录
          "rows_produced_per_join": 968,    # 驱动表s1的扇出是968
          "filtered": "10.00",  # condition filtering代表的百分比
          "cost_info": {
            "read_cost": "1840.84",     # 稍后解释
            "eval_cost": "193.76",      # 稍后解释
            "prefix_cost": "2034.60",   # 单次查询s1表总共的成本
            "data_read_per_join": "1M"  # 读取的数据量
          },
          "used_columns": [     # 执行查询中涉及到的列
            "id",
            "key1",
            "key2",
            "key3",
            "key_part1",
            "key_part2",
            "key_part3",
            "common_field"
          ],
          
          # 对s1表访问时针对单表查询的条件
          "attached_condition": "((`xiaohaizi`.`s1`.`common_field` = 'a') and (`xiaohaizi`.`s1`.`key1` is not null))"
        }
      },
      {
        "table": {
          "table_name": "s2",   # s2表是被驱动表
          "access_type": "ref",     # 访问方法为ref，意味着使用索引等值匹配的方式访问
          "possible_keys": [    # 可能使用的索引
            "idx_key2"
          ],
          "key": "idx_key2",    # 实际使用的索引
          "used_key_parts": [   # 使用到的索引列
            "key2"
          ],
          "key_length": "5",    # key_len
          "ref": [      # 与key2列进行等值匹配的对象
            "xiaohaizi.s1.key1"
          ],
          "rows_examined_per_scan": 1,  # 查询一次s2表大致需要扫描1条记录
          "rows_produced_per_join": 968,    # 被驱动表s2的扇出是968（由于后边没有多余的表进行连接，所以这个值也没啥用）
          "filtered": "100.00",     # condition filtering代表的百分比
          
          # s2表使用索引进行查询的搜索条件
          "index_condition": "(`xiaohaizi`.`s1`.`key1` = `xiaohaizi`.`s2`.`key2`)",
          "cost_info": {
            "read_cost": "968.80",      # 稍后解释
            "eval_cost": "193.76",      # 稍后解释
            "prefix_cost": "3197.16",   # 单次查询s1、多次查询s2表总共的成本
            "data_read_per_join": "1M"  # 读取的数据量
          },
          "used_columns": [     # 执行查询中涉及到的列
            "id",
            "key1",
            "key2",
            "key3",
            "key_part1",
            "key_part2",
            "key_part3",
            "common_field"
          ]
        }
      }
    ]
  }
}
1 row in set, 2 warnings (0.00 sec)
```

我们看一下cost_info属性：

``` sql
"cost_info": {
    "read_cost": "1840.84",
    "eval_cost": "193.76",
    "prefix_cost": "2034.60",
    "data_read_per_join": "1M"
}
```

Resd_cost = io成本+rows*（1-filter）cpu成本

eval_cost = rows × filter 检测成本

prefix_cost = 单独查询表的成本 = read_cost + eval_cost（被驱动表需要额外加上驱动表的该项成本）

data_read_per_join 表示需要读取的数据量

#### Extented EXPLAIN

可以使用`SHOW WARNINGS`语句查看与这个查询的执行计划有关的一些扩展信息

``` sql
mysql> EXPLAIN SELECT s1.key1, s2.key1 FROM s1 LEFT JOIN s2 ON s1.key1 = s2.key1 WHERE s2.common_field IS NOT NULL;
+----+-------------+-------+------------+------+---------------+----------+---------+-------------------+------+----------+-------------+
| id | select_type | table | partitions | type | possible_keys | key      | key_len | ref               | rows | filtered | Extra       |
+----+-------------+-------+------------+------+---------------+----------+---------+-------------------+------+----------+-------------+
|  1 | SIMPLE      | s2    | NULL       | ALL  | idx_key1      | NULL     | NULL    | NULL              | 9954 |    90.00 | Using where |
|  1 | SIMPLE      | s1    | NULL       | ref  | idx_key1      | idx_key1 | 303     | xiaohaizi.s2.key1 |    1 |   100.00 | Using index |
+----+-------------+-------+------------+------+---------------+----------+---------+-------------------+------+----------+-------------+
2 rows in set, 1 warning (0.00 sec)

mysql> SHOW WARNINGS\G
*************************** 1. row ***************************
  Level: Note
   Code: 1003
Message: /* select#1 */ select `xiaohaizi`.`s1`.`key1` AS `key1`,`xiaohaizi`.`s2`.`key1` AS `key1` from `xiaohaizi`.`s1` join `xiaohaizi`.`s2` where ((`xiaohaizi`.`s1`.`key1` = `xiaohaizi`.`s2`.`key1`) and (`xiaohaizi`.`s2`.`common_field` is not null))
1 row in set (0.00 sec)

```



#### optimizer trace

mysql内可以通过optimizer trace 来查看优化器生成执行计划的整个过程：

``` sql
# 1. 打开optimizer trace功能 (默认情况下它是关闭的):
SET optimizer_trace="enabled=on";

# 2. 这里输入你自己的查询语句
SELECT ...; 

# 3. 从OPTIMIZER_TRACE表中查看上一个查询的优化过程
SELECT * FROM information_schema.OPTIMIZER_TRACE;

# 4. 可能你还要观察其他语句执行的优化过程，重复上边的第2、3步
...

# 5. 当你停止查看语句的优化过程时，把optimizer trace功能关闭
SET optimizer_trace="enabled=off";
```

