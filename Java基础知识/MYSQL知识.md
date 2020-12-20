# MYSQL知识

##### 1.数据库架构设计

![](http://static.xiany.top/markdown/20201128235006.png)

##### 2.索引模块

1. 为什么要使用索引？

   快速查询数据

2. 索引的数据结构
   * 生成索引，建立二叉查找树进行二分查找
   * 生成索引，建立B-Tree结构进行查找
   * 生成索引，建立B+-Tree结构进行查找
   * 生成索引，建立Hash结构进行查找

3. 二叉树结构

   [这篇二分查找树的内容，有几年项目经验也不一定会！]: https://my.oschina.net/u/4477286/blog/4295741

    对于每一个结点，左孩子小于该节点，右孩子大于该结点。
   
4. B -Tree树

   [漫画算法：什么是 B 树？]: https://www.jianshu.com/p/8b653423c586
   [平衡二叉树、B树、B+树、B*树 理解其中一种你就都明白了]: https://zhuanlan.zhihu.com/p/27700617

   定义:

   * 根节点至少包含二个孩子

   * 数中每个节点最多包含有m个孩子(m>=2)

   * 除根节点和叶子节点外，其他每个节点至少有celi(m/2)个孩子(例如:3/2=1.5 取2，类似最大正整数一样)

   * 所有叶子节点都位于同一层

   * 假设每个非终端结点包含n个关键字信息，其中:

     a. Ki(i=1...n)为关键字，且关键字按顺序升序排序K(i-1)<ki

     b. 关键字的个数N必须满足：[ceil(m/2)-1]<=n<=m-1(关键字的字数永远比他孩子节点少一个)

     c. 非叶子节点的指针：P[1],P[2],...,P[M];其中P[1]指向关键字小于K[1]的子数，P[M]指向关键字大于K[M-1]的子树，其它P[i]指向关键字属于(K[i-1],K[i])的子树

5.B+ - Tree

   * 与B-Tree的定义一样，B+-Tree添加了新的定义
   * 非叶子节点的子树指针与关键字个数相同
   * 非叶子节点的子树指针P[i]，指向关键字值[K[i],K[i+1]]的子树
   * 非叶子节点仅用来做索引，数据都保存在叶子节点中
   * 所有叶子节点均有一个链指针指向下一个叶子节点

   为什么数据库选择B+-Tree做索引的原因？

   * B+Tree的磁盘读写代价更低
   * B+Tree的查询效率更加稳定
   * B+Tree更有利于对数据库的扫描

6.Hash索引

* Hash索引仅仅能满足”=”,”IN”和”<=>”查询，不能使用范围查询。

由于 MySQL Hash索引比较的是进行 Hash 运算之后的 Hash 值，所以它只能用于等值的过滤，不能用于基于范围的过滤，因为经过相应的 Hash 算法处理之后的 Hash 值的大小关系，并不能保证和Hash运算前完全一样。

* Hash索引无法被用来避免数据的排序操作。

由于Hash索引中存放的是经过 Hash 计算之后的 Hash 值，而且Hash值的大小关系并不一定和 Hash 运算前的键值完全一样，所以数据库无法利用索引的数据来避免任何排序运算；

* MySQL Hash索引不能利用部分索引键查询。

对于组合索引，Hash 索引在计算 Hash 值的时候是组合索引键合并后再一起计算 Hash 值，而不是单独计算 Hash 值，所以通过组合索引的前面一个或几个索引键进行查询的时候，Hash 索引也无法被利用。

* MySQL Hash索引在任何时候都不能避免表扫描。

前面已经知道，Hash 索引是将索引键通过 Hash 运算之后，将 Hash运算结果的 Hash 值和所对应的行指针信息存放于一个 Hash 表中，由于不同索引键存在相同 Hash 值，所以即使取满足某个 Hash 键值的数据的记录条数，也无法从 Hash 索引中直接完成查询，还是要通过访问表中的实际数据进行相应的比较，并得到相应的结果。

* MySQL Hash索引遇到大量Hash值相等的情况后性能并不一定就会比B-Tree索引高。

对于选择性比较低的索引键，如果创建 Hash 索引，那么将会存在大量记录指针信息存于同一个 Hash 值相关联。这样要定位某一条记录时就会非常麻烦，会浪费多次表数据的访问，而造成整体性能低下。

7. BitMap索引（目前只有Oracle支持,使用的较少）

   [位图索引:原理（BitMap index）——浅显易懂]: https://blog.csdn.net/qq_24236769/article/details/75801687

8. 密集索引和稀疏索引的区别

   [密集索引和稀疏索引的区别]: https://blog.csdn.net/fansenjun/article/details/85647734

   innodb的索引和数据是存储到一起，myisam是分开存储的

9. 如何定位并且优化慢查询的Sql?

   * 根据慢日志定位慢查询sql

     ```mysql
     -- 查询系统变量
     show variavles like '%quer%';
     -- 查询慢查询记录数，本次慢sql的条数，重启客户端工具就会清零
     show status like '%slow_queries%';
     
     -- 此方式设置环境变量数据库重启后会恢复到默认的状态，如果要永久有效liunx系统修改my.cnf文件，Windows修改my.ini文件
     -- 打开慢查询记录
     set global slow_query_log = on;
     -- 设置慢查询时间,设置完成需要重新打开sql连接工具才生效
     set global long_query_time = 1;
     ```

     | variables_name                         | value                                | desc                          |
     | -------------------------------------- | ------------------------------------ | ----------------------------- |
     | binlog_rows_query_log_events           | OFF                                  |                               |
     | ft_query_expansion_limit               | 20                                   |                               |
     | have_query_cache                       | NO                                   |                               |
     | log_queries_not_using_indexes          | OFF                                  |                               |
     | log_throttle_queries_not_using_indexes | 0                                    |                               |
     | long_query_time                        | 10.000000                            | sql超过多少此时间记录到慢日志 |
     | query_alloc_block_size                 | 8192                                 |                               |
     | query_prealloc_size                    | 8192                                 |                               |
     | slow_query_log                         | OFF                                  | 慢日志是否打开                |
     | slow_query_log_file                    | /var/lib/mysql/b2fee5a1dd8a-slow.log | 慢日志文件存放地址            |

   * 使用explain等工具分析sql

     ```mysql
     -- 使用explain分析sql语句执行过程
     explain select name from person_info_large;
     
     |id|select-type|table            |partitions|type|possible_keys|key|key_len|ref|rows|filtered|     extra    |
     |1|SIMPLE      |person_info_large|          |ALL |             |   |       |   |36966|100.00 |Using filesort|
     ```

     * id：标识符，表示执行顺序

     * select _type：查询类型

     * table：输出行所引用的表

     * partitions：使用的哪个分区，需要结合表分区才可以看到

     * type：表示按某种类型来查询，例如按照索引类型查找，按照范围查找。从最好到最差的连接类型为const、eq_reg、ref、range、index和all

     * possible_keys：可能用到的索引，保存的是索引名称，如果是多个索引的话，用逗号隔开

     * key：实际用到的索引，保存的是索引名称，如果是多个索引的话，用逗号隔开

     * key_len：表示本次查询中，所选择的索引长度有多少字节

     * ref：显示索引的哪一列被使用了，如果可能的话，是一个常数

     * rows：显示mysql认为执行查询时必须要返回的行数

     * filtered：通过过滤条件之后对比总数的百分比

     * extra：额外的信息，例如：using file sort ，using where， using join buffer，using index等

       [MySQL EXPLAIN详解]: https://www.jianshu.com/p/ea3fc71fdc45

   * 修改sql尽量使sql走索引

     ```mysql
     -- 不走索引
     SELECT name FROM person_info_large ORDER BY NAME DESC;
     
     # Time: 2020-11-29T07:04:27.621266Z
     # User@Host: root[root] @  [14.120.35.208]  Id:  2014
     # Query_time: 2.474162  Lock_time: 0.000220 Rows_sent: 37180  Rows_examined: 74360
     # SET timestamp=1606633465;
     # SELECT name FROM person_info_large ORDER BY NAME DESC;
     
     -- 走唯一索引
     SELECT account FROM person_info_large ORDER BY account DESC;
     
     -- 走索引查询时间小于1秒，慢查询中并未记录sql
     explain SELECT account FROM person_info_large ORDER BY account DESC;
     -- 查看执行过程type已经使用了索引key为account
     |id|select-type|table            |partitions|type|possible_keys|key    |key_len|ref|rows|filtered|     extra    |
     |1|SIMPLE      |person_info_large|          |index |           |account| 33    |   |36966|100.00 |Backward index scan; Using index|
     
     -- 为name字段添加索引
     alter table person_info_large add index idx_name(name);
     -- 在查看name查询的执行过程,发现sql语句已经走索引了
     explain SELECT name FROM person_info_large ORDER BY NAME DESC;
     
     |id|select-type|table            |partitions|type|possible_keys|key    |key_len|ref|rows|filtered|     extra    |
     |1|SIMPLE      |person_info_large|          |index |           |idx_name| 63    |   |36966|100.00 |Backward index scan; Using index|
     
     -- 虽然走了索引但是，查询快了0.3秒并无很大的变化
     # Time: 2020-11-29T07:14:37.185190Z
     # User@Host: root[root] @  [14.120.35.208]  Id:  2014
     # Query_time: 2.163776  Lock_time: 0.000208 Rows_sent: 37180  Rows_examined: 37180
     # SET timestamp=1606634075;
     # SELECT name FROM person_info_large ORDER BY NAME DESC;
     ```

     扩展知识: count是走的那个索引?

     ```mysql
     EXPLAIN SELECT COUNT(id) FROM person_info_large;
     
     EXPLAIN SELECT COUNT(id) FROM person_info_large FORCE INDEX(PRIMARY);
     
     |id|select-type|table            |partitions|type|possible_keys|key    |key_len|ref|rows|filtered|     extra    |
     |1|SIMPLE      |person_info_large|          |index |           |account| 33    |   |36966|100.00 | BUsing index |
     ```

       走那个索引取决于mysql的查询优化的选择，会根据表里索引进行分析，选择一个最快的索引进行查询。

10. Mysql的左前匹配原则

    1.按照文档, 更准确的说法应该是最左前缀原则, 即如果你创建一个联合索引, 那 这个索引的任何前缀都会用于查询, (col1, col2, col3)这个联合索引的所有前缀 就是(col1), (col1, col2), (col1, col2, col3), 包含这些列的查询都会启用索 引查询.
    2.其他所有不在最左前缀里的列都不会启用索引, 即使包含了联合索引里的部分列 也不行. 即上述中的(col2), (col3), (col2, col3) 都不会启用索引去查询.
     **注意, (col1, col3)会启用(col1)的索引查询**

    3.最左前缀匹配原则，非常重要的原则，mysql会一直向右匹配直到遇到范围查询(>,<,between,like)就停止匹配，比如a=3 and b=4 and c > 5 and d = 6如果建立(a,b,c,d)顺序的索引，d是不能用到索引的，如果建立(a,b,d,c)的索引则都可以用到，a，b，d的顺序可以任意调整。

    4.=和in可以乱序，比如a=1 and b =2 and c =3 建立(a,b,c)索引可以任意顺序，mysql的查询优化器会帮你优化成可以识别的形式

##### 3.数据库锁

3.1 MyISAM与Innodb关于锁方面的区别是什么？

 * MyISAM默认用的是表级锁，不支持行级锁
 * InnoDB默认用的是行级锁，也支持表级锁

3.2 MyISAM表锁

```mysql
-- 加锁
lock tables 表名 read | write;

-- 释放锁
unlock tables;
```

[重新认识Mysql之MyISAM表锁]: https://segmentfault.com/a/1190000019899944

3.2 Innodb行锁和表锁

```mysql
-- 查看数据库是否是自动提交事务的
show variables like 'autocommit';

-- 关闭自动提交，只对当前session有效
set autocommit = 0;

-- 开始事务
begin;
-- 回滚事务
rollback;
-- 提交事务
commit;
-- 开始sql语句读锁
select * from person_info_large where id = 3 lock in share mode;

```

3.3 MyISAM适用场景

* 频繁执行全表count语句
* 对数据进行增删改的频率不高，但是查询比较频繁
* 没有事务

3.4 InnoDB适用场景

* 数据整删改查都相当频繁
* 可靠性要求比较高，要求支持事务

3.5 数据库锁的分类

* 按锁的粒度划分，可分为表级锁，行级锁，页级锁
* 按锁级别划分，可分为共享锁，排它锁
* 按加锁方式划分，可分为自动锁，显示锁
* 按操作划分，可分为DML锁，DDL锁
* 按使用方式划分，可分为乐观锁，悲观锁

3.6 事务的四大特性

* 原子性（Atomicity）
  原子性是指事务包含的所有操作要么全部成功，要么全部失败回滚，因此事务的操作如果成功就必须要完全应用到数据库，如果操作失败则不能对数据库有任何影响。
* 2  一致性（Consistency）
  　　一致性是指事务必须使数据库从一个一致性状态变换到另一个一致性状态，也就是说一个事务执行之前和执行之后都必须处于一致性状态。
    　　拿转账来说，假设用户A和用户B两者的钱加起来一共是5000，那么不管A和B之间如何转账，转几次账，事务结束后两个用户的钱相加起来应该还得是5000，这就是事务的一致性。
* 3  隔离性（Isolation
  　　隔离性是当多个用户并发访问数据库时，比如操作同一张表时，数据库为每一个用户开启的事务，不能被其他事务的操作所干扰，多个并发事务之间要相互隔离。
    　　即要达到这么一种效果：对于任意两个并发的事务T1和T2，在事务T1看来，T2要么在T1开始之前就已经结束，要么在T1结束之后才开始，这样每个事务都感觉不到有其他事务在并发地执行。
* 4  持久性（Durability）
  　　持久性是指一个事务一旦被提交了，那么对数据库中的数据的改变就是永久性的，即便是在数据库系统遇到故障的情况下也不会丢失提交事务的操作。
    　　例如我们在使用JDBC操作数据库时，在提交事务方法后，提示用户事务操作完成，当我们程序执行完成直到看到提示后，就可以认定事务以及正确提交，即使这时候数据库出现了问题，也必须要将我们的事务完全执行完成，否则就会造成我们看到提示事务处理完毕，但是数据库因为故障而没有执行事务的重大错误。

3.7 事务并发访问引起的问题如何避免

* 更新丢失--mysql所有事务隔离级别在数据库层面上都可避免（**READ_UNCOMMITTED** ）

  | 取款事务                  | 存款事务                |
  | ------------------------- | ----------------------- |
  | 开始事务                  | 开始事务                |
  | 查询转账余额为100元       |                         |
  |                           | 查询转账余额为100元     |
  |                           | 存入20元，余额变为120元 |
  |                           | 提交事务                |
  | 取出10元，余额改为90元    |                         |
  | 回滚事务，余额恢复为100元 | 更新丢失                |

* 脏读-READ-COMMITTED事务隔离级别以上避免

  ```mysql
  -- 查看数据事务隔离级别,mysql 8之前select @@tx_isolation;
  select @@transaction_isolation; 
  show variables like '%transaction_isolation%';
  
  -- 默认事务隔离级别是REPEATABLE-READ,设置事务隔离级别为读未提交的级别
  set session transaction isolation level read uncommitted;
  
  -- session1
  start transaction;
  
  update account_innodb set balance = balance - 100 where id = 1;
  
  select * from account_innodb where id = 1;
  
  -- | id | name   | balance |
  -- | 1  | xiaomi | 900     |
  -- 此时rollback数据，session1 balance的值为100
  rollback;
  
  -- commit;
  
  
  -- session2
  start transaction;
  
  -- session1未提交事务，session2已经可以读取到提交后的数据
  select * from account_innodb where id = 1;
  
  -- | id | name   | balance |
  -- | 1  | xiaomi | 900     |
  
  update account_innodb set balance = 900 + 200 where id =1;
  
  select * from account_innodb where id = 1;
  -- 查询结果为1100不是1200数据为错误的
  -- | id | name   | balance |
  -- | 1  | xiaomi | 1100    |
  
  commit;
  
  -- 如何解决上面问题,把事务隔离级别改为READ_COMMITTED;
  set session transaction isolation level read committed;
  ```

* 不可重复读--REPEATABLE-READ事务隔离级别以上可以避免

  ```mysql
  -- session1
  start transaction;
  
  SELECT * FROM account_innodb WHERE id = 1;
  
  -- | id | name   | balance |
  -- | 1  | xiaomi | 900     |
  
  SELECT * FROM account_innodb WHERE id = 1;
  -- session1再次查询发现数据变为1000了,重复读取的情况下查询得到别人提交后的数据，因该结果还是原来的不变
  -- | id | name   | balance |
  -- | 1  | xiaomi | 1000    |
  commit;
  
  -- session2
  start transaction;
  -- 表里数据为900，加到1000
  UPDATE account_innodb SET balance = balance + 100 WHERE id = 1;
  
  SELECT * FROM account_innodb WHERE id = 1;
  -- 查询结果为1000
  -- | id | name   | balance |
  -- | 1  | xiaomi | 1000    |
  commit;
  
  -- 解决方法把事务隔离级别设置为REPEATABLE-READ
  set session transaction isolation level repeatable read;
  
  ```

* 幻读-SERIALIZABLE事务隔离级别可避免

  ```mysql
  -- 设置事务隔离级别
  set session transaction isolation level read committed;
  -- session1
  START TRANSACTION;
  -- 1.查询数据加锁
  SELECT * FROM account_innodb LOCK IN SHARE MODE;
  -- 4.修改数据成功，修改了4条数据，存在数据欢读现象(正常应该只能修改3条)
  UPDATE account_innodb SET balance = 1000;
  
  commit;
  
  -- session2
  START TRANSACTION;
  -- 2.添加数据到表中，此时不会卡住，会直接执行成功
  INSERT INTO account_innodb VALUES(4,'wangwu',100);
  -- 3.session2提交了事务
  commit;
  
  -- 设置事务隔离级别
  set session transaction isolation level repeatable read;
  
  -- session1
  start transaction;
  -- 1.读取数据添加锁(3条)
  SELECT * FROM account_innodb LOCK IN SHARE MODE;
  -- 3.修改数据只会影响到(上面3条),如果不加锁session2不会卡住新增成功提交事务，会导致当前修改了4条数据造成了幻读
  UPDATE account_innodb SET balance = 1000;
  -- 4.提交事务
  commit;
  
  -- session2
  start transaction;
  -- 2.进行数据新增,会一直卡住等session1把事务提交了才能完成
  INSERT INTO account_innodb VALUES(4,'wangwu',100);
  -- 3.新增成功,提交事务
  commit;
  
  -- 解决幻读的方式除了上面那种还可以把事务隔离级别设置为SERIALIZABLE,在此隔离级别下所有的操作都会加上锁
  set session transaction isolation level serializable;
  ```

  

| 事务隔离级别               | 更新丢失 | 脏读 | 不可重复读（侧重与数据修改) | 幻读(侧重于新增或删除)           |
| -------------------------- | -------- | ---- | --------------------------- | -------------------------------- |
| 未提交读(read uncommitted) | 避免     | 发生 | 发生                        | 发生                             |
| 已提交读(read committed)   | 避免     | 避免 | 发生                        | 发生                             |
| 可重复读(repeatable read)  | 避免     | 避免 | 避免                        | 发生(可以采取查询加锁的方式避免) |
| 串行化(serializable)       | 避免     | 避免 | 避免                        | 避免(所有操作加锁)               |

事务隔离级别越高串行化操作会降低数据的并发度，应该根据业务的需要选择合适的事务隔离级别

3.8 InnoDB可重复读隔离级别下如何避免幻读

* 表象：快照读(非阻塞读)--伪MVCC

  * 当前读：select ...lock in share mode(共享锁),select ... for update(排他锁)

  * 当前读：insert,update,delete(排他锁)当前读是指加锁锁的CRUD语句

    ![当前读](http://static.xiany.top/markdown/20201129220444.png)

  * 快照读：不加锁的非阻塞读，select(事务隔离级别不是serializable)

    ```mysql
    -- 事务隔离级别read committed
    -- session1 
    start transaction;
    -- 3.快照读
    select * from account_innodb where id = 2;
    -- 4.当前读
    select * from account_innodb where id = 2 lock in share mode;
    -- read committed 级别下当前读和快照读的结果一致
    commit;
    
    -- session2
    start transaction;
    -- 1.修改表数据，在去session1中读
    update account_innodb set balance = 300 where id =2;
    -- 2.提交事务
    commit;
    
    
    -- 事务隔离级别repeatable read
    -- session1 
    start transaction;
    -- 3.快照读 600(原始300)
    select * from account_innodb where id = 2;
    -- 4.当前读 300
    select * from account_innodb where id = 2 lock in share mode;
    -- repeatable read 级别下当前读和快照读的结果不一致
    commit;
    
    -- session2
    start transaction;
    -- 1.修改表数据，在去session1中读
    update account_innodb set balance = 300 where id =2;
    -- 2.提交事务
    commit;
    
    
    -- 事务隔离级别repeatable read,修改操作顺序
    -- session1 
    start transaction;
    -- 3.快照读 0
    select * from account_innodb where id = 2;
    -- 3.当前读 0
    select * from account_innodb where id = 2 lock in share mode;
    -- repeatable read 级别下当前读和快照读的结果一致
    -- 在repeatable read情况下数据读取的顺序很重要取决数据读取的版本,开启session1不进行查询，直接在session2修改数据提交数据，在来session1快照读就能获取最新的数据
    commit;
    
    -- session2
    start transaction;
    -- 1.修改表数据，在去session1中读
    update account_innodb set balance = 0 where id =2;
    -- 2.提交事务
    commit;
    ```

    

* 内在：next-key锁(行锁+gap锁)

3.9 RC，RR级别下的InnoDB的非阻塞读如何实现

* 数据行里的DB_TRX_ID,DB_ROLL_PTR_,DB_ROW_ID字段
* undo日志
* read view