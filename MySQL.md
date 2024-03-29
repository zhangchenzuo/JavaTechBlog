- [数据库范式](#数据库范式)
- [Mysql](#mysql)
  - [Mysql架构](#mysql架构)
    - [Q: 优化器的具体策略？](#q-优化器的具体策略)
  - [语句执行顺序](#语句执行顺序)
- [索引](#索引)
  - [为什么是B+树](#为什么是b树)
    - [B 树和 B+树有什么不同呢？](#b-树和-b树有什么不同呢)
  - [聚集索引和辅助索引（非聚集索引）](#聚集索引和辅助索引非聚集索引)
  - [联合索引、索引覆盖、最左前缀原则、索引下推](#联合索引索引覆盖最左前缀原则索引下推)
    - [数据库的索引存在磁盘还是内存？](#数据库的索引存在磁盘还是内存)
    - [为什么使用自增id作为主键，而不是UUID](#为什么使用自增id作为主键而不是uuid)
    - [添加索引](#添加索引)
  - [如何设计数据库的索引](#如何设计数据库的索引)
    - [索引失效的场景](#索引失效的场景)
- [锁](#锁)
  - [锁的类型](#锁的类型)
  - [Innodb中的行锁](#innodb中的行锁)
  - [一致性非锁定读和一致性锁定读（当前读/对照读）](#一致性非锁定读和一致性锁定读当前读对照读)
    - [一致性非锁定读](#一致性非锁定读)
    - [一致性锁定读](#一致性锁定读)
  - [死锁--锁的性能问题](#死锁--锁的性能问题)
- [事务](#事务)
  - [事务的特点](#事务的特点)
  - [隔离级别](#隔离级别)
  - [隔离问题](#隔离问题)
  - [MVCC原理](#mvcc原理)
  - [幻读](#幻读)
    - [什么是幻读？幻读与不可重复读有什么区别？](#什么是幻读幻读与不可重复读有什么区别)
    - [什么是间隙锁？](#什么是间隙锁)
    - [可重复读隔离下的间隙锁加锁条件](#可重复读隔离下的间隙锁加锁条件)
- [日志系统](#日志系统)
  - [redo log（重做日志）](#redo-log重做日志)
    - [redo log与WAL](#redo-log与wal)
    - [WAL技术](#wal技术)
  - [binlog（归档日志）](#binlog归档日志)
    - [binlog和redo log不同？](#binlog和redo-log不同)
  - [undo log(重做日志)](#undo-log重做日志)
    - [什么时候可以删除undo log？](#什么时候可以删除undo-log)
  - [更新流程中的两阶段提交](#更新流程中的两阶段提交)
    - [为什么需要两阶段提交呢?](#为什么需要两阶段提交呢)
    - [完整的落盘流程图](#完整的落盘流程图)
  - [了解组提交group commit么？](#了解组提交group-commit么)
- [MySQL的主从复制](#mysql的主从复制)
  - [保证主从一致性](#保证主从一致性)
- [性能优化](#性能优化)
  - [如何查看命中索引](#如何查看命中索引)
  - [解决慢查询问题](#解决慢查询问题)
  - [sql语句优化器的操作](#sql语句优化器的操作)
    - [Mysql优化的一些思路](#mysql优化的一些思路)
    - [走索引一定比不走索引的查询效率高么？](#走索引一定比不走索引的查询效率高么)
  - [降低数据库的压力](#降低数据库的压力)
  - [Mysql的分页查询的优化](#mysql的分页查询的优化)
  - [如何科学的进行水平分表](#如何科学的进行水平分表)
  - [如何进行跨库分页的几种常见方案](#如何进行跨库分页的几种常见方案)
- [Innodb引擎优化](#innodb引擎优化)
  - [体系架构](#体系架构)
    - [后台线程](#后台线程)
    - [内存池](#内存池)
  - [概念](#概念)
    - [什么是checkpoint](#什么是checkpoint)
      - [何时触发checkpoint？](#何时触发checkpoint)
    - [什么是LSN（log sequence number）](#什么是lsnlog-sequence-number)
    - [什么是insert buffer？](#什么是insert-buffer)
      - [insert buffer的原理](#insert-buffer的原理)
    - [自适应哈希](#自适应哈希)
    - [刷新邻接页 Flush Neighbor Page](#刷新邻接页-flush-neighbor-page)
  - [Innodb和MyISAM对比](#innodb和myisam对比)
  - [为什么推荐使用自增ID](#为什么推荐使用自增id)
- [sql语句](#sql语句)
  - [join和union有什么区别](#join和union有什么区别)
  - [count(\*)与count(1)](#count与count1)
- [其他](#其他)
  - [单机MySQL](#单机mysql)
  - [缓存（读写分离）+MySQL](#缓存读写分离mysql)
  - [垂直拆分+水平拆分+集群](#垂直拆分水平拆分集群)
  - [数据类型丰富，改用非关系型数据库](#数据类型丰富改用非关系型数据库)
  - [mybatis中#和$的区别是什么](#mybatis中和的区别是什么)
  - [如何处理海量数据中的topK](#如何处理海量数据中的topk)


> MySQL与Redis复习
# 数据库范式
第一范式（1NF）:列不可再分。确保不冗余。比如地址就写`省份 市 具体地址`，而不是很粗略。

第二范式（2NF）属性完全依赖于主键 不能一行依赖多个列的属性。 比如 主键是学生序号，列就不可以出现课程学分这种依赖于课程的属性。

第三范式（3NF）属性不依赖于其它非主属性 属性直接依赖于主键（学号，姓名，年龄，性别，所在院校）--（所在院校，院校地址，院校电话）
# Mysql
## Mysql架构
![mysql 架构](https://img-blog.csdnimg.cn/20210126233908827.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pjejU1NjY3MTk=,size_16,color_FFFFFF,t_70)
MySQL可以分为**Server层**和**存储引擎层**两部分。
* Server层包括**连接器、查询缓存、分析器、优化器、执行器**等，涵盖MySQL的大多数核心服务
功能，以及所有的内置函数（如日期、时间、数学和加密函数等），所有跨存储引擎的功能都在
这一层实现，比如存储过程、触发器、视图等。
* 存储引擎层负责数据的存储和提取。其架构模式是插件式的，支持InnoDB、MyISAM、
Memory等多个存储引擎。现在最常用的存储引擎是InnoDB，它从MySQL 5.5.5版本开始成为了
默认存储引擎。

* 连接器：主要是连接客户端，在每次登陆的时候更新你的登录权限
* 查询缓存：从缓存中查找，k-v对都是保存在连接器中。不建议使用这个方法，因为命中率很低。在8.0版本中直接删除了查询缓存
* 分析器：分析sql语句的语法是否正确，是否表存在对应的内容。
* 优化器：优化器是在表里面有多个索引的时候，决定使用哪个索引；或者在一个语句有多表关联（join）的时候，决定各个表的连接顺序
* 执行器：首先判断有无权限打开，然后调用引擎提供的接口去执行查询。


### Q: 优化器的具体策略？

优化器会在内部创建一个解析树，然后尝试各种优化，包括重写查询，决定表的读写顺序，选择合适的索引。可以通过explain进行查看。
## 语句执行顺序


sql语句可选的内容包括`<SELECT clause> [<FROM clause>] [<WHERE clause>] [<GROUP BY clause>] [<HAVING clause>] [<ORDER BY clause>] [<LIMIT clause>] `
```sql
select 考生姓名, max(总成绩) as max总成绩 
from tb_Grade 
where 考生姓名 is not null 
group by 考生姓名 
having max(总成绩) > 600 
order by max总成绩 
```

>执行顺序

开始->FROM子句->WHERE子句->GROUP BY子句->HAVING子句->ORDER BY子句->SELECT子句->LIMIT子句->最终结果 
# 索引
[参考文献](https://mp.weixin.qq.com/s/qHJiTjpvDikFcdl9SRL97Q)

## 为什么是B+树
* **哈希表**，直接查找快速，但是**不适用于范围查找**。
* **BST二叉搜索树**，*不适用于自增主键，会退化成链表*。
* **红黑树**是可以自动平衡的，但是对于自增问题还是无法保证右倾的趋势。效率不是很高
* **AVL树**的平衡是更为严格的，但是由于AVL和红黑树都是二叉的，效率也不太高。我们希望可以**多叉树**更好。**为了让一个查询尽量少地读磁盘，就必须让查询过程访问尽量少的数据块**

![在这里插入图片描述](https://img-blog.csdnimg.cn/img_convert/0083d8d057d22cbd4068ffd7299aceee.png)
AVL树，查询id=7，需要3次红黑树，查询id=7，需要4次
* **B树** 是一种多叉结构，并且平衡的。分叉越多，效率越高。 IO 读一个数据和读 100 个数据消耗的时间基本一致。其实满足了我们的要求。但是在范围查找时候依然是要依次进行。
* **B+树** 是对于B树的一个升级，区分开了普通节点和叶子节点。**每一个普通字节点都会出现在叶字节点中**。并且**在叶子节点中用了链表进行连接，方便读取叶子节点中的数据**。是我们的最佳选择

### B 树和 B+树有什么不同呢？

第一，**B 树一个节点里存的是数据，而 B+树存储的是索引**，所以 B 树里一个节点存不了很多个数据，但是 B+树一个节点能存很多索引，**B+树叶子节点存所有的数据**。

第二，**B+树的叶子节点是数据阶段用了一个链表串联起来，便于范围查找。**

通过 B 树和 B+树的对比我们看出，B+树节点存储的是索引，在单个节点存储容量有限的情况下，**单节点也能存储大量索引，使得整个 B+树高度降低，减少了磁盘 IO**。其次，**B+树的叶子节点是真正数据存储的地方，叶子节点用了链表连接起来，这个链表本身就是有序的**，在数据范围查找时，更具备效率

* 如何减少IO消耗的

树高决定IO的次数，多叉的B+树每个节点的数据多，并且这个数据是存放在一块的，对应的是数据库中的读取的最小单位页，一次IO就可以将这些数据读取出来，虽然比较的次数有可能会增加，但是在内存中的比较和磁盘IO相比差几个数量级，整体上效率还是提高了。

## 聚集索引和辅助索引（非聚集索引）

**Innodb中主索引是聚集索引，也就是索引和数据是在一起的**，位于.ibd文件中；而**辅助索引是单独的**。主键索引中普通节点都是位置信息，而**叶节点是数据**。而辅助索引中，**叶节点是主键索引中主键的key**，需要再到主键索引中去读取相应的数据（回表）。

之所以设计成这样是为了节省空间。相比起非聚集索引的mylsam，查询辅助索引时候性能略差。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210127162512306.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pjejU1NjY3MTk=,size_16,color_FFFFFF,t_70)



最后再总结一下什么时候需要给你的表里的字段加索引吧：   
1. 较频繁的作为查询条件的字段应该创建索引；
2. 唯一性太差的字段不适合单独创建索引，即使该字段频繁作为查询条件；（特异性）
3. 更新非常频繁的字段不适合创建索引。

## 联合索引、索引覆盖、最左前缀原则、索引下推
* 联合索引


在mysql中是可以构建联合索引的，但是需要注意**联合索引也是有先后顺序的**，可以理解为在多叉树上的查找还是按照联合索引的第一个来，在后续的叶节点中则是可以参考后一个索引进行查找。

建立（a,b）的联合索引，有a的单独索引
  1. 只查询a，可以使用联合索引。但是优先使用a的单独索引，因为一页可以放下的记录更多，IO更少。
  2. where a = 10 and b >15 或者 where a = 10 order by b DESC limit 3 优先使用联合索引，可以减少一次排序。

* 覆盖索引

如果我们正好有一个a的辅助索引，主键是b，我们的目标是 `select b from T where a between 3 and 5`，这样的话我们其实可以直接借助辅助索引就可以完成全部查找。完全不需要回表。因为辅助索引的叶节点的内容就是a。这就称为**索引覆盖**。

还有一个情况也会优先使用联合索引。如果是统计操作，且索引覆盖，优化器可以进行使用联合索引。
   ```sql
   select count(*) from buy_log where a>='2011-01-01' and a<'2011-02-01'
   ```
对（usrid, a）有联合索引，按理说不会走，但是因为是统计，并且是覆盖索引，因此可以使用联合索引。
**对于不能索引覆盖的情况，选择辅助索引的情况是通过辅助索引查找的数据是少量的。**
* 最左前缀原则


如果我们有一个（a,b）的联合索引。我们优先查找可以完成 `select * from user where  a like '张%' and b=10 and ismale=1;`的操作。因为辅助联合索引中是按照a进行排列的，然后是b。这符合我们的查找逻辑。

需要注意最左匹配时候，sql句子里面有and是会进行优化的。可以交换位置，以满足最左前缀。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210127165519802.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pjejU1NjY3MTk=,size_16,color_FFFFFF,t_70)


* 索引下推


可以在索引遍历过程中，对索引中包含的字段先做判断，直接过滤掉不满足条件的记录，减少回表次数。**通过索引下推对于非主键索引进行优化，可有效减少回表次数，从而提高效率。**

当我们创建一个用户表(userinfo),其中有字段：id,name,age,addr。我们将name,age建立联合索引。`select * from userinfo where name like "ming%" and age=20;`
![](https://pic3.zhimg.com/v2-04b4a496ab53eccc5feba150bf9fb7ea_r.jpg)
对于MySQL5.6之前：我们在索引内部首先通过name进行查找，在联合索引name,age树形查询结果可能存在多个，然后再拿着id值去回表查询，整个过程需要回表多次。

对于MySQL5.6之后：我们是在索引内部就判断age是否等于20，对于不等于20跳过。因此在联合索引name,age索引树只匹配一个记录，此时拿着这个id去主键索引树种回表查询全部数据，整个过程就回一次表。
### 数据库的索引存在磁盘还是内存？
一般来说，索引本身也很大，不可能全部存储在内存中，**因此索引往往以索引文件的形式存储的磁盘上**。这样的话，索引查找过程中就要产生磁盘I/O消耗，相对于内存存取，I/O存取的消耗要高几个数量级，所以评价一个数据结构作为索引的优劣最重要的指标就是在查找过程中磁盘I/O操作次数的渐进复杂度。换句话说，索引的结构组织要尽量减少查找过程中磁盘I/O的存取次数。

### 为什么使用自增id作为主键，而不是UUID
- UUID占16个字节，占用空间大，间接导致数据库性能下降
- 非主键索引B+树中都存有一个主键索引，相比整数id，大小增加很多
- UUID肯定比整数慢，另外非主键索引最终都会进行一次主键索引查找
- innodb 主键索引和数据存储位置相关（簇类索引），uuid 主键可能会引起数据位置频繁变动，严重影响性能。
- UUID目前不是顺序增长，做为主键写入导致，随机IO严重。
- UUID并不具有有序性，会导致B+树索引在写的时候有过多的随机写操作(连续的ID会产生部分顺序写);

> 为什么使用自增主键
- 可以顺序插入，防止裂页
   
  如果主键为自增 id 的话，mysql 在写满一个数据页的时候，直接申请另一个新数据页接着写就可以了。

  如果主键是非自增 id，为了确保索引有序，mysql 就需要将每次插入的数据都放到合适的位置上。

  容易造成页分列。

- 降低树的高度

主键占用空间越大，每个页存储的主键个数越少，B+树的深度会边长，导致IO次数会变多。
### 添加索引
1. 添加PRIMARY KEY（主键索引）
mysql>ALTER TABLE `table_name` ADD PRIMARY KEY ( `column` )

2. 添加UNIQUE(唯一索引)
mysql>ALTER TABLE `table_name` ADD UNIQUE ( `column` )
3. 添加INDEX(普通索引)
mysql>ALTER TABLE `table_name` ADD INDEX index_name ( `column` )
4. 添加FULLTEXT(全文索引)
mysql>ALTER TABLE `table_name` ADD FULLTEXT ( `column`)
5. 添加多列索引
mysql>ALTER TABLE `table_name` ADD INDEX index_name ( `column1`, `column2`, `column3` ) 
## 如何设计数据库的索引
 
1. 较频繁的作为查询条件的字段应该创建索引；
2. 唯一性太差的字段不适合单独创建索引，即使该字段频繁作为查询条件；（特异性）
3. 更新非常频繁的字段不适合创建索引。
4. 尽量可以使用索引覆盖，必要的时候建立联合索引。=和in可以乱序，比如a = 1 and b = 2 and c = 3 建立(a,b,c)索引可以任意顺序，**mysql的查询优化器会帮你优化成索引可以识别的形式**。
5. 索引列不能参与计算，保持列“干净”，比如from_unixtime(create_time) = ’2014-05-29’就不能使用到索引，原因很简单，b+树中存的都是数据表中的字段值，但进行检索时，需要把所有元素都应用函数才能比较，显然成本太大。所以语句应该写成create_time = unix_timestamp(’2014-05-29’)。

### 索引失效的场景
1. 有or的情况下几乎无法索引，除非所有的or的条件都有索引;
2. 复合索引未用左列字段，最左匹配原则;
3. like以%开头;
4. 需要类型转换;比如列类型是字符串，那一定要在条件中将数据使用引号引用起来,否则不使用索引
5. where中索引列有运算或者函数使用了函数; where id=id+1(ABS(id) = 1)
7. 如果mysql觉得全表扫描更快时（数据少）;
# 锁
## 锁的类型
mysql中的锁，可以分为全局锁，表锁和行锁。部分引擎不支持行锁，但是**Innodb是支持行锁。**

**全局锁**一般用于备份，尤其是对于不支持事务的引擎。对于Innodb引擎，我们一般不进行使用，因为我们可以MVCC得到一个版本快照进行备份。

对于**表锁**是很多引擎的最细锁，可以分为普通表锁lock tables … read/write，和元数据锁MDL（metadata lock)。保证读写的正确性。MDL会直到事务提交才释放，在做表结构变更的时候，你一定要小心不要导致锁住线上查询和更新。

行锁是Innodb的特色，也是Innodb流行的原因。**在InnoDB事务中，行锁是在需要的时候才加上的，但并不是不需要了就立刻释
放，而是要等到事务结束时才释放** 。解决不可重复读，别人无法事务执行期间进行修改。

**三级封锁协议**：
1. 对于事务写加X锁，事务完成释放（解决修改丢失） 
2. 对于读加S锁，完成读立刻释放 （解决脏读） 
3. 对于读加X锁，完成事务以后释放。（解决不可重复读）

**两段锁协议**： 是指所有的事务必须分两个阶段对**数据项加锁和解锁**。即事务分两个阶段，第一个阶段是获得封锁。事务可以获得任何数据项上的任何类型的锁，但是不能释放；第二阶段是释放封锁，事务可以释放任何数据项上的任何类型的锁，但不能申请。

如果你的事务中需要锁多个行，要把最可能造成锁冲突、最可能影响并发度的锁尽量往后放。

## Innodb中的行锁
首先行锁可以分为**共享锁S（读锁）和排他锁X（写锁）**。更细的层次上还可以分为两者**意向锁IS/IX**。执行逻辑是，如果你希望执行行锁，那么首先需要对更大的级别，如表，页等加上意向锁。**意向锁之间都是可以兼容的**。然后再对具体的行加行锁。其实意向锁不会阻止除全表扫描意外的请求。

![在这里插入图片描述](https://img-blog.csdnimg.cn/2021012723465728.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pjejU1NjY3MTk=,size_16,color_FFFFFF,t_70)

## 一致性非锁定读和一致性锁定读（当前读/对照读）
### 一致性非锁定读
一致性非锁定读：是在不需要锁的前提下。利用快照完成查询。对于可重复读的模式下，快照是在事务发起阶段创建的。**对于读提交，是在可以看到事务发起之后已提交的结果的。**

   - 概念：在**读已提交**和**可重复读**场景下，如果读取的行正在被`Delete`或者`update`而**加上了X锁**。此时InnoDB去读取一个快照数据，而不是等待。
   - 原理/实现：通过MVCC实现。
   - 特点：提高了并发性能，是默认读取方式，但是不同的事务隔离级别下，读取的MVCC的方法不同。以及不是所有的事务下都是 一致性非锁定读。
     - 读已提交：总是读取被锁定行的最新一份快照数据
     - 可重复读：读取事务开始时候的行数据版本。

### 一致性锁定读
一致性锁定读：是借助锁实现，其他的事务无法得到这一行的相关锁。

但是对于事务中的增删改业务，都是需要先读再改，这个读只能是借助行锁的一致性锁定读。如果其他事务正在使用，当前事务就会被阻塞。

   - 概念：通过对数据库加锁保证数据逻辑一致性。对于select语句有两种支持一致性锁定读的操作：
   ```sql
   select ... for update // 对读取行加一个X锁
   select ... lock in share mode // 对读取行记录加一个S锁
   ```
   

> 当前读与快照读
- 当前读


在mysql中存在当前读与快照读的区别。**对于简单的读取操作，是在事务开始的阶段就获得事务当前的快照，也就说只能看到事务请求之前已经提交的内容**。当前读是一种加锁的读取， 阻塞其他事务同时改动相同记录，避免出现安全问题。。
```sql
select...lock in share mode (共享读锁)
select...for update
update , delete , insert
```
实现方式是：next-key锁(行记录锁+Gap间隙锁)。对于**增删改**操作，是在命令进行前获取当前的快照，进行更新。<font color='red'>更新数据都是先读后写的，而这个读，只能读当前的值，称为“当前读”（current read）。</font>
- 快照读

单纯的select操作，不包括上述 `select ... lock in share mode, select ... for update`。　　　　

>Read Committed隔离级别：每次select都生成一个快照读。

>Read Repeatable隔离级别：开启事务后第一个select语句才是快照读的地方，而不是一开启事务就快照读。

实现方式：利用undolog和多版本并发控制MVCC。这种读取方式不会加锁，因此读操作时非阻塞的，因此也叫非阻塞读。

查询也是可重复的。同时在事务开始时候看无法看到正在活跃的事务提交的内容的。


## 死锁--锁的性能问题
同时我们不能忽视可能出现的锁问题。

死锁：是两个事务相互持有对方需要的锁，且都在等待彼此都不释放自己的所持有的锁。这种现象会消耗大量的系统资源。

解决办法一般有两个:
- **设置一个死锁等待时间每隔**，如50s，如果依然无法得到锁，**就释放自己的锁**，一会重试。另外就是
- **死锁检测**，发现死锁以后主动回滚死锁链条中的某个事务，让其他事务执行。当然死锁检测也是消耗资源的，因此我们需要合理的控制并发度。采取一些别的手段去预防死锁。（拆分资源，ConcurrentHashMap中的多个count加和的思路）
# 事务

事务的隔离其实是mysql中很大的一个问题，因为还会进而设计到锁，MVCC，版本快照等问题。

## 事务的特点
>事务的四大特点ACID，原子性，一致性，隔离性，持久性。
   - A 原子性：事务要不全部完成，要不全部失败回滚。使用redo log+undolog。
   - C 一致性：如果事务中的某个动作失败了，系统可以自动撤销事务并返回初始状态。利用undo log。
   - I 隔离性：多个事务之间不会相互干扰。通过各种锁，MVCC，串行化的方法。
   - D 持久性：事务提交以后将被数据库永久保存再次读取不会发生改变。通过redo log，bin log等。

## 隔离级别
mysql中的隔离级别可以分为，读未提交（read uncommitted）,读提交（read committed）、可重复读（repeatable read）和串行化（serilaizable ）。
- **读未提交** 是**隔离性**遭到了破坏，事务中还没提交的修改也会被其他正在进行的事务读取到，也就是**脏读**。
- **读提交**是只能看到其他事务已经提交的修改，但是可能出现**不可重复读**的问题。也就是一次事务中多次查询得到的结果不一样。解决方法：一致性非锁定读（MVCC）
- **可重复读** mysql的默认级别，每次看到在事务开始时候都是拿到了表的一张快照，可以重复查询。但是仍然存在**幻读**风险。
- **串行化** 最安全，效率最低，通过加锁的方式实现了更高的安全性。

## 隔离问题
>mysql中容易出现的隔离问题包括，
- 修改丢失: 发生在两个或多个事务同时访问和修改相同数据时。当一个事务正在修改数据时，另一个事务也尝试修改同一数据，但在第一个事务提交前，第二个事务的修改被覆盖或丢失。这种情况下，第二个事务的更新被第一个事务的更新覆盖，导致第二个事务的修改丢失。
- 脏读（dirty read）:发生在一个事务读取了另一个事务未提交的数据时。在某个事务进行修改并未提交时，另一个事务读取了这些未提交的数据，这样就读取到了不一致或无效的数据，称为脏读。解决方法：通过加锁。
- 不可重复读（nonrepeatable read）:两次读读到的内容不一样。、解决方法：一致性非锁定读（MVCC）
- 幻读（phantom read）：对于范围查找，突然多了几行数据。通过多版本并发控制（MVCC）+ 间隙锁（Next-Key Locking）



**Innodb引擎在可重复读隔离级别下，通过多版本并发控制（MVCC）+ 间隙锁（Next-Key Locking）防止幻影读**。（这个问题和网上一些文档不一致，查询了高性能mysql，Innodb确实是可以的。）

多版本控制（MVCC）会在读提交和可重复读中被使用。数据库中表的每一行都记录了一个版本，update时只拿出早于自己的版本。但是select会得到更新的版本。



## MVCC原理
**通过undo log实现的**。在undo log中存在一些回退操作，可以使得数据库得到在对应的row trx_id的历史记录。这里row trx_id在可重复读中是事务发起时候的任务号。对于读提交就是在指令前已经提交的内容。

回滚日志（undo log）是InnoDB引擎提供的日志，当事务对数据库进行修改，InnoDB引擎不仅会记录redo log，还会生成对应的undo log日志。如果事务执行失败或调用了rollback，导致事务需要回滚，就可以利用undo log中的信息将数据回滚到修改之前的样子。

但是undo log与redo log不一样，它属于**逻辑日志**。它对SQL语句执行相关的信息进行记录。当发生回滚时，InnoDB引擎会根据undo log日志中的记录做与之前相反的工作。undo log有两个作用，一是提供回滚，二是实现MVCC功能。


>使用REPEATABLE READ隔离级别，快照是基于执行第一个读操作的时间。

>使用READ COMMITTED隔离级别，快照被重置为每个一致的读取操作的时间。

## 幻读
### 什么是幻读？幻读与不可重复读有什么区别？
幻读也是一种在mysql中会出现的奇特现象，具体的表现每次**范围查询**得到的结果会发生增加。尤其注意区分幻读和不可重复读之间的区别。<font color=red>不可重复读是的重点是其他线程对当前数据的修改</font>，每次查询到的内容修改都是不一样的。但是<font color=red>对于幻读来说，重点在于其他线程对当前数据的增删</font>，可能上次查询的行数只有一个，这次就出现了两个。

比如在读提交的页面中，我们A事务查询id=3 的内容，只查询出来一个，但是我们在事务B中可以再插入一个id=3的东西。这样再A事务再次查询的时候就会发现两个。出现了幻读。

幻读的情况与不可重复读现象的解决方案也不一样。因为对于后者，我们只需要加一个行锁，保证别的事务不进行修改即可。但是对于幻读的场景，可能开始时候不存在这个行，我们也就无法加锁。

### 什么是间隙锁？
**间隙锁+行锁，称为next-key lock**

Innodb引擎为了在可重复读场景下解决这个幻读问题，引入了一个新的锁，叫**间隙锁**。这个加锁的实体是一个区间范围。比如我们是select id = 3。而表中的数据还有1，3，5，7。间隙锁就是加在（$-\infty$, 1] (1, 3] (3, 5] (5, 7] (7, $\infty$]上。保证在区间内也是不可以被操作的。间隙锁之间不存在排他的，这个性质也可能导致死锁。

### 可重复读隔离下的间隙锁加锁条件
两个原则两个优化一个bug

原则1：加锁的最小单位是nextkey-lock，前开后闭
原则2：只有访问到的对象才会进行加锁。（因此在普通索引上查询不会再主键索引上加间隙锁，如果是覆盖索引，不会加任何的锁）

优化1：对于等值查询，会向右遍历且遍历到第一个不满足的情况下时，nextkey-gap会退变为纯间隙锁。
优化2：对于等值查询，且给唯一索引加锁时候，nextkey-gap会退化为行锁

一个bug：对于唯一索引的范围查询，会向右遍历到第一个不满足的情况。

如果是limit的话，不会加最后一个间隙锁。


# 日志系统

分析器会通过词法和语法解析知道这是一条更新语句。优化器决定要使用ID这个索引。然后，执行器负责具体执行，找到这一行，然后更新。除此以外还有重要的概念是更新两个备份日志：**redo log** 和 **binlog**。

Redo Log用于持久化事务的修改操作，它用于在数据库发生崩溃或断电时进行恢复，以保证数据的一致性。

而Undo Log用于回滚和并发控制。是InnoDB存储引擎层面的概念，而不是Server层的概念。

binLog一般用于主从复制。

日志的出现是为了防止每次都直接往磁盘写入会造成太大的IO压力。因此采用备忘的形式，一段时间存一次。WAL技术（Write-Ahead Logging），它的关键点就是**先写日志，再写磁盘**。



## redo log（重做日志）

重做日志，两部分组成：一个是内存中的缓存 redo log buffer 是易失的，一个是重做日志文件redo log file是持久的。这里的日志包括了undo log 和redo log。redo log都是顺序写一般不需要读，undo log则可能随机读。

**redo log是Innodb引擎特有的，是物理存储日志， 记录的是实际的物理操作，例如对磁盘上的数据页进行的插入、更新或删除操作，是循环写入的**。在Innodb中，redo log的大小是固定的，有两个指针标识一个是write pos，一个是checkpos。标识了当前写的位置和未进行备份的位置，如果write追上（或者说套圈更好理解）了checkpos，系统就擦除部分内容。（也就是保存一段时间的记忆）
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210127112240956.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pjejU1NjY3MTk=,size_16,color_FFFFFF,t_70)

### redo log与WAL
redo log的背后其实是WAL（Write-ahead-log）。是一种**将磁盘的随机写改为顺序写**的操作。mysql每执行一条DML语句，会先把记录写入**redo log buffer**，后续某个时间点再一次性将多个操作记录写到redo log file。这种先写日志，再写磁盘的技术，就是WAL。
![在这里插入图片描述](https://img-blog.csdnimg.cn/2beb6ccd714746f18f457236955eb0c4.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAemN6NTU2NjcxOQ==,size_20,color_FFFFFF,t_70,g_se,x_16)

redo log落盘的时机一般有四个，首先是**每个事务commit**的时候，都会进行落盘。**当redo log的使用达到了redo log buffer size的一半**的时候也会进行落盘。其余的，系统每1s会后台执行一次持久化，以及数据库关闭时候也会落盘。   **在进行事务提交的时候，先将事务的所有日志文件写到重做日志文件进行持久化，然后事务提交。**

### WAL技术
我们都知道，数据库的最大性能挑战就是磁盘的读写，许多先辈在提供数据存储性能上绞尽脑汁，提出和实验了一套又一套方法。其实所有方案最终总结出来就三种：「随机读写改顺序读写」、「缓冲单条读写改批量读写」、「单线程读写改并发读写」。WAL 其实也是这两种思路的一种实现，**一方面 WAL 中记录事务的更新内容，通过 WAL 将随机的脏页写入变成顺序的日志刷盘，另一方面，WAL 通过 buffer 的方式改单条磁盘刷入为缓冲批量刷盘，再者从 WAL 数据到最终数据的同步过程中可以采用并发同步的方式**。这样极大提升数据库写入性能，因此，WAL 的写入能力决定了数据库整体性能的上限，尤其是在高并发时。

>redo log的写入方式

在计算机操作系统中，用户空间(user space)下的缓冲区数据，一般是无法直接写入磁盘的，必须经过操作系统内核空间缓冲区(即OS Buffer)。

- 日志最开始会写入位于存储引擎Innodb的redo log buffer，这个是在用户空间完成的。
- 然后再将日志保存到操作系统内核空间的缓冲区(OS buffer)中。
- 最后，通过系统调用fsync()，从OS buffer写入到磁盘上的redo log file中，完成写入操作。这个写入磁盘的操作，就叫做刷盘。


## binlog（归档日志）
**binlog是Sever层持有的，是逻辑存储日志，是追加写入的。写满以后会切换到下一个，不会覆盖。可以用户恢复和主从复制。**其实也可以使用row变成二进制日志。如果是STATEMENT可以记录语句。但是核心区别在于**binlog记录的是一个事务的具体操作内容，逻辑日志。而redolog是每个page的更改的物理情况**

### binlog和redo log不同？

   - redo/undo log是在存储引擎产生的，binlog则是更高的数据库层。
   - 记录内容不同：binlog记录的是sql语句，是逻辑日志。 redolog是物理日志，记录的是队每个页的修改。
   - 落盘时间不同：binlog在事务提交以后一次写入。redo/undo 是在事务进行中不断被写入。
   - 恢复时：优先使用物理日志redo log
>三者对比
- Redo Log 记录了物理修改操作，保证事务的持久性和恢复性。
- Undo Log 记录了撤销操作，用于回滚事务、提供一致性读取和并发控制。
- Binlog 记录了逻辑操作，用于数据恢复、数据复制和高可用性方案。

需要注意的是，Binlog 记录的是逻辑操作，而不是实际的物理操作。与 Redo Log（物理日志）不同，Binlog 记录的是对数据的逻辑修改，以 SQL 语句的形式记录下来。因此，当进行数据恢复或数据复制时，需要将 Binlog 中的逻辑操作重新应用到目标数据库上。

## undo log(重做日志)
undo log与redo log不一样，undo log是逻辑日志，可以将数据库逻辑的恢复到原来的样子。回滚日志（undo log）是InnoDB引擎提供的日志，当事务对数据库进行修改，InnoDB引擎不仅会记录redo log，还会生成对应的undo log日志。如果事务执行失败或调用了rollback，导致事务需要回滚，就可以利用undo log中的信息将数据回滚到修改之前的样子。

因为存在并发，所以是不可能物理恢复（redolog不可用）的（A线程和B线程同时修改，只能回退A的）。undo存放在数据库的一个特殊段内undo 段，在共享表空间中。除了用户回滚，可以用用于MVCC，可以读取到这一行之前的行版本信息。当然undo 的产生也会伴随redo 的产生。因为其实是逻辑存储了undo 日志。

**undo log有两个作用，一是提供回滚，二是实现MVCC功能。**

### 什么时候可以删除undo log？

  undolog 可以分为两种，insert undolog和update undolog。其中insert操作的记录只对事务本身可见，其他事务不可见，因此在事务提交以后就可以直接删除。而update undo log是对delete和update的操作，需要提供MVCC给其他事务。需要在事务提交以后放入undo log链表，等待purge线程删除。对于一个undo log 不在可能被使用的时候，才会被删除。
## 更新流程中的两阶段提交
为了保证事务和二进制日志的一致性，采用了两阶段事务2PC（下面这个有点矛盾，待检查）
- 当事务提交时，InnoDB存储引擎进行prepare操作。
- mysql数据库上层写入binlog
- 存储引擎层写入重做redo/undo日志
  - 组提交的两步
一般步骤2的操作完成，就确保了事务的提交。

图中浅色框表示是在InnoDB内部执行的，深色框表示是在执行器中执行的
![图中浅色框表示是在InnoDB内部执行的，深色框表示是在执行器中执行的](https://img-blog.csdnimg.cn/20210127112446577.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pjejU1NjY3MTk=,size_16,color_FFFFFF,t_70)

### 为什么需要两阶段提交呢?
**两阶段提交保证了事务的一致性**。不论mysql什么时刻crash，最终是commit还是rollback完全取决于MySQL能不能判断出binlog和redolog在逻辑上是否达成了一致。只要逻辑上达成了一致就可以commit，否则只能rollback。也就是说我们就看是不是redo log和binlog一致了。


如果不用两阶段提交的话，可能会出现这样情况：
- bin log写入之前，机器crash导致需要重启。重启后redo log继续重放crash之前的操作，而当bin log后续需要作为**备份恢复时，会出现数据不一致的情况**。
- 如果是bin log commit之前crash，那么重启后，发现redo log是prepare状态且bin log完整（bin log写入成功后，redo log会有bin log的标记），就会自动commit，让存储引擎提交事务。
- 两阶段提交就是为了保证redo log和binlog数据的安全一致性。只有在这两个日志文件逻辑上高度一致了。你才能放心的使用redo log帮你将数据库中的状态恢复成crash之前的状态，使用binlog实现数据备份、恢复、以及主从复制。



>为什么两阶段提交好，这个思想的本质是什么?
**因为最大程度降低了网络危险期，本质是分布式理论中的XA事务（分布式事务）**

1.准备阶段：事务协调者(事务管理器)给每个参与者(资源管理器)发送Prepare消息，每个参与者要么直接返回失败(如权限验证失败)，要么在本地执行事务，写本地的redo和undo日志，但不提交，到达一种“万事俱备，只欠东风”的状态。(关于每一个参与者在准备阶段具体做了什么目前我还没有参考到确切的资料，但是有一点非常确定：参与者在准备阶段完成了几乎所有正式提交的动作，有的材料上说是进行了“试探性的提交”，只保留了最后一步耗时非常短暂的正式提交操作给第二阶段执行。)

2.提交阶段：如果协调者收到了参与者的失败消息或者超时，直接给每个参与者发送回滚(Rollback)消息；否则，发送提交(Commit)消息；参与者根据协调者的指令执行提交或者回滚操作，释放所有事务处理过程中使用的锁资源。(注意:必须在最后阶段释放锁资源)

将提交分成两阶段进行的目的很明确，就是*尽可能晚地提交事务，让事务在提交前尽可能地完成所有能完成的工作*，这样，最后的提交阶段将是一个耗时极短的微小操作，这种操作在一个分布式系统中失败的概率是非常小的，也就是所谓的“网络通讯危险期”非常的短暂，这是两阶段提交确保分布式事务原子性的关键所在。（唯一理论上两阶段提交出现问题的情况是当协调者发出提交指令后当机并出现磁盘故障等永久性错误，导致事务不可追踪和恢复）

### 完整的落盘流程图
![在这里插入图片描述](https://img-blog.csdnimg.cn/b620e41be7674babb75d5db044d68fb0.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAemN6NTU2NjcxOQ==,size_20,color_FFFFFF,t_70,g_se,x_16)

## 了解组提交group commit么？
在传统的事务提交机制中，当一个事务提交时，数据库引擎会将事务的日志写入磁盘并进行持久化操作，确保事务的持久性。然而，每次事务提交都会引起磁盘的写入操作，这可能导致磁盘的性能瓶颈，特别是在高并发的数据库负载下。

Group Commit 技术通过将**多个事务的提交操作合并为一个批量操作来减少磁盘写入的次数，从而提高提交的效率**。

具体来说，当多个事务准备提交时，这些事务的提交操作被收集到一个批量中。然后，数据库引擎将整个批量的日志写入磁盘一次，并进行持久化操作，减少了磁盘写入的次数。这样可以减轻磁盘的负载，提高事务的提交性能。

但是，因为事务的提交操作被延迟到一定程度上，所以在出现数据库故障的情况下，**可能会丢失一些最近提交的事务**。

      对于事务提交会分为两个阶段：
   - 修改内存中事务对应信息，并将日志写入重做日志缓冲。
   - 调用fsync将确保日志都从重做日志缓冲写入磁盘。
   其中步骤2比较慢，因此在可以在多个事务的步骤1完成以后，一次fsync降低写盘次数。
   对于采用binlog的数据库，采用这种思路
   - Flush阶段：将每个事务的二进制日志写入内存
   - Sync阶段，将内存的二进制日志刷新到磁盘中，如果队列中有多个事务，一次刷盘就可以完成全部binlog写入。
   - Commit阶段，leader根据顺序调用存储引擎层事务的提交。
   有一组事务在commit阶段的时候，其他新事务可以进行flush。
  

# MySQL的主从复制
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210325225604571.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pjejU1NjY3MTk=,size_16,color_FFFFFF,t_70)

1. 首先写入master数据库。
2. 然后将变更数据写入二进制日志，binlog。
3. slave数据库会订阅master数据库的binlog文件，slave通过一个IO线程进行的主从同步。此时master数据库有一个Binlog Dump线程读取binlog日志与salve IO线程同步。
4. slave IO线程读取以后会先写入relay log 重放日志中。
5. slave数据库通过一个SQL线程读取relay log进行日志重放。

## 保证主从一致性
数据库的内部XA事务保证主从一致性


当事务提交时候，存储引擎首先进行一个prepare操作，将事务的uxid写入，然后写binlog。如果在redo log提交前，系统宕机了，mysql在重启以后先检查准备的uxid是否已经提交，若没有，引擎层再进行一次提交。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210711000233925.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pjejU1NjY3MTk=,size_16,color_FFFFFF,t_70)
# 性能优化

性能优化主要考虑三个方面，查询优化，索引优化，库表结构优化。 
1. 最大化利用索引；
2. 尽可能避免全表扫描；
3. 减少无效数据的查询；
## 如何查看命中索引
使用explain，分析执行计划。需要重点关注type、rows、filtered、extra。
- type：all全表扫< index索引全扫描< range索引范围扫描< ref 使用非唯一索引扫描或唯一索引前缀扫描，返回单条记录，常出现在关联查询中< eq_ref类似ref，区别在于使用的是唯一索引，使用主键的关联查询< const/system单条记录，系统会把匹配行中的其他列作为常数处理，如主键或唯一索引查询 < null MySQL不访问任何表或索引，直接返回结果.

- extra: 
  - Using filesort：MySQL需要额外的一次传递，以找出如何按排序顺序检索行。通过根据联接类型浏览所有行并为所有匹配WHERE子句的行保存排序关键字和行的指针来完成排序。然后关键字被排序，并按排序顺序检索行。
  - Using temporary：使用了临时表保存中间结果，性能特别差，需要重点优化
  - Using index：表示相应的 select 操作中使用了覆盖索引（Coveing Index）,避免访问了表的数据行，效率不错！如果同时出现 using where，意味着无法直接通过索引查找来查询到符合条件的数据。
  - Using index condition：MySQL5.6之后新增的ICP，using index condition就是使用了ICP（索引下推），在存储引擎层进行数据过滤，而不是在服务层过滤，利用索引现有的数据减少回表的数据。
## 解决慢查询问题

慢查询可能因为 ：
1. 查询了不需要的太多行或者列，解决：加limit，多表关联时候不应该返回所有列，禁止select *。
2. mysql服务器是不是在分析大量超过需要的行。（可以通过explain查看type，速度从慢到快依次是，全表扫描，索引扫描，范围扫描，唯一索引查询，常数引用。）如果发现扫描大量行，但是只是返回少数行。可以看看是不是可以进行覆盖扫描（减少回表）

重构查询方法：将大查询拆分为多个小查询。比如可能对占用锁，但是切成小的可以好一些。


寻找慢查询
```sql
# 取出使用最多的10条慢查询
mysqldumpslow -s c -t 10 /var/run/mysqld/mysqld-slow.log

# 取出查询时间最慢的3条慢查询
mysqldumpslow -s t -t 3 /var/run/mysqld/mysqld-slow.log 
```
## sql语句优化器的操作
[参考](https://zhuanlan.zhihu.com/p/56790651)


mysql采用了**基于成本的优化器**，会尝试预测一个查询的执行成本，选择成本最小的一个策略去执行。在数据库里面，扫描行数是影响执行代价的因素之一。扫描的行数越少，意味着访问磁盘数据的次数越少，消耗的CPU资源越少。当然，扫描行数并不是唯一的判断标准，优化器还会结合是否使用临时表、是否排序等因素进行综合判断。需要根据一系列的统计信息：包括表和索引的页面个数，索引的基数（区分度，不同索引值得数量），索引的分布，数据行的长度等。这个优化并不准确，因此可能错过最优策略。

MySQL整个查询优化器能够分为两个阶段，一是逻辑查询优化阶段，二是物理查询优化阶段。

优化可以分为静态优化和动态优化。静态就是代数关系（逻辑）对SQL语句进行等价变换。动态优化有查询上下文有关，比如数值。

优化包括重新定义的关联表的顺序，内外连接转换，覆盖索引扫描。

> 静态在第一次查询就完成，与数值无关，编译时优化。动态可以随时变化，类似运行时优化

> * 逻辑查询优化阶段：主要依据关系代数可以推知的规则和启发式规则。
MySQL淋漓尽致地使用了关系代数中可推定的各项规则，对投影、选择等操作进行句式的优化；对条件表达式进行了谓词的优化、条件化简；对连接语义进行了外连接、嵌套连接的优化；对集合、GROUPBY等尽量利用索引、排序算法进行优化。另外还利用子查询优化、视图重写、语义优化等技术对查询语句进行了优化。
>* 在物理查询优化阶段:通过贪婪算法，并依据代价估算模型，在求解多表连接顺序的过程中，对多个连接的表进行排序并探索连接方式，找出花费最小的路径，据此生成查询执行计划。在这个阶段，对于单表扫描和两表连接的操作，高效地使用了索引，提高了查询语句的执行速度。


* Q:mysql在使用like查询中，能不能用到索引？在什么地方使用索引呢？
  
LIKE 语句不允许使用 % 开头，否则索引会失效；
当真的需要两边都使用%来模糊查询时，只有当这个作为模糊查询的条件字段（例子中的name）以及所想要查询出来的数据字段（例子中的 id & name & age）都在索引列上时，才能真正使用索引，否则，索引失效全表扫描（比如多了一个 salary 字段）

* Q：为什么不select * from
1. 业务上无法修改
2. 读取了太多的冗余数据
3. 无法使用索引覆盖。
### Mysql优化的一些思路
[优化思路](https://zhuanlan.zhihu.com/p/265852739)
首先，对于MySQL层优化我一般遵从五个原则：

1. 减少数据访问： 设置合理的字段类型，启用压缩，通过索引访问等减少磁盘IO返回更少的数据： 
2. 只返回需要的字段和数据分页处理 减少磁盘io及网络io减少交互次数： 
3. 批量DML操作，函数存储等减少数据连接次数减少服务器CPU开销： 
4. 尽量减少数据库排序操作以及全表查询，减少cpu 内存占用利用更多资源： 
5. 使用表分区，可以增加并行操作，更大限度利用cpu资源

总结到SQL优化中，就三点:

 最大化利用索引；尽可能避免全表扫描；减少无效数据的查询；

### 走索引一定比不走索引的查询效率高么？
不一定，可能这个和Io读写有关系。如果我们希望得到前20%数据，我们可以直接读盘。这样写入的数据块可能比离散的分页查询读取数据页要好。

## 降低数据库的压力
1. 读写分离:读写分离主要是为了将数据库的读和写操作分不到不同的数据库节点上。主服务器负责写，从服务器负责读。另外，一主一从或者一主多从都可以。读写分离可以大幅提高读性能，小幅提高写的性能。因此，读写分离更适合单机并发读请求比较多的场景。

2. 分库分表:分库分表是为了解决由于库、表数据量过大，而导致数据库性能持续下降的问题。 
3. 负载均衡:负载均衡系统通常用于将任务比如用户请求处理分配到多个服务器处理以提高网站、应用或者数据库的性能和可靠性。
## Mysql的分页查询的优化

MySQL的limit工作原理就是先读取前面n条记录，然后抛弃前n条，读后面m条想要的，所以n越大，偏移量越大，性能就越差。

首先我们假设 主键索引id，辅助索引name，text。
1. 基础的分页`select * from t5 order by text limit 100000, 10;`这个方法很差，从执行计划可以看出，在大分页的情况下，MySQL没有走索引扫描，即使text字段我已经加上了索引。从200万数据中取出这10行数据的代价是非常大的，需要先排序查出前1000010条记录，然后抛弃前面1000000条。

这是为什么呢？MySQL数据库的查询优化器是采用了基于代价的，而查询代价的估算是基于CPU代价和IO代价。如果MySQL在查询代价估算中，认为全表扫描方式比走索引扫描的方式效率更高的话，就会放弃索引，直接全表扫描。这就是为什么在**超大分页的SQL查询中，明明给该字段加了索引，但是MySQL却走了全表扫描的原因。**如果小一些是会走索引的。

2. 优化：采用覆盖索引`select id, text from t5 order by text limit 1000000, 10;`
3. 优化：子查询优化，`select * from t5 where id>=(select id from t5 order by text limit 1000000, 1) limit 10;` 使用了id这个主键索引。这种方式假设数据表的id是连续递增的。因为子查询是在索引上完成的，而普通的查询时在数据文件上完成的，通常来说，索引文件要比数据文件小得多，所以操作起来也会更有效率。
4. 优化：延迟关联。我们可以使用JOIN，先在索引列上完成分页操作，然后再回表获取所需要的列。`select a.* from t5 a inner join (select id from t5 order by text limit 1000000, 10) b on a.id=b.id;`


## 如何科学的进行水平分表
水平分表：一个表由多个子表组成。不同的子表甚至可以在不同的数据库里，这样降低某个数据库集中的压力。

1. 设计好分布式全局唯一ID：twitter的雪花算法，UUID的算法。在不同的分片上也保证id不会重复

2. 分片字段该如何选择

在开始分片之前，我们首先要确定分片字段（也可称为“片键”）。很多常见的例子和场景中是**采用ID或者时间字段**进行拆分。这也并不绝对的，我的建议是**结合实际业务，通过对系统中执行的sql语句进行统计分析，选择出需要分片的那个表中最频繁被使用，或者最重要的字段来作为分片字段。**

有一些是**按照时间段拆分**，这样的可以保证大小固定，容义范围查找，但是存在问题是热度不一样，服务器负载不均衡。

或者对**数据取模进行拆分**。这样数据均匀，负载均衡。但是缺点是不容易扩容，比如原来变成3张子表，我现在切成4张就很麻烦。

## 如何进行跨库分页的几种常见方案
[参考](https://www.cnblogs.com/zhumengke/articles/12173314.html)

数据进行了分表拆分，然后我们需要`select * from t_user order by time offset 200 limit 100;`这里对于跨库就很难实现。

方案一：全局视野。每个库都返回3页数据；所得到的6页数据在服务层进行内存排序，得到数据全局视野；再取第3页数据，便能够得到想要的全局分页数据。缺点就是消耗资源。

方案二：一页一页。每次记录下来time_max，下次就`order by time where time>$time_max limit 100;`这样每次读取两页。

方案三（最佳）：二次查询。
order by time offset X/N limit Y;多页返回，找到最小值time_min；
between二次查询，order by time between timemin and time_i_max;

（4）设置虚拟time_min，找到time_min在各个分库的offset，从而得到time_min在全局的offset；

（5）得到了time_min在全局的offset，自然得到了全局的offset X limit Y；


# Innodb引擎优化

## 体系架构

   InnoDB有多个内存块，可以认为这些内存块组成了一个大的内存池，负责维护进程/线程需要的内部数据结构、缓存磁盘的数据、redo log缓冲
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20210704201241542.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pjejU1NjY3MTk=,size_16,color_FFFFFF,t_70)

### 后台线程

 1. Master Tread：核心线程，负责将缓冲池的数据异步落盘，包括脏页刷新、合并插入缓冲（insert buffer）、undo页回收。
 2. IO thread：使用了AIO来处理IO请求，主要是负责IO请求的回调。包括四个线程，write、read、insert buffer、log IO thread。
 3. Purge thread：事务提交以后，undolog不在需要，这个线程用于回收已使用的undolog页。
 4. page cleaner thread：单独进行脏页的刷新。

### 内存池
 1. 缓冲池：从磁盘读到的页放在缓存池中，下次再请求先查询。从缓冲池落盘的操作不是每次都进行，而是利用**checkpoint**机制。缓存的类型如下图：大头是索引页和数据页，还有部分是插入缓冲和锁信息、自适应哈希索引等。
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20210704201829444.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pjejU1NjY3MTk=,size_16,color_FFFFFF,t_70)
 2. LRU、Free、Flush list：LRU进行管理，最频繁使用页放在队头，最少使用在队尾。并且有一个midpoint，防止一些全表扫描刷出去常用页，因此LRU可以被分为冷端和热端。在一定的时间后，冷端内容被刷到热端。Free list表示空闲的可以加入LRU的页。一般命中率不能低于95%，否则需要看看是不是因为全表扫描被污染了。Flush list记录了脏页。也就是脏页既在flush，也在lru中。
 3. redo log buffer：redolog每秒被刷回盘一次、或者事务提交时候、或者redo log buffer剩余空间小于1/2。redolog是可以重复使用的。

## 概念
### 什么是checkpoint
它定期将内存中的数据页（也称为缓冲池）中的被修改的数据写入磁盘上的数据文件。当应用程序对数据库进行修改时，InnoDB会将修改操作写入内存中的缓冲池中，而不是直接写入磁盘。这样可以提高写入性能，因为内存中的操作比磁盘上的操作快得多。

  为了避免数据丢失，一般事务数据库都使用WAL技术，**即事务提交时，先写redo log，再修改页**。Checkpoint是为了：缩短数据库恢复时间、缓冲池不够时，脏页落盘、redolog满了时候，脏页数据落盘，清理redolog。

   #### 何时触发checkpoint？

  数据库关闭时触发sharp checkpoint，落盘全部脏页。平时一般Fuzzy checkpoints，包括 主线程定期落盘、 LRU满了弹出脏页需要落盘（page clean thread）、 redolog满了需要刷回（page clean thread）、 数据页缓冲池满了。 
### 什么是LSN（log sequence number）
标记版本，8字节的数字，每个页有，重做日志也有，checkpoints也有。
### 什么是insert buffer？


核心是将 随机写 变为 顺序写，适用范围 **索引是辅助索引、索引不是唯一索引**。聚集索引一般是顺序写的，而辅助一般是离散的。而非聚集索引的插入很容易不是顺序的；唯一索引需要查表验证，因此也不能。

因此，**对于非聚集索引的插入和更新，先判断非聚集索引页在不在缓冲池中，如果在直接插入；若不在，先加入insert buffer中。随后以一定的频率将insert buffer和辅助索引页子节点的merge，这时候往往可以将多个操作合并为一个**。 合并Insert Buffer中的数据时，InnoDB会将相邻的行按照索引的顺序进行排序，然后以顺序写入的方式将这些行写入到表的数据文件中。 

存在的问题：宕机以后恢复很慢，因为没有实际的落盘。以及写密集情况下占用的缓冲池内存太多。使用了一个bitmap存储了辅助索引页的存在性和空间等信息。
#### insert buffer的原理
使用了一个B+树，非叶节点是search key，维护了需要插入的表id。叶子节点记录了实际需要插入的字段。执行实际插入的时间有三个：
1. 辅助索引页被读取到缓冲池
2. bitmap发现该辅助索引页已经无空间，
3. master thread定期刷

### 自适应哈希


InnoDB会监控表上各索引页的查询。如果观察到建立hash索引可以来带速度提升，则建立hash索引，称为自适应哈希。比如经常进行同一个索引的读写。
### 刷新邻接页 Flush Neighbor Page


刷新一个脏页时候，会检测该页所在区的所有页，如果是脏页，连带一起刷新。将多次IO变成一次。使用与机械硬盘，对于高性能的固态其实没必要。


## Innodb和MyISAM对比
建议一定使用Innodb

- 事务：支持；不支持
- 索引：聚集索引；非聚集索引，索引与数据分离。
- 锁：支持行级锁；只支持表级锁
- 主键：必须有唯一索引，主键；可以没有
- 外键：支持；不支持

Innodb的恢复更好，myISAM没有很好的备份。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210325104814313.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pjejU1NjY3MTk=,size_16,color_FFFFFF,t_70)

## 为什么推荐使用自增ID
答：自增ID可以保证每次插入时B+索引是从右边扩展的，可以避免B+树和频繁合并和分裂（对比使用UUID）。如果使用字符串主键和随机主键，会使得数据随机插入，效率比较差。


# sql语句
## join和union有什么区别
join是连接查询，将两个查询结果进行横向拼接。union是联合查询，类似于将查询结果纵向拼接在一起。
## count(*)与count(1)
count(字段)，根绝字段判断为不为不空，根据字段定义，考虑要不要累加返回值，既然你引擎都返回值了，那我server层 “ +1 ”
count(id),根据id主键取值，累加返回值，也是server层 “ +1 ”
count(1)，同样会遍历，但不取值，引擎告诉不为空那我就 “+1”
count(*),也不取值，而且人家还是经过优化的

根据上面的推倒，搜主键肯定比搜正常字段快， 不取值的一定比取值的快（我就是查数统计一下，你给我这一行所有的值也没啥用啊）， 优化过的比没优化过的快

以下排行是按照效率，而不是时间
count（*） > count（1） > count（id） > count（字段）

# 其他

## 单机MySQL
瓶颈：1.数据量太大，一个机器放不下；2.数据索引（B+tree），一个机器的内存放不下；3。访问量大（无法读写分离），一个服务器无法处理
## 缓存（读写分离）+MySQL
利用缓存提高读效率，不用每次都进行数据库的查询。
## 垂直拆分+水平拆分+集群
分库分表，解决写的压力
## 数据类型丰富，改用非关系型数据库
关系型数据库不方便。采用redis为代表的非关系型数据库。

键值对类型数据库：redis，主要是内存存储，查找速度快。
列存储数据库：HBase 分布式的分拣系统
文档型数据库：MongoDB


## mybatis中#和$的区别是什么

1. 传入的参数在SQL中显示不同

**#传入的参数在SQL中显示为字符串（当成一个字符串），会对自动传入的数据加一个双引号。**

例：使用以下SQL
`select id,name,age from student where id =#{id}`
当我们传递的参数id为 "1" 时，上述 sql 的解析为：
`select id,name,age from student where id ="1"`
$传入的参数在SqL中直接显示为传入的值。

例：使用以下SQL
`select id,name,age from student where id =${id}`
当我们传递的参数id为 "1" 时，上述 sql 的解析为：
`select id,name,age from student where id =1`

2. #可以防止SQL注入的风险（语句的拼接）；但$无法防止Sql注入。

3. $方式一般用于传入数据库对象，例如传入表名。

4. 大多数情况下还是经常使用#，一般能用#的就别用$；但有些情况下必须使用$，例：MyBatis排序时使用order by 动态参数时需要注意，用$而不是#。


##  如何处理海量数据中的topK
> 对于简单的topK问题就是直接用堆就可。

从何解决我们上面提到的两个问题：时间问题和空间问题

* 时间角度，我们需要设计巧妙的算法配合合适的数据结构来解决；例如常用到的数据结构：哈希、位图、堆、数据库（B树），红黑树、倒排索引、Trie树等。
* 空间角度：我们只需要分而治之即可，两大步操作：分治、合并。MapReduce就是基于这个原理

任何的搜索引擎（百度、Google等）都会将用户的查询记录到日志文件。对于百度这种公司，我们知道每天有很多Query查询，假设有100G的日志文件，只有一台4G内存的电脑，现在让你统计某一天热门查询的Top 100. （Top 100的统计是很有用意义的，可以提供给用户，知道目前什么是最热门的，关注热点，例如金庸老先生的去世，现在应该就是热门。）

1. 首先划分文件为25个文件，每个文件4g。然后我们计算每个文件的top50。
2. 这里需要用到hash，因为hash可以保证对于字符串的映射是均匀的。
3. 然后对于每个文件，我们内部可以使用字典的方法来统计频次。O(n) （对于字符串可以使用trie树 复杂度是O(n*len)len是平均长度）
4. 并且用一个给定大小的堆来维护出来复杂度。O(logK)
5. 最后进行归并。
