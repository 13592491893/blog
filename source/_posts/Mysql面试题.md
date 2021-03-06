---
abbrlink: 62688
title: mysql面试题
comments: true
toc: true
description: mysql面试题
top_img: https://gitee.com/gsshy/picgo/raw/master/img/top.jpg
categories:
  - sql
tags:
  - mysql
  - sql
  - 面试
date: 2021-6-21 16:00:00
---
# mysql面试题

作者：阿亮
链接：https://zhuanlan.zhihu.com/p/116866170
来源：知乎
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

## 一、数据库字段设计

### 1、为什么要一定要设置主键?

其实这个不是一定的，有些场景下，小系统或者没什么用的表，不设置主键也没关系，mysql最好是用自增主键，主要是以下两个原因：果定义了主键，那么InnoDB会选择主键作为聚集索引、如果没有显式定义主键，则innodb 会选择第一个不包含有NULL值的唯一索引作为主键索引、如果也没有这样的唯一索引，则innodb 会选择内置6字节长的ROWID作为隐含的聚集索引。所以，反正都要生成一个主键，那你还不如自己指定一个主键，提高查询效率！

### 2、主键是用自增还是UUID?

最好是用自增主键，主要是以下两个原因：

　　1. 如果表使用自增主键，那么每次插入新的记录，记录就会顺序添加到当前索引节点的后续位置，当一页写满，就会自动开辟一个新的页。
　　2. 如果使用非自增主键（如uuid），由于每次插入主键的值近似于随机，因此每次新纪录都要被插到索引页的随机某个位置，此时MySQL为了将新记录插到合适位置而移动数据，甚至目标页面可能已经被回写到磁盘上而从缓存中清掉，此时又要从磁盘上读回来，这增加了很多开销，同时频繁的移动、分页操作造成索引碎片，得到了不够紧凑的索引结构，后续不得不通过OPTIMIZE TABLE来重建表并优化填充页面。

不过，也不是所有的场景下都得使用自增主键，可能场景下，主键必须自己生成，不在乎那些性能的开销。那也没有问题。

### 3、自增主机用完了怎么办?

在mysql中，Int整型的范围（-2147483648~2147483648），约20亿！因此不用考虑自增ID达到最大值这个问题。而且数据达到千万级的时候就应该考虑分库分表了。

### 4、主键为什么不推荐有业务含义?

最好是主键是无意义的自增ID，然后另外创建一个业务主键ID，

因为任何有业务含义的列都有改变的可能性,主键一旦带上了业务含义，那么主键就有可能发生变更。主键一旦发生变更，该数据在磁盘上的存储位置就会发生变更，有可能会引发页分裂，产生空间碎片。

还有就是，带有业务含义的主键，不一定是顺序自增的。那么就会导致数据的插入顺序，并不能保证后面插入数据的主键一定比前面的数据大。如果出现了，后面插入数据的主键比前面的小，就有可能引发页分裂，产生空间碎片。

### 5、货币字段用什么类型?
货币字段一般都用 Decimal类型，
float和double是以二进制存储的，数据大的时候，可能存在误差。

以下是FLOAT和DOUBLE的区别：

浮点数以8位精度存储在FLOAT中，并且有四个字节。

浮点数存储在DOUBLE中，精度为18位，有八个字节。

### 6、时间字段用什么类型?

这个看具体情况和实际场景，timestamp ，datatime ，bigint 都行！

timestamp，该类型是四个字节的整数，它能表示的时间范围为1970-01-01 08:00:01到2038-01-19 11:14:07。2038年以后的时间，是无法用timestamp类型存储的。
但是它有一个优势，timestamp类型是带有时区信息的。一旦你系统中的时区发生改变，例如你修改了时区，该字段的值会自动变更。这个特性用来做一些国际化大项目，跨时区的应用时，特别注意！

datetime，占用8个字节，它存储的时间范围为1000-01-01 00:00:00 ~ 9999-12-31 23:59:59。显然，存储时间范围更大。但是它坑的地方在于，它存储的是时间绝对值，不带有时区信息。如果你改变数据库的时区，该项的值不会自己发生变更！

bigint，也是8个字节，自己维护一个时间戳，查询效率高，不过数据写入，显示都需要做转换。这种存储方式的具有 Timestamp 类型的所具有一些优点，并且使用它的进行日期排序以及对比等操作的效率会更高，跨系统也很方便，毕竟只是存放的数值。缺点也很明显，就是数据的可读性太差了，你无法直观的看到具体时间。

### 7、为什么不直接存储图片、音频、视频等大容量内容?

我们在实际应用中，都是文件形式存储的。mysql中，只存文件的存放路径。虽然mysql中blob类型可以用来存放大容量文件，但是，我们在生产中，基本不用！ 主要有如下几个原因：

　　1. Mysql内存临时表不支持TEXT、BLOB这样的大数据类型，如果查询中包含这样的数据，查询效率会非常慢。

　　2. 数据库特别大，内存占用高，维护也比较麻烦。

　　3. binlog太大，如果是主从同步的架构，会导致主从同步效率问题！

因此，不推荐使用blob等类型！

### 8、表中有大字段X(例如：text类型)，且字段X不会经常更新，以读为主，那么是拆成子表好？还是放一起好？

其实各有利弊，拆开带来的问题：连接消耗；不拆可能带来的问题：查询性能，所以要看你的实际情况，如果表数据量比较大，最好还是拆开为好。这样查询速度更快。

### 9、字段为什么要定义为NOT NULL?

一般情况，都会设置一个默认值，不会出现字段里面有null，又有空的情况。主要有以下几个原因：

\1. 索引性能不好，Mysql难以优化引用可空列查询，它会使索引、索引统计和值更加复杂。可空列需要更多的存储空间，还需要mysql内部进行特殊处理。可空列被索引后，每条记录都需要一个额外的字节，还能导致MYisam 中固定大小的索引变成可变大小的索引。

\2. 如果某列存在null的情况，可能导致count() 等函数执行不对的情况。

\3. sql 语句写着也麻烦，既要判断是否为空，又要判断是否为null等。

## 二、数据库查询优化

### 10、where执行顺序是怎样的？

where 条件从左往右执行的，在数据量小的时候不用考虑，但数据量多的时候要考虑条件的先后顺序，此时应遵守一个原则：排除越多的条件放在第一个。
### 11、应该在这些列上创建索引：

在经常需要搜索的列上，可以加快搜索的速度；在作为主键的列上，强制该列的唯一性和组织表中数据的排列结构；在经常用在连接的列上，这些列主要是一些外键，可以加快连接的速度；在经常需要根据范围进行搜索的列上创建索引，因为索引已经排序，其指定的范围是连续的；在经常需要排序的列上创建索引，因为索引已经排序，这样查询可以利用索引的排序，加快排序查询时间；在经常使用在WHERE子句中的列上面创建索引，加快条件的判断速度。

### 12、mysql联合索引

联合索引是两个或更多个列上的索引。对于联合索引:Mysql从左到右的使用索引中的字段，一个查询可以只使用索引中的一部分，但只能是最左侧部分。例如索引是key index (a,b,c). 可以支持a 、 a,b 、 a,b,c 3种组合进行查找，但不支持 b,c进行查找 .当最左侧字段是常量引用时，索引就十分有效。
利用索引中的附加列，您可以缩小搜索的范围，但使用一个具有两列的索引 不同于使用两个单独的索引。复合索引的结构与电话簿类似，人名由姓和名构成，电话簿首先按姓氏对进行排序，然后按名字对有相同姓氏的人进行排序。如果您知 道姓，电话簿将非常有用；如果您知道姓和名，电话簿则更为有用，但如果您只知道名不姓，电话簿将没有用处。

### 13、什么是最左前缀原则？

最左前缀原则指的是，如果查询的时候查询条件精确匹配索引的左边连续一列或几列，则此列就可以被用到。如下：

```text
select * from user where name=xx and city=xx ; ／／可以命中索引
select * from user where name=xx ; // 可以命中索引
select * from user where city=xx ; // 无法命中索引
```

这里需要注意的是，查询的时候如果两个条件都用上了，但是顺序不同，如 city= xx and name ＝xx，那么现在的查询引擎会自动优化为匹配联合索引的顺序，这样是能够命中索引的。

由于最左前缀原则，在创建联合索引时，索引字段的顺序需要考虑字段值去重之后的个数，较多的放前面。ORDER BY子句也遵循此规则。

### 14、怎么验证 MySQL 的索引是否满足需求？

使用 explain 查看 SQL 是如何执行查询语句的，从而分析你的索引是否满足需求。

explain 语法：explain select * from table where type=1。

具体来说 MySQL 中的索引，不同的数据引擎实现有所不同，但目前主流的数据库引擎的索引都是 B+ 树实现的，B+ 树的搜索效率，可以到达二分法的性能，找到数据区域之后就找到了完整的数据结构了，所有索引的性能也是更好的。

### 15、问了下MySQL数据库cpu飙升到100%的话他怎么处理？

\1. 列出所有进程 show processlist 观察所有进程 多秒没有状态变化的(干掉)

\2. 查看慢查询，找出执行时间长的sql；explain分析sql是否走索引，sql优化；

\3. 检查其他子系统是否正常，是否缓存失效引起，需要查看buffer命中率；

4.开启慢查询日志，查看慢查询的 SQL。

### 16、mysql中表锁和行锁的区别

在开发的时候，应该很少会注意到这些锁的问题，也很少会给程序加锁(除了库存这些对数量准确性要求极高的情况下)，即使我们不会这些锁知识，我们的程序在一般情况下还是可以跑得好好的。因为这些锁数据库隐式帮我们加了，只会在某些特定的场景下才需要手动加锁。

![img](https://pic1.zhimg.com/v2-6273e736ab50b5df0260f998b69fcf54_b.jpg)

对于**UPDATE、DELETE、INSERT**语句，InnoDB会自动给涉及数据集加排他锁（X) MyISAM在执行查询语句SELECT前，会自动给涉及的所有表加读锁，在执行**增、删、改**操作前，会自动给涉及的表加写锁，这个过程并不需要用户干预

Mysql有很多这种锁机制，比如行锁，表锁等，读锁，写锁等，都是在做操作之前先上锁；这些锁统称为悲观锁(Pessimistic Lock)

**行锁**

特点：锁的粒度小，发生锁冲突的概率低、处理并发的能力强；开销大、加锁慢、会出现死锁不同的存储引擎支持的锁粒度是不一样的==：InnoDB行锁和表锁都支持、MyISAM只支持表锁！InnoDB只有通过索引条件检索数据才使用行级锁==，否则，InnoDB使用表锁也就是说，InnoDB的行锁是基于索引的！

InnoDB和MyISAM有两个本质的区别：**InnoDB支持行锁、InnoDB支持事务**

- 共享锁（S锁、读锁）：允许一个事务去读一行，阻止其他事务获得相同数据集的排他锁。即多个客户可以同时读取同一个资源，但不允许其他客户修改。
- 排他锁（X锁、写锁)：允许获得排他锁的事务更新数据，阻止其他事务取得相同数据集的读锁和写锁。写锁是排他的，写锁会阻塞其他的写锁和读锁。

另外，为了允许行锁和表锁共存，实现多粒度锁机制，InnoDB还有两种内部使用的意向锁（Intention Locks），这两种意向锁都是表锁：  

- 意向共享锁（IS）：事务打算给数据行加行共享锁，事务在给一个数据行加共享锁前必须先取得该表的IS锁。
- 意向排他锁（IX）：事务打算给数据行加行排他锁，事务在给一个数据行加排他锁前必须先取得该表的IX锁。
- 意向锁也是数据库隐式帮我们做了，不需要程序员关心！

加锁的方式：自动加锁。对于UPDATE、DELETE和INSERT语句，InnoDB会自动给涉及数据集加排他锁；对于普通SELECT语句，InnoDB不会加任何锁。

**表锁**

特点：开销小、加锁快、无死锁；锁粒度大，发生锁冲突的概率高，高并发下性能低

加锁的方式：自动加锁。查询操作（SELECT），会自动给涉及的所有表加读锁，更新操作（UPDATE、DELETE、INSERT），会自动给涉及的表加写锁。

- 如果某个进程想要获取读锁，同时另外一个进程想要获取写锁。在mysql中，写锁是优先于读锁的！
- 写锁和读锁优先级的问题是可以通过参数调节的：max_write_lock_count和low-priority-updates

![img](https://pic1.zhimg.com/v2-334c965a8a5e5e100c5a6e5f63aa5c28_b.png)

**表锁下又分为两种模式：** 表读锁（Table Read Lock）&& 表写锁（Table Write Lock） 从下图可以清晰看到，在表读锁和表写锁的环境下：读读不阻塞，读写阻塞，写写阻塞！ 读读不阻塞：当前用户在读数据，其他的用户也在读数据，不会加锁 读写阻塞：当前用户在读数据，其他的用户不能修改当前用户读的数据，会加锁！ 写写阻塞：当前用户在修改数据，其他的用户不能修改当前用户正在修改的数据，会加锁！

![img](https://pic1.zhimg.com/v2-4db3d35dad829a35b9d4f94e4c21e860_b.jpg)

从上面已经看到了：读锁和写锁是互斥的，读写操作是串行。

MVCC MVCC(Multi-Version ConcurrencyControl)多版本并发控制，可以简单地认为：MVCC就是行级锁的一个变种(升级版)。在表锁中我们读写是阻塞的，基于提升并发性能的考虑，MVCC一般读写是不阻塞的(很多情况下避免了加锁的操作)。 可以简单的理解为：对数据库的任何修改的提交都不会直接覆盖之前的数据，而是产生一个新的版本与老版本共存，使得读取时可以完全不加锁。 事务的隔离级别 事务的隔离级别就是通过锁的机制来实现，**锁的应用最终导致不同事务的隔离级别**，只不过隐藏了加锁细节，事务的隔离级别有4种：  

- Read uncommitted：会出现脏读，不可重复读，幻读
- Read committed：会出现不可重复读，幻读
- Repeatable read：会出现幻读(Mysql默认的隔离级别，但是Repeatable read配合gap锁不会出现幻读！)
- Serializable：串行，避免以上的情况

![img](https://pic4.zhimg.com/v2-d7f7407e3e63862327f928245c1b68cb_b.jpg)



![img](https://pic2.zhimg.com/v2-060e82508a8cff82562db7553fd25505_b.jpg)

**悲观锁** 我们使用悲观锁的话其实很简单(手动加行锁就行了)：select * from xxxx for update，在select 语句后边加了for update相当于加了排它锁(写锁)，加了写锁以后，其他事务就不能对它修改了！需要等待当前事务修改完之后才可以修改.

**乐观锁** 乐观锁不是数据库层面上的锁，需要用户手动去加的锁。一般我们在数据库表中添加一个版本字段version来实现，在更新User表的时，执行语句如下： update A set Name=lisi,version=version+1 where ID=#{id} and version=#{version}，  此时即可避免更新丢失。

举例：下单操作包括3步骤：

1.查询出商品信息

select (status,status,version) from t_goods where id=#{id}

2.根据商品信息生成订单

3.修改商品status为2

update t_goods

set status=2,version=version+1

where id=#{id} and version=#{version};

除了自己手动实现乐观锁之外，现在网上许多框架已经封装好了乐观锁的实现，如hibernate，需要时，可能自行搜索"hiberate 乐观锁"试试看。

间隙锁GAP 当我们用范围条件检索数据而不是相等条件检索数据，并请求共享或排他锁时，InnoDB会给符合范围条件的已有数据记录的索引项加锁；对于键值在条件范围内但并不存在 的记录，叫做“间隙（GAP)”。InnoDB也会对这个“间隙”加锁，这种锁机制就是所谓的间隙锁。例子：假如emp表中只有101条记录，其empid的值分别是1,2,...,100,101 Select * from emp where empid > 100 for update;  上面是一个范围查询，InnoDB不仅会对符合条件的empid值为101的记录加锁，也会对empid大于101（这些记录并不存在）的“间隙”加锁

**InnoDB使用间隙锁的目的有2个：** 

- 为了防止幻读(上面也说了，Repeatable read隔离级别下再通过GAP锁即可避免了幻读)
- 满足恢复和复制的需要：MySQL的恢复机制要求在一个事务未提交前，其他并发事务不能插入满足其锁定条件的任何记录，也就是不允许出现幻读

死锁 并发的问题就少不了死锁，在MySQL中同样会存在死锁的问题 锁总结 表锁其实我们程序员是很少关心它的：  

- 在MyISAM存储引擎中，当执行SQL语句的时候是自动加的。
- 在InnoDB存储引擎中，如果没有使用索引，表锁也是自动加的。

现在我们大多数使用MySQL都是使用InnoDB，InnoDB支持行锁：  

- 共享锁--读锁--S锁
- 排它锁--写锁--X锁

在默认的情况下，select是不加任何行锁的~事务可以通过以下语句显示给记录集加共享锁或排他锁。  

- 共享锁（S）：SELECT * FROM table_name WHERE ... LOCK IN SHARE MODE
- 排他锁（X)：SELECT * FROM table_name WHERE ... FOR UPDATE
- InnoDB基于行锁还实现了MVCC多版本并发控制，MVCC在隔离级别下的Read committed和Repeatable read下工作。MVCC实现了读写不阻塞

事务是数据库中的一个核心概念，**指的是对数据库的一组操作作为一个整体，要么都执行要么都不执行**。
事务有四大特性：
\1. **原子性**：每个事务都是一个整体，不可再拆分，事务中的sql语句要么都执行成功，要么都执行失败。
\2. **一致性**：事务执行前后数据库的状态保持一致。比如不管如何转账，转账前后的总钱数是不变的。
\3. **隔离性**：事务和事务之间不应该相互影响,保持隔离。
\4. **持久性**：事务一旦提交对数据库的修改就是永久的，即使电脑发生故障也不会影响该修改，因为他的结果是记录在存储设备上的。
事务中有一个重要的特性“事务的隔离性”指的是事务和事务之间不应该相互影响,保持隔离，然而在现实中多个事务可能会操作同一个数据，造成并发问题：

\1. **脏读**：一个事务读取到了另一个事务尚未提交的数据。
\2. **不可重复读**：事务一读取到了age的值20，事务二将该值修改成了28，事务一再次读取age的值28，事务一两次读取的age值不一致。
\3. **幻读**：事务一读取到A表中有一条记录，事务二往A表中插入一条记录，事务一再次读取的时候记录变成了两条，就像发生幻觉一样。
不可重复读和幻读很相似，可以从两个角度理解两者的差别：
\1. 不可重复读是另一个事务修改了数据，导致该事务多次读取出来的值不一样，而幻读是另一个事务插入或删除了记录，导致该事务多次读取出来的记录数不一样
\2. 不可重复读的解决只需要锁住会发生修改的记录就可以，幻读需要锁住更大的范围。
正是因为有这些问题存在，数据库设置了隔离级别来处理：
\1. **读未提交**（read uncommitted）: 事务中的修改，即使没有提交，其他事务也可以看得到
在这种隔离级别下有可能发生脏读，不可重复读和幻读。
一家酒店对外预定房间，现在还剩四间房，一个顾客到小王这里来预定四间房，小王查询系统发现还剩四间就将这四间房预定出去，该事务还没提交的时候另一个顾客到小李这里来预定房间，小李查询系统发现没房了，就拒绝了这个订单，此时小王的电脑发生故障，事务回滚，订单失效，这就是脏读造成的影响。
\2. **读已提交**（read committed）: 事务中的修改只有提交以后才能被其它事务看到。在这种隔离级别下有可能发生不可重复读和幻读。
还是定房间的例子，一个顾客到小王这里来预定四间房，小王将这四间房预定了出去，该事务还没提交的时候另一个顾客到小李这里来预定房间，小李查询系统发现还有四间房，刚想预定的时候小王的事务提交了，小李的系统立马呈现0间房。这就是不可重复读造成的影响。
\3. **可重复读** （repeatable read）：该级别保证了在事务中看到的每行的记录的结果是一致的，但是这种级别下有可能发生幻读。默认是可重复读
公司规定如果销售额达不到就要扣工资，经理查询小王的销售业绩，发现还差几间房，经理喜上眉梢，把结果打印出来，结果打印出来的结果业绩正好合格，原来小王在这当口又卖了几张票正好填上了这个空缺。这就是幻读造成的影响。
**串行化**（serializable）：该级别下所有的事务都是串行执行的，一个事务执行完了才能执行其它的事务，可以解决所有的并发问题，它是靠大量加锁实现的，所以效率很低下。只有在需要绝对保证数据一致性，并且并发量不大的情况下，可以考虑。

### 17、mysql主键索引和普通索引之间的区别是什么

1）普通索引（INDEX）； 2）唯一索引（UNIQUE INDEX）； 3）全文索引（FULLTEXT）（全文索引是MyISAM的一个特殊索引类型，主要用于全文检索）； 4）主键索引； 5）组合索引（最左前缀）。

**普通索引**

普通索引是最基本的索引类型，而且它没有唯一性之类的限制。普通索引可以通过以下几种方式创建：

创建索引，例如

**CREATEINDEX**<索引的名字>**ON**tablename (列的列表);

修改表，例如

**ALTERTABLE**tablename**ADDINDEX**[索引的名字] (列的列表);

创建表的时候指定索引，例如

**CREATETABLE**tablename ( [...],**INDEX**[索引的名字] (列的列表) );

**主键索引**

主键是一种唯一性索引，但它必须指定为“PRIMARY KEY”。

主键一般在创建表的时候指定，例如

**CREATETABLE**tablename ( [...],**PRIMARYKEY**(列的列表) );

但是，我们也可以通过修改表的方式加入主键，例如“ALTER TABLE tablename ADD PRIMARY KEY (列的列表); ”。每个表只能有一个主键。

**区别**

普通索引是最基本的索引类型，没有任何限制，值可以为空，仅加速查询。普通索引是可以重复的，一个表中可以有多个普通索引。

主键索引是一种特殊的唯一索引，一个表只能有一个主键，不允许有空值；索引列的所有值都只能出现一次，即必须唯一。简单来说：主键索引是加速查询 + 列值唯一（不可以有null）+ 表中只有一个。
2.唯一索引和主键索引区别
答：1）主键为一种约束，唯一索引为一种索引，本质上就不同；
2）主键创建后一定包含唯一性索引，而唯一索引不一定就是主键；
3）主键不允许空值，唯一索引可以为空；
4）主键可以被其他表引用，而唯一索引不可以；
5）主键只允许一个，唯一索引允许多个；
6）主键和索引都是键，主键是逻辑键，索引为物理键，即主键不实际存在。

3.索引失效
答：1）最佳左前缀原则（组合索引，不按索引定义时制定的顺序，最左优先）；
2）like模糊查询时，以%开头，导致索引失效;
3）使用“!=”和“<>”都会使索引失效，如果是主键或者索引列是整数，索引不会失效；
4）遇到null值，索引失效；
5）索引列上的显式或者隐式运算，导致索引失效；
6）如果列类型是字符串，那一定要在条件中将数据使用引号引用起来,否则不使用索引（如select * from USER where name=123;
7）用or连接导致索引失效（or条件有未建立索引的列导致索引失效）。

4.索引的坏处
答：1）创建索引和维护索引要耗费时间，这种时间随着数据量的增加而增加；
2）索引需要占物理空间，除了数据表占数据空间之外，每一个索引还要占一定的物理空间。如果要建立聚簇索引，那么需要的空间就会更大；
3）当对表中的数据进行增加、删除和修改的时候，索引也要动态的维护，这样就降低了数据的维护速度。
因此索引也会有它的缺点：虽然索引大大提高了查询速度，同时却会降低更新表的速度，如对表进行INSERT、UPDATE和DELETE。

5.索引的方法
答：主要有以下几种索引方法：B-Tree，Hash，R-Tree。
1）B-Tree：
B-Tree是最常见的索引类型，所有值（被索引的列）都是排过序的，每个叶节点到跟节点距离相等。所以B-Tree适合用来查找某一范围内的数据，而且可以直接支持数据排序（ORDER BY）B-Tree在MyISAM里的形式和Innodb稍有不同：
MyISAM表数据文件和索引文件是分离的，索引文件仅保存数据记录的磁盘地址，
InnoDB表数据文件本身就是主索引，叶节点data域保存了完整的数据记录。
2）Hash索引：
a.仅支持"=",“IN"和”<=>"精确查询，不能使用范围查询：
由于Hash索引比较的是进行Hash运算之后的Hash值，所以它只能用于等值的过滤，不能用于基于范围的过滤，因为经过相应的Hash算法处理之后的Hash；
b.不支持排序：
由于Hash索引中存放的是经过Hash计算之后的Hash值，而且Hash值的大小关系并不一定和Hash运算前的键值完全一样，所以数据库无法利用索引的数据来避免任何排序运算；
c.在任何时候都不能避免表扫描：
由于Hash索引比较的是进行Hash运算之后的Hash值，所以即使取满足某个Hash键值的数据的记录条数，也无法从Hash索引中直接完成查询，还是要通过访问表中的实际数据进行相应的比较，并得到相应的结果；
d.检索效率高，索引的检索可以一次定位，不像B-Tree索引需要从根节点到枝节点，最后才能访问到页节点这样多次的IO访问，所以Hash索引的查询效率要远高于B-Tree索引；
e.只有Memory引擎支持显式的Hash索引，但是它的Hash是nonunique的，冲突太多时也会影响查找性能。Memory引擎默认的索引类型即是Hash索引，虽然它也支持B-Tree索引。
3）R-Tree索引：
R-Tree在MySQL很少使用，仅支持geometry数据类型，支持该类型的存储引擎只有MyISAM、BDb、InnoDb、NDb、Archive几种。

### 18、SQL JOIN 中 on 与 where 的区别

- **left join** : 左连接，返回左表中所有的记录以及右表中连接字段相等的记录。
- **right join** : 右连接，返回右表中所有的记录以及左表中连接字段相等的记录。
- **inner join** : 内连接，又叫等值连接，只返回两个表中连接字段相等的行。
- **full join** : 外连接，返回两个表中的行：left join + right join。
- **cross join** : 结果是笛卡尔积，就是第一个表的行数乘以第二个表的行数。

**关键字 on**
数据库在通过连接两张或多张表来返回记录时，都会生成一张中间的临时表，然后再将这张临时表返回给用户。
在使用 **left jion** 时，**on** 和 **where** 条件的区别如下：

- 1、 **on** 条件是在生成临时表时使用的条件，它不管 **on** 中的条件是否为真，都会返回左边表中的记录。
- 2、**where** 条件是在临时表生成好后，再对临时表进行过滤的条件。这时已经没有 **left join** 的含义（必须返回左边表的记录）了，条件不为真的就全部过滤掉。

假设有两张表：
**表1：tab2**
idsize110220330
**表2：tab2**
sizename10AAA20BBB20CCC
两条 SQL:
select * form tab1 left join tab2 on (tab1.size = tab2.size) where tab2.name='AAA'
select * form tab1 left join tab2 on (tab1.size = tab2.size and tab2.name='AAA')

第一条SQL的过程：
1、中间表
on条件:
tab1.size = tab2.sizetab1.idtab1.sizetab2.sizetab2.name11010AAA22020BBB22020CCC330(null)(null)
2、再对中间表过滤
where 条件：
tab2.name='AAA'tab1.idtab1.sizetab2.sizetab2.name11010AAA
第二条SQL的过程：
1、中间表
on条件:
tab1.size = tab2.size and tab2.name='AAA'
(条件不为真也会返回左表中的录)

tab1.idtab1.sizetab2.sizetab2.name11010AAA220(null)(null)330(null)(null)
其实以上结果的关键原因就是 **left join、right join、full join** 的特殊性，不管 **on** 上的条件是否为真都会返回 **left** 或 **right** 表中的记录，**full** 则具有 **left** 和 **right** 的特性的并集。 而 **inner jion**没这个特殊性，则条件放在 **on** 中和 **where** 中，返回的结果集是相同的。

**1、一张表，里面有ID自增主键，当insert了17条记录之后，删除了第15,16,17条记录，再把Mysql重启，再insert一条记录，这条记录的ID是18还是15 ？**

如果表的类型是myisam，那么是18。

因为myisam表会把自增主键的最大ID记录到数据文件里，重启mysql自增主键的最大ID也不会丢失。

如果表的类型是innoDB，那么是15.

innoDB表只是把自增主键的最大ID记录到内存中，所以重启数据库或者是对表进行OPTIMIZE操作，都会导致最大ID丢失

### 19、优化 MYSQL 数据库的方法

**1、对查询进行优化,应尽可能避免全表扫描**

**1）减少where 字段值null判断**

**2）应尽量避免在 where 子句中使用!=或<>操作符**

**3）应尽量避免在 where 子句中使用 or 来连接条件**

**4）in 和 not in 也要慎用**

**5）少使用模糊匹配 like**

**6）应尽量避免在 where 子句中对字段进行表达式操作**

**7）任何地方都不要使用\*通配符去查询所有**

**3、不要在条件判断时进行 算数运算**

**4、很多时候用 exists 代替 in 是一个好的选择**

(1) 选取最适用的字段属性，尽可能减少定义字段长度，尽量把字段设置 NOT NULL, 例如’省份，性别’, 最好设置为 ENUM

二、SQL语句中IN包含的值不应过多

三、SELECT语句务必指明字段名称

SELECT *增加很多不必要的消耗（cpu、io、内存、网络带宽）；增加了使用覆盖索引的可能性；当表结构发生改变时，前断也需要更新。所以要求直接在select后面接上字段名。

四、当只需要一条数据的时候，使用limit 1

这是为了使EXPLAIN中type列达到const类型

五、如果排序字段没有用到索引，就尽量少排序

六、如果限制条件中其他字段没有索引，尽量少用or

七、尽量用union all代替union

九、区分in和exists， not in和not exists

十、使用合理的分页方式以提高分页的效率

十一、分段查询

十二、避免在 where 子句中对字段进行 null 值判断

十三、不建议使用%前缀模糊查询

例如LIKE “%name”或者LIKE “%name%”，这种查询会导致索引失效而进行全表扫描。但是可以使用LIKE “name%”。

十四、避免在where子句中对字段进行表达式操作

十六、对于联合索引来说，要遵守最左前缀法则

十九、关于JOIN优化

### 20、适用MySQL 5.0以上版本:

1.一个汉字占多少长度与编码有关: UTF-8:一个汉字=3个字节 GBK:一个汉字=2个字节

### 21、什么时候适合创建索引

1、适合创建索引条件

　　1.、主键自动建立唯一索引

　　2、频繁作为查询条件的字段应该建立索引

　　3、查询中与其他表关联的字段，外键关系建立索引

　　4、单键/组合索引的选择问题，组合索引性价比更高

　　5、查询中排序的字段，排序字段若通过索引去访问将大大提高排序效率

　　6、查询中统计或者分组字段

2、不适合创建索引条件

　　1、表记录少的

　　2、经常增删改的表或者字段

　　3、where条件里用不到的字段不创建索引

　　4、过滤性不好的不适合建索引

9、在Mysql中ENUM的用法是什么？

ENUM是一个字符串对象，用于指定一组预定义的值，并可在创建表时使用。

Create table size(name ENUM('Smail,'Medium','Large');

### 22、常见的sql语句

1、表名order中有 1 2 3 4 1 

去掉重复值 sql : select distinct from order

结果为company 1 2 3 4 

2、asc 是升序 是从小到大 desc 是大到小 group 是分组

3、IFNULL() 函数用于判断第一个表达式是否为 NULL，如果为 NULL 则返回第二个参数的值，如果不为 NULL 则返回第一个参数的值。

6、查找学生 查询姓“赵”的用户 select * from table where name like '赵%'

查询姓名中最后一个字段带赵字  select * from table where name like '%赵'

查询姓名中带有赵的字段  select * from table where name like '%赵%'

7、汇总分析  查询一个学生总分 select sum(*) from table where 课程号='0002' 

查询选课程的学生人数  select count(distinct 学号) as 学生人数 from table

8、分组 查询各科成绩最高和最低的分 select 课程号 max（成绩）as 最高分，min（成绩）as 最低分 from table group by 课程号

查询每门课程被选修的学生数  select 课程号，count(学号) from score group by 课程号

查询男生 和女生人数 select 性别，count(*) from tabel group by 性别

9、分组结果的条件 查询平均成绩大于60分的学号和平均成绩 

先说下Having是一个过滤声明，是在查询返回结果集以后对查询结果进行的过滤操作，在Having中可以使用聚合函数。

在查询过程中聚合语句(sum,min,max,avg,count)要比having子句优先执行。而where子句在查询过程中执行优先级高于聚合语句。

select 学号，avg(成绩)from group by 学号 having avg(成绩) > 60 

查询至少选修俩门课程的学生学号 select 学号,count(课程号)as 选修课程数目 from table group by 学号 having count(课程号)>=2;

查询同名同性学生名单并统计人数 select 姓名，count(**)as 人数from table group by  姓名 having count(***)>=2;

查询不及格的课程并按课程号从大到小排序

select 课程号 from table where 成绩<60 order by 课程号 desc;

10、类似于成绩这一类型表类型为float

### 23、php读取文件内容的几种方法和函数？

打开文件，然后读取。Fopen() fread()

打开读取一次完成 file_get_contents()

### 24、数据库开启慢查询

show variables like 'slow_query%';

![img](https://pic1.zhimg.com/v2-2d69d7233369c2dc263d2e0c79393c08_b.jpg)

show variables like 'long_query_time';

![img](https://pic1.zhimg.com/v2-b86bce59c65d72c7ad95061c2cfe034c_b.jpg)

方法一：全局变量设置 可以在navicat设置方便 有得时候修改时间不变还是10那就采用第二种

将 slow_query_log 全局变量设置为“ON”状态

set global slow_query_log='ON'; 

设置慢查询日志存放的位置

set global slow_query_log_file='/usr/local/mysql/data/www-slow.log';

查询超过10秒就记录

set global long_query_time=10;

方法二：配置文件设置

修改配置文件my.cnf，在[mysqld]下的下方加入

```text
[mysqld]
slow_query_log = ON
slow_query_log_file = /usr/local/mysql/data/www-slow.log
long_query_time = 1
```

3.重启MySQL服务

```text
service mysqld restart
```

4.查看设置后的参数

```text
mysql> show variables like 'slow_query%';
+---------------------+--------------------------------+
| Variable_name       | Value                          |
+---------------------+--------------------------------+
| slow_query_log      | ON                             |
| slow_query_log_file | /usr/local/mysql/data/slow.log |
+---------------------+--------------------------------+

mysql> show variables like 'long_query_time';
+-----------------+----------+
| Variable_name   | Value    |
+-----------------+----------+
| long_query_time | 1.000000 |
+-----------------+----------+
```

### 25、MYSQL的主从延迟怎么解决。

实际上主从同步延迟根本没有什么一招制敌的办法，因为所有的 SQL 必须都要在从服务器里面执行一遍，但是主服务器如果不断的有更新操作源源不断的写入， 那么一旦有延迟产生，那么延迟加重的可能性就会越来越大。 当然我们可以做一些缓解的措施。

a）、最简单的减少 slave 同步延时的方案就是在架构上做优化，尽量让主库的 DDL 快速执行。还有就是主库是写，对数据安全性较高，比如 sync_binlog=1，innodb_flush_log_at_trx_commit = 1 之类的设置，而 slave 则不需要这么高的数据安全，完全可以将 sync_binlog 设置为 0 或者关闭 binlog，innodb_flushlog 也可以设置为 0 来提高 sql 的执行效率。另外就是使用比主库更好的硬件设备作为 slave。

b）、把一台从服务器当作备份使用， 而不提供查询， 这样他的负载就下来了， 执行 relay log 里面的 SQL 效率自然就高了。

c）、增加从服务器，这个目的还是分散读的压力， 从而降低服务器负载。

### 26、mysql中in 和exists 区别。

mysql 中的 in 语句是把外表和内表作 hash 连接，而 exists 语句是对外表作 loop 循环，每次 loop 循环再对内表进行查询。一直大家都认为 exists 比 in 语句的效率要高，这种说法其实是不准确的。这个是要区分环境的。

㊤、如果查询的两个表大小相当，那么用 in 和 exists 差别不大。

㊥、如果两个表中一个较小，一个是大表，则子查询表大的用 exists，子查询表小的用 in。

㊦、not in 和 not exists 如果查询语句使用了not in 那么内外表都进行全表扫描，没有用到索引；而 not extsts 的子查询依然能用到表上的索引。所以无论那个表大，用 not exists 都比 not in 要快。

EXISTS 只返回 TRUE 或 FALSE，不会返回 UNKNOWN IN 当遇到包含NULL的情况，那么就会返回 UNKNOWN

### 27、mysql数据库监控指标之吞吐量

如果你的数据库运行缓慢，或者出于某种原因无法响应查询，技术栈中每个依赖数据库的组件都会遭受性能问题。为了保证数据库的平稳运行，你可以监控下吞吐量这个指标。

吞吐量

在监控任何系统时，我们最关心的应该是确保系统能够高效地完成工作。数据库的工作是运行查询，因此首要任务是确保 MySQL 能够如期执行查询。

MySQL 有一个名为 Questions 的内部计数器（根据 MySQL 用语，这是一个服务器状态变量），客户端每发送一个查询语句，其值就会加一。

### 28、MySQL中通过EXPLAIN如何分析SQL的执行计划详解

![img](https://pic1.zhimg.com/v2-fa044dd3c8a511580a888a90dac751cc_b.jpg)

简单说下有用的字段

**table**: sql所查询的表名

**possible_keys**: sql可能用到的索引

**key**: sql实际用到的索引,如果是Null,说明没有用到索引

**ows**: MySQL认为执行查询时必须检查的行数

下面分别对EXPLAIN命令结果的每一列进行说明：

**.select_type:**表示SELECT的类型，常见的取值有：

类型说明

SIMPLE简单表，不使用表连接或子查询

PRIMARY主查询，即外层的查询

UNIONUNION中的第二个或者后面的查询语句

SUBQUERY子查询中的第一个

**.table:**输出结果集的表（表别名）

**.type:**表示MySQL在表中找到所需行的方式，或者叫访问类型。常见访问类型如下，从上到下，性能由差到最好：

| ALL          | 全表扫描             |
| ------------ | -------------------- |
| index        | 索引全扫描           |
| range        | 索引范围扫描         |
| ref          | 非唯一索引扫描       |
| eq_ref       | 唯一索引扫描         |
| const,system | 单表最多有一个匹配行 |
| NULL         | 不用扫描表或索引     |
|              |                      |

type结果值从好到坏依次是：

system > const > eq_ref > ref > fulltext > ref_or_null > index_merge > unique_subquery > index_subquery > range > index > ALL

一般来说，得保证查询至少达到range级别，最好能达到ref，否则就可能会出现性能问题

一般来说，得保证查询至少达到range级别，最好能达到ref，否则就可能会出现性能问题。

**1、type=ALL，全表扫描，MySQL遍历全表来找到匹配行**

一般是没有where条件或者where条件没有使用索引的查询语句

```text
EXPLAIN SELECT * FROM customer WHERE active=0;
```

![img](https://pic3.zhimg.com/v2-bff660f59d12171048251740021d73e2_b.png)

**2、type=index，索引全扫描，MySQL遍历整个索引来查询匹配行，并不会扫描表**

一般是查询的字段都有索引的查询语句

```
EXPLAIN SELECT store_id FROM customer;
```

![img](https://pic1.zhimg.com/v2-07e410c4dcf2c13020145255b4646468_b.png)

**3、type=range，索引范围扫描，常用于<、<=、>、>=、between等操作**

```
EXPLAIN SELECT* FROM customer WHEREcustomer_id>=10 ANDcustomer_id<=20;
```

![img](https://pic1.zhimg.com/v2-d4ecd00d262eed0fdde2f319b1a1b5a4_b.png)

**注意**这种情况下比较的字段是需要加索引的，如果没有索引，则MySQL会进行全表扫描，如下面这种情况，create_date字段没有加索引：

EXPLAIN SELECT * FROM customer WHERE create_date>='2006-02-13' ;

![img](https://pic2.zhimg.com/v2-0038deb6db359392311eb0fd5ce2a9ad_b.png)

**4、type=ref，使用非唯一索引或唯一索引的前缀扫描，返回匹配某个单独值的记录行**

`store_id`字段存在普通索引（非唯一索引）

```
EXPLAIN SELECT* FROMcustomer WHEREstore_id=10;
```

![img](https://pic4.zhimg.com/v2-28704c3ccb414d8fda75940abb5b753f_b.png)

ref类型还经常会出现在join操作中：

customer、payment表关联查询，关联字段`customer.customer_id`（主键），`payment.customer_id`（非唯一索引）。表关联查询时必定会有一张表进行全表扫描，此表一定是几张表中记录行数最少的表，然后再通过非唯一索引寻找其他关联表中的匹配行，以此达到表关联时扫描行数最少。

![img](https://pic2.zhimg.com/v2-21e872d4ee49067d02d47133760ab2ad_b.jpg)

因为customer、payment两表中customer表的记录行数最少，所以customer表进行全表扫描，payment表通过非唯一索引寻找匹配行。

```text
EXPLAIN SELECT * FROM customer customer INNER JOIN payment payment ON customer.customer_id = payment.customer_id;
```

![img](https://pic2.zhimg.com/v2-87a82a6070843ce025ead199d9b03f39_b.png)

**6、type=const/system，单表中最多有一条匹配行，查询起来非常迅速，所以这个匹配行的其他列的值可以被优化器在当前查询中当作常量来处理**

const/system出现在根据主键primary key或者 唯一索引 unique index 进行的查询

根据主键primary key进行的查询：

```
EXPLAIN SELECT* FROMcustomer WHEREcustomer_id =10;
```

![img](https://pic2.zhimg.com/v2-33782e55c986dc5b6bca0e89cde49531_b.png)

根据唯一索引unique index进行的查询：

```text
EXPLAIN SELECT * FROM customer WHERE email ='MARY.SMITH@sakilacustomer.org';
```

![img](https://pic4.zhimg.com/v2-71e65de46caf22a78c7bc5cae2ce6e6f_b.jpg)

**7、type=NULL，MySQL不用访问表或者索引，直接就能够得到结果**

![img](https://pic3.zhimg.com/v2-9a358a3c7b052adee7d2e84d21187f56_b.png)

**.possible_keys:** 表示查询可能使用的索引

**.key:** 实际使用的索引

**.key_len:** 使用索引字段的长度

**.ref:** 使用哪个列或常数与key一起从表中选择行。

**.rows:** 扫描行的数量

**.filtered:** 存储引擎返回的数据在server层过滤后，剩下多少满足查询的记录数量的比例(百分比)

**.Extra:** 执行情况的说明和描述，包含不适合在其他列中显示但是对执行计划非常重要的额外信息

最主要的有一下三种：

| Using Index           | 表示索引覆盖，不会回表查询                            |
| --------------------- | ----------------------------------------------------- |
| Using Where           | 表示进行了回表查询                                    |
| Using Index Condition | 表示进行了ICP优化                                     |
| Using Flesort         | 表示MySQL需额外排序操作, 不能通过索引顺序达到排序效果 |

**什么是ICP？**

MySQL5.6引入了Index Condition Pushdown（ICP）的特性，进一步优化了查询。Pushdown表示操作下放，某些情况下的条件过滤操作下放到存储引擎。

```
EXPLAIN SELECT* FROMrental WHERErental_date='2005-05-25'ANDcustomer_id>=300 ANDcustomer_id<=400;
```

**在5.6版本之前：**

优化器首先使用复合索引idx_rental_date过滤出符合条件`rental_date='2005-05-25'`的记录，然后根据复合索引idx_rental_date回表获取记录，最终根据条件`customer_id>=300 AND customer_id<=400`过滤出最后的查询结果（在服务层完成）。

**在5.6版本之后：**

MySQL使用了ICP来进一步优化查询，在检索的时候，把条件`customer_id>=300 AND customer_id<=400`也推到存储引擎层完成过滤，这样能够降低不必要的IO访问。Extra为`Using index condition`就表示使用了ICP优化。

![img](https://pic4.zhimg.com/v2-39809c410afe4b1b6432b33b17d48693_b.png)

查看当前表中哪些字段创建索引了

show index from table;

### 29、Hash索引和B+树区别是什么？你在设计索引是怎么抉择的？

- B+树可以进行范围查询，Hash索引不能。
- B+树支持联合索引的最左侧原则，Hash索引不支持。
- B+树支持order by排序，Hash索引不支持。
- Hash索引在等值查询上比B+树效率更高。
- B+树使用like 进行模糊查询的时候，like后面（比如%开头）的话可以起到优化的作用，Hash索引根本无法进行模糊查询。

**1、查询Questions服务器状态变量值**

SHOW GLOBAL STATUS LIKE "Questions";

![img](https://pic2.zhimg.com/v2-abc5c25500a73093ea8e53ce06753c05_b.jpg)

**2、监控读指令的分解情况**

SHOW GLOBAL STATUS LIKE "Com_select";

![img](https://pic4.zhimg.com/v2-4042e8f31e0de892d072420f9741ad0f_b.jpg)

**3、监控写指令的分解情况**

Writes = Com_insert + Com_update + Com_delete；

show GLOBAL status like "com_insert";

SHOW GLOBAL STATUS LIKE "com_update";

SHOW GLOBAL STATUS LIKE "com_delete";

![img](https://pic4.zhimg.com/v2-9bd2eea6e3a12dfdcb64d38a354bff1b_b.jpg)

### 30、MyISAM 和 InnoDB 的基本区别？索引结构如何实现？

MyISAM类型不支持事务，表锁，易产生碎片，要经常优化，读写速度较快，而InnoDB类型支持事务，行锁，有崩溃恢复能力。读写速度比MyISAM慢，适合插入和更新操作比较多的应用，占空间大。

创建索引：alert table tablename add index (`字段名`)

### 31、sql语句应该考虑哪些安全性

（1）防止sql注入，对特殊字符进行转义，过滤或者使用预编译sql语句绑定

（2）使用最小权限原则，特别是不要使用root账户，微不同的动作或者操作建立不同的账户

（3）当sql出错时，不要把数据库出错的信息暴露到客户端

### 32、mysql_fetch_array和mysql_fetch_object的区别是什么？

以下是mysql_fetch_array和mysql_fetch_object的区别：

mysql_fetch_array（） - 将结果行作为关联数组或来自数据库的常规数组返回。

mysql_fetch_object - 从数据库返回结果行作为对象。

### 33、mysql_fetch_row() 和mysql_fetch_array之间有什么区别?

Mysql_fetch_row()是从结果集中取出一行作为枚举数组，mysql_fetch_array()是从结果集中取出一行作为索引数组或关联数组或两种方式都有。

实现中文字串截取无乱码的方法

Mb_substr();

### 34、请写出数据类型(int char varchar datetime text)的意思；请问 varchar 和 char有什么区别？

Int 整数  Datetime 日期时间型 Text 文本型

- char(n) ：固定长度类型，比如订阅 char(10)，当你输入"abc"三个字符的时候，它们占的空间还是 10 个字节，其他 7 个是空字节。

chat 优点：效率高；缺点：占用空间；适用场景：存储密码的 md5 值，固定长度的，使用 char 非常合适。

- varchar(n) ：可变长度，存储的值是每个值占用的字节再加上一个用来记录其长度的字节的长度。

所以，从空间上考虑 varcahr 比较合适；从效率上考虑 char 比较合适，二者使用需要权衡。