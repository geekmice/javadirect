
# Mysql

## Mysql 三大范式

第一范式：每个列都不可以再拆分。

第二范式：在第一范式的基础上，非主键列完全依赖于主键，而不能是依赖于主键的一部分；属性必须完全依赖于主键

第三范式：在第二范式的基础上，非主键列只依赖于主键，不依赖于其他非主键。

在设计数据库结构的时候，要尽量遵守三范式，如果不遵守，必须有足够的理由。比如性能。事实上我们经常会为了性能而妥协数据库的设计。

> 图 1.1

<table  border="1" width="400" cellpadding="0" cellspacing="0">
			<tr>
				<td colspan="2" rowspan="2">编号</td>
				<td colspan="2" rowspan="2">品名</td>
				<td colspan="2">进货</td>
				<td colspan="2">销售</td>
				<td colspan="2" rowspan="2">备注</td>
			</tr>
			<tr>
				<td>数量</td>
				<td>单价</td>
				<td>数量</td>
				<td>单价</td>
			</tr>
			<tr>
				<td colspan="2">&nbsp;</td>
				<td colspan="2">&nbsp;</td>
				<td>&nbsp;</td>
				<td>&nbsp;</td>
				<td>&nbsp;</td>
				<td>&nbsp;</td>
				<td colspan="2">&nbsp;</td>
			</tr>
		</table>


这张表就不符合第一范式规定的原子性，不符合关系型数据库的基本要求，在关系型数据库中创建这个表的操作就不能成功。不得不将数据表设计为如下形式

<table border="1" cellspacing="0" cellpadding="0" width="500">
	<tr>
		<td>编号</td>
		<td>品名</td>
		<td>进货数量</td>
		<td>进货单价</td>
		<td>销售数量</td>
		<td>销售单价</td>
		<td>备注</td>
	</tr>
	<tr>
		<td>&nbsp;</td>
		<td>&nbsp;</td>
		<td>&nbsp;</td>
		<td>&nbsp;</td>
		<td>&nbsp;</td>
		<td>&nbsp;</td>
		<td>&nbsp;</td>
	</tr>
</table>


> 图 1.2

<table border="1" cellspacing="0" cellpadding="0" width="500">
<tr>
  <th>学号</th>
  <th>姓名</th>
  <th>系名</th>
  <th>系主任</th>
  <th>课程</th>
  <th>分数</th>
</tr>
<tr>
  <td>1011011</td>
  <td>张三</td>
  <td>计算机</td>
  <td>汪强</td>
  <td>高数</td>
  <td>97</td>
</tr>
<tr>
  <td>1011011</td>
  <td>张三</td>
  <td>化学</td>
  <td>汪强</td>
  <td>英语</td>
  <td>87</td>
</tr>
<tr>
  <td>1011011</td>
  <td>张三</td>
  <td>计算机</td>
  <td>汪强</td>
  <td>C++</td>
  <td>67</td>
</tr>
<tr>
  <td>1011012</td>
  <td>李四</td>
  <td>化学系</td>
  <td>沈宇飞</td>
  <td>java</td>
  <td>57</td>
</tr>
<tr>
  <td>1011012</td>
  <td>李四</td>
  <td>化学系</td>
  <td>沈宇飞</td>
  <td>c#</td>
  <td>37</td>
</tr>
<tr>
  <td>1011013</td>
  <td>王五</td>
  <td>数学系</td>
  <td>沈宇</td>
  <td>spring</td>
  <td>97</td>
</tr>
</table>


_缺点_

- 表中第一行数据都存储了系名、系主任,数据的冗余太大
- 如果有一个新的系还没有开始找到学生，那么不能讲该系的信息添加到数据表中去，从数据表中看不到该系的存在
- 如果将某个系的学生信息全部删除，那么这个系在数据表里也就不存在了，但这个系还存在。
- 如果某个人要转系，那么为了保证数据库中数据的一致性，需要修改三条记录中系与系主任的数据

_依赖_
在数据表中，属性（属性组）X 确定的情况下，能完全退出来属性 Y 完全依赖于 X。

_部分依赖_
完全依赖是针对于属性组来说，当一组属性 X 能推出来 Y 的时候就说 Y 完全依赖于 X。

_完全依赖_
一组属性 X 中的其中一个或几个属性能推出 Y 就说 Y 部分依赖于 X。

![20211203032154](https://geekmice-blog.oss-cn-beijing.aliyuncs.com/blog/20211203032154.png)

该表中的存在部分依赖

- 姓名对学号存在部分依赖
- 系名对学号存在部分依赖
- 系主任对学号存在部分依赖

<table border="1" cellspacing="0" cellpadding="0" width="500"><table border="1" cellspacing="0" cellpadding="0" width="500">
<tr>
  <th>学号</th>
  <th>课程</th>
  <th>分数</th>
</tr>
<tr>
  <td>1011011</td>
  <td>高数</td>
  <td>98</td>
</tr>
</table>


<table border="1" cellspacing="0" cellpadding="0" width="500"><table border="1" cellspacing="0" cellpadding="0" width="500">
<tr>
  <th>学号</th>
  <th>姓名</th>
  <th>系名</th>
  <th>系主任</th>
</tr>
<tr>
  <td>1011011</td>
  <td>高数</td>
  <td>98</td>
  <td>沈宇飞</td>
</tr>
</table>


表一中分数完全依赖于学号和课程的属性
表二中姓名、系名、系主任完全依赖于学号的属性
第二范式消除了第一范式的部分依赖



## innodb 与 myisam 存储引擎区别？

MyISAM：以读写插入为主的应用程序，查询居多，比如博客系统、新闻门户网站。

Innodb：更新（删除）操作频率也高，或者要保证数据的完整性；并发量高，支持事务和外键。比如 OA 自动化办公系统。

## 什么是聚簇索引？何时使用聚簇索引与非聚簇索引？

聚簇索引：将数据存储与索引放到了一块，找到索引也就找到了数据
非聚簇索引：将数据存储于索引分开结构，索引结构的叶子节点指向了数据的对应行，myisam 通过 key_buffer 把索引先缓存到内存中，当需要访问数据时（通过索引访问数据），在内存中直接搜索索引，然后通过索引找到磁盘相应数据，这也就是为什么索引不在 key buffer 命中时，速度慢的原因

## 联合索引是什么？为什么需要注意联合索引中的顺序？

MySQL 可以使用多个字段同时建立一个索引，叫做联合索引。在联合索引中，如果想要命中索引，需要按照建立索引时的字段顺序挨个使用，否则无法命中索引。

## 不可重复读和幻读区别？

不可重复读重点在于 update 和 delete，而幻读的重点在于 insert。

## 事务出现问题

 脏读：指一个线程中的事务读取到了另外一个线程中未提交的数据。

 > 脏读是读到了别的事务回滚前的脏数据。比如事务B执行过程中修改了数据X，在未提交前，事务A读取了X，而事务B却回滚了，这样事务A就形成了脏读。

 不可重复读：指一个线程中的事务读取到了另外一个线程中提交的 update 的数据。

> 事务A首先读取了一条数据，然后执行逻辑的时候，事务B将这条数据改变了，然后事务A再次读取的时候，发现数据不匹配了，就是所谓的不可重复读了。

 幻读：指一个线程中的事务读取到了另外一个线程中提交的 insert 的数据。

 > 事务A首先根据条件索引得到N条数据，然后事务B改变了这N条数据之外的M条或者增添了M条符合事务A搜索条件的数据，导致事务A再次搜索发现有N+M条数据了，就产生了幻读。

## 索引有哪几种（重点）

主键索引: 数据列不允许重复，不允许为 NULL，一个表只能有一个主键。

唯一索引: 数据列不允许重复，允许为 NULL 值，一个表允许多个列创建唯一索引。

- 可以通过 `ALTER TABLE table_name ADD UNIQUE (column);` 创建唯一索引
- 可以通过 `ALTER TABLE table_name ADD UNIQUE (column1,column2);` 创建唯一组合索引

普通索引: 基本的索引类型，没有唯一性的限制，允许为 NULL 值。

组合索引：多个字段组合一起使用起效

8、索引基本原理

就是把无序的数据变成有序的查询

1. 把创建了索引的列的内容进行排序

2. 对排序结果生成倒排表

3. 在倒排表内容上拼上数据地址链

4. 在查询的时候，先拿到倒排表内容，再取出数据地址链，从而拿到具体数据

9、索引创建原则

1） 最左前缀匹配原则，组合索引非常重要的原则，mysql 会一直向右匹配直到遇到范围查询(>、<、between、like)就停止匹配，比如 a = 1 and b = 2 and c > 3 and d = 4 如果建立(a,b,c,d)顺序的索引，d 是用不到索引的，如果建立(a,b,d,c)的索引则都可以用到，a,b,d 的顺序可以任意调整。

2）较频繁作为查询条件的字段才去创建索引

3）更新频繁字段不适合创建索引

4）若是不能有效区分数据的列不适合做索引列(如性别，男女未知，最多也就三种，区分度实在太低)

5）尽量的扩展索引，不要新建索引。比如表中已经有 a 的索引，现在要加(a,b)的索引，那么只需要修改原来的索引即可。

6）定义有外键的数据列一定要建立索引。

7）对于那些查询中很少涉及的列，重复值比较多的列不要建立索引。

10、索引创建过多劣势
索引需要额外的磁盘空间，并降低写操作的性能。在修改表内容的时候，索引会进行更新甚至重构，索引列越多，这个时间就会越长。所以只保持需要的索引有利于查询即可。

11、B 树与 b+树区别？
在 B 树中，你可以将键和值存放在内部节点和叶子节点；但在 B+树中，内部节点都是键，没有值，叶子节点同时存放键和值。

B+树的叶子节点有一条链相连，而 B 树的叶子节点各自独立。

12、了解数据库事务
数据库的事务是指一组 sql 语句组成的数据库逻辑处理单元，在这组的 sql 操作中，要么全部执行成功，要么全部执行失败。
例子就是转账了，事务 A 中要进行转账，那么转出的账号要扣钱，转入的账号要加钱，这两个操作都必须同时执行成功，为了确保数据的一致性。

## 事务四大特性 (重点)

在 Mysql 中事务的四大特性主要包含：原子性（Atomicity）、一致性（Consistent）、隔离性（Isalotion）、持久性(Durable)，简称为 ACID。
原子性是指事务的原子性操作，对数据的修改要么全部执行成功，要么全部失败，实现事务的原子性，是基于日志的 Redo/Undo 机制。
持久性则是指在一个事务提交后，这个事务的状态会被持久化到数据库中，也就是事务提交，对数据的新增、更新将会持久化到数据库中。
在我的理解中，原子性、隔离性、持久性都是为了保障一致性而存在的，一致性也是最终的目的。

*原子性是基于日志的 Redo/Undo 机制，你能说一说 Redo/Undo 机制吗？*
Redo/Undo 机制比较简单，它们将所有对数据的更新操作都写到日志中。
Redo log 用来记录某数据块被修改后的值，可以用来恢复未写入 data file 的已成功事务更新的数据；Undo log 是用来记录数据更新前的值，保证数据更新失败能够回滚。
假如数据库在执行的过程中，不小心崩了，可以通过该日志的方式，回滚之前已经执行成功的操作，实现事务的一致性。

*可以举一个场景，说一下具体的实现流程吗？*

可以的，假如某个时刻数据库崩溃，在崩溃之前有事务 A 和事务 B 在执行，事务 A 已经提交，而事务 B 还未提交。当数据库重启进行 crash-recovery 时，就会通过 Redo log 将已经提交事务的更改写到数据文件，而还没有提交的就通过 Undo log 进行 roll back。

## 什么是事务的隔离级别？MySQL 的默认隔离级别是什么？（重点）

为了达到事务的四大特性，数据库定义了 4 种不同的事务隔离级别，由低到高依次为 Read uncommitted、Read committed、Repeatable read、Serializable，这四个级别可以逐个解决脏读、不可重复读、幻读这几类问题。

| 隔离级别         | 脏读 | 不可重复读 | 幻影读 |
| ---------------- | ---- | ---------- | ------ |
| READ-UNCOMMITTED | √    | √          | √      |
| READ-COMMITTED   | ×    | √          | √      |
| REPEATABLE-READ  | ×    | ×          | √      |
| SERIALIZABLE     | ×    | ×          | ×      |

## SQL 标准定义了四个隔离级别（重点）

- READ-UNCOMMITTED(读取未提交)： 最低的隔离级别，允许读取尚未提交的数据变更，可能会导致脏读、幻读或不可重复读。
- READ-COMMITTED(读取已提交)： 允许读取并发事务已经提交的数据，可以阻止脏读，但是幻读或不可重复读仍有可能发生。
- REPEATABLE-READ(可重复读)： 对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，可以阻止脏读和不可重复读，但幻读仍有可能发生。
- SERIALIZABLE(可串行化)： 最高的隔离级别，完全服从 ACID 的隔离级别。所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，该级别可以防止脏读、不可重复读以及幻读。

这里需要注意的是：Mysql 默认采用的 `REPEATABLE_READ` 隔离级别 Oracle 默认采用的 `READ_COMMITTED` 隔离级别

## MySQL锁
锁可以分为分享锁/读锁（Shared Locks）、排他锁/写锁（Exclusive Locks） 、间隙锁、行锁（Record Locks）、表锁。
在四个隔离级别中加锁肯定是要消耗性能的，而读未提交是没有加任何锁的，所以对于它来说也就是没有隔离的效果，所以它的性能也是最好的。

我： 对于串行化加的是一把大锁，读的时候加共享锁，不能写，写的时候，家的是排它锁，阻塞其它事务的写入和读取，若是其它的事务长时间不能写入就会直接报超时，所以它的性能也是最差的，对于它来就没有什么并发性可言。

我： 对于读提交和可重复读，他们俩的实现是兼顾解决数据问题，然后又要有一定的并发行，所以在实现上锁机制会比串行化优化很多，提高并发性，所以性能也会比较好。

行级锁 行级锁是 Mysql 中锁定粒度最细的一种锁，表示只针对当前操作的行进行加锁。行级锁能大大减少数据库操作的冲突。其加锁粒度最小，但加锁的开销也最大。行级锁分为共享锁 和 排他锁。

特点：开销大，加锁慢；会出现死锁；锁定粒度最小，发生锁冲突的概率最低，并发度也最高。

表级锁 表级锁是 MySQL 中锁定粒度最大的一种锁，表示对当前操作的整张表加锁，它实现简单，资源消耗较少，被大部分 MySQL 引擎支持。最常使用的 MYISAM 与 INNODB 都支持表级锁定。表级锁定分为表共享读锁（共享锁）与表独占写锁（排他锁）。

特点：开销小，加锁快；不会出现死锁；锁定粒度大，发出锁冲突的概率最高，并发度最低。

页级锁 页级锁是 MySQL 中锁定粒度介于行级锁和表级锁中间的一种锁。表级锁速度快，但冲突多，行级冲突少，但速度慢。所以取了折衷的页级，一次锁定相邻的一组记录。

特点：开销和加锁时间界于表锁和行锁之间；会出现死锁；锁定粒度界于表锁和行锁之间，并发度一般

## 乐观锁与悲观锁（重点）

### 乐观锁
顾名思义，就是很乐观，每次去拿数据的时候都认为别人不会修改，所以不会上锁，但是在更新的时候会判断一下在此期间别人有没有去更新这个数据，可以使用版本号等机制。乐观锁适用于多读的应用类型，这样可以提高吞吐量

#### 如何实现乐观锁

第一种方案
通过 数据版本（Version）记录机制实现，这是乐观锁最常用的一种实现方式。何谓数据版本？即为数据增加一个版本标识，一般是通过为数据库表增加一个数字类型的 “version” 字段来实现。当读取数据时，将 version 字段的值一同读出，数据每更新一次，对此 version 值加一。当我们提交更新的时候，判断数据库表对应记录的当前版本信息与第一次取出来的 version 值进行比对，如果数据库表当前版本号与第一次取出来的 version 值相等，则予以更新，否则认为是过期数据

第二种方案
在需要乐观锁控制的 table 中增加一个字段，名称无所谓，字段类型使用时间戳（timestamp）, 和上面的 version 类似，也是在更新提交的时候检查当前数据库中数据的时间戳和自己更新前取到的时间戳进行对比，如果一致则 OK，否则就是版本冲突。

#### 应用场景

比如A、B操作员同时读取一余额为1000元的账户，A操作员为该账户增加100元，B操作员同时为该账户扣除50元，A先提交，B后提交。最后实际账户余额为1000-50=950元，但本该为1000+100-50=1050。这就是典型的并发问题。

### 悲观锁
对于并发间操作产生的线程安全问题持悲观状态，悲观锁认为竞争总是会发生，因此每次对某资源进行操作时，都会持有一个独占的锁，然后再操作资源。

悲观锁采用的是「先获取锁再访问」的策略，来保障数据的安全。但是加锁策略，依赖数据库实现，会增加数据库的负担，且会增加死锁的发生几率。此外，对于不会发生变化的只读数据，加锁只会增加额外不必要的负担。在实际的实践中，对于并发很高的场景并不会使用悲观锁，因为当一个事务锁住了数据，那么其他事务都会发生阻塞，会导致大量的事务发生积压拖垮整个系统。

#### 场景

在商品购买场景中，当有多个用户对某个库存有限的商品同时进行下单操作。若采用先查询库存，后减库存的方式进行库存数量的变更，将会导致超卖的产生。

若使用悲观锁，当B用户获取到某个商品的库存数据时，用户A则会阻塞，直到B用户完成减库存的整个事务时，A用户才可以获取到商品的库存数据。则可以避免商品被超卖。

##### 初始化表结构和数据

```sql
CREATE TABLE `tbl_user` (
  `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
  `status` int(11) DEFAULT NULL,
  `name` varchar(255) COLLATE utf8_bin DEFAULT NULL,
  PRIMARY KEY (`id`)
);


INSERT INTO `tbl_user` (`id`, `status`, `name`)
VALUES
    (1,1,X'7469616E'),
    (2,1,X'63697479');
```

##### 事务操作

窗口1

```sql
// 关闭mysql数据库的自动提交属性
set autocommit=0;

// 开启事务
BEGIN;

SELECT * FROM tbl_user where id=1 for update;
```

窗口2

```sql
SELECT * FROM tbl_user where id=1 for update;
```

![image-20211214200901760](http://oss.geekmice.cn/blog/image-20211214200901760.png)

#### 如何使用

如何使用悲观锁

用法：SELECT … FOR UPDATE;

如下操作

```csharp
select * from tbl_user where id=1 for update;
```

获取锁的前提：结果集中的数据没有使用排他锁或共享锁时，才能获取锁，否则将会阻塞。

需要注意的是， `FOR UPDATE` 生效需要同时满足两个条件时才生效：

- 数据库的引擎为 innoDB
- 操作位于事务块中（BEGIN/COMMIT）



## 子查询

 一般在子查询中，程序先运行在嵌套在最内层的语句，再运行外层。因此在写子查询语句时，可以先测试下内层的子查询语句是否输出了想要的内容，再一层层往外测试，增加子查询正确率。否则多层的嵌套使语句可读性很低。

`IN、NOT IN、EXISTS、NOT EXISTS`

## 关联查询

外连接，内连接，自连接

内连接：inner join

所有查询出的结果都是能够在连接的表中有对应记录的

外连接：左外连接（left outer join 《=》left jon），右外连接(right outer join 《=》right join)，全外连接(union)

自连接：自连接查询就是当前表与自身的连接查询，关键点在于虚拟化出一张表给一个别名

自连接查询一般用作表中的某个字段的值是引用另一个字段的值，比如权限表中，父权限也属于权限。

## SQL 优化（重点）

### SQL 语句优化

索引失效情况，全表扫描的情况

1、查询 SQL 尽量不要使用 select \*，而是 select 具体字段

2、如果知道查询结果只有一条或者只要最大/最小一条记录，建议用 limit 1

3、应尽量避免在 where 子句中使用 or 来连接条件，or 可能使索引失效

4、优化你的 like 语句

```sql
反例
select
 userId，name
from
 user
where
 userId like
'%123'
;
```

```sql
正确
select
 userId，name
from
 user
where
 userId like
'123%'
;
```

5、应尽量避免在 where 子句中对字段进行表达式操作，这将导致系统放弃使用索引而进行全表扫

6、选择最合适的字段属性：类型、⻓度、是否允许 NULL 等；尽量把字段设为 not null，⼀⾯查询时对⽐是否为 null；

7、避免全表扫描，⾸先应考虑在 where 及 order by 涉及的列上建⽴索引。

8、应尽量避免在 where ⼦句中对字段进⾏ null 值判断、使⽤!= 或 <> 操作符，否则将导致引擎放弃使⽤索引⽽进⾏全表扫描

9、应尽量避免在 where ⼦句中使⽤ or 来连接条件，如果⼀个字段有索引，⼀个字段没有索引，将导致引擎放弃使⽤索引⽽进⾏全表扫描

10、in 和 not in 也要慎⽤，否则会导致全表扫描

11、尽可能的使⽤ varchar/nvarchar 代替 char/nchar ，因为⾸先变⻓字段存储空间⼩，可以节省存储空间，其次对于查询来 说，在⼀个相对较⼩的字段内搜索效率显然要⾼些。

12、尽量使⽤数字型字段，若只含数值信息的字段尽量不要设计为字符型，这会降低查询和连接的性能，并会增加存储开销。这是因为 引擎在处理查询和连 接时会逐个⽐较字符串中每⼀个字符，⽽对于数字型⽽⾔只需要⽐较⼀次就够了。

13、索引并不是越多越好，索引固然可以提⾼相应的 select 的效率，但同时也降低了 insert 及 update 的效率，因为 insert 或 update 时有可能会重建索引，所以怎样建索引需要慎重考虑，视具体情况⽽定。⼀个表的索引数最好不要超过 6 个，若太多则应考虑⼀些不常使 ⽤到的列上建的索引是否有 必要。

14、select count(\*) from table；这样不带任何条件的 count 会引起全表扫描，并且没有任何业务意义，是⼀定要杜绝的。

15、在使⽤索引字段作为条件时，如果该索引是复合索引，那么必须使⽤到该索引中的第⼀个字段作为条件时才能保证系统使⽤该索 引，否则该索引将不会被使⽤，并且应尽可能的让字段顺序与索引顺序相⼀致。

16、应尽量避免在 where ⼦句中对字段进⾏函数操作，这将导致引擎放弃使⽤索引⽽进⾏全表扫描。

### 表结构优化

#### 将字段很多的表分解成多个表

对于字段较多的表，如果有些字段的使用频率很低，可以将这些字段分离出来形成新表。

因为当一个表的数据量很大时，会由于使用频率低的字段的存在而变慢。

单表的字段应该少而精，那多少合适呢？一般单表字段上限控制在20到50个。

大表：当一个表的数据超过千万行的时候，就会对数据库造成影响

大事务：运行的时间比较长，操作的数据比较多的事务

风险：

锁定太多的数据，造成大量的阻塞和锁超时

回滚所需要的时间比较长

执行时间长，容易造成主从延迟

如何处理大事务？避免一次处理太多的数据

移出不必要在事务中的SELECT操作

#### 增加中间表

对于需要经常联合查询的表，可以建立中间表以提高查询效率。

通过建立中间表，将需要通过联合查询的数据插入到中间表中，然后将原来的联合查询改为对中间表的查询。

比如权限管理（五个表）

#### 增加冗余字段

简单介绍

冗余字段，我们有一个表是记录图片的，另一个表是记录商品的。

我们可以在图片你放商品图片里的 url，同时商品里放图片 id 和图片 URL，这两个字段是重复的，这就是数据冗余，设计数据表时应尽量遵循范式理论的规约，尽可能的减少冗余字段，让数据库设计看起来精致、优雅。但是，合理的加入冗余字段可以提高查询速度。

表的规范化程度越高，表和表之间的关系越多，需要连接查询的情况也就越多，性能也就越差。

注意：

冗余字段的值在一个表中修改了，就要想办法在其他表中更新，否则就会导致数据不一致的问题。

#### 三大范式




### 大表数据查询优化

表结构优化+SQL语句优化+索引优化

添加缓存redis

主从复制，读写分离；

①当Master节点进行insert、update、delete操作时，会按顺序写入到binlog中。

②salve从库连接master主库，Master有多少个slave就会创建多少个binlog dump线程。

③当Master节点的binlog发生变化时，binlog dump 线程会通知所有的salve节点，并将相应的binlog内容推送给slave节点。

④I/O线程接收到 binlog 内容后，将内容写入到本地的 relay-log。

⑤SQL线程读取I/O线程写入的relay-log，并且根据 relay-log 的内容对从数据库做对应的操作。

垂直拆分，根据你模块的耦合度，将一个大的系统分为多个小的系统，也就是分布式系统；

为搜索字段建索引；

分表分库分区

分表

水平分表

**水平分表是在同一个数据库内，把同一个表的数据按一定规则拆到多个表中。**

目的：**为了我解决单表数据量大的问题。**

> 水平分表是分割行，将一个表的记录分配到两个表，不会更改表结构
>
> 库内的水平分表，解决了单一表数据量过大的问题，分出来的小表中只包含一部分数据，从而使得单个表的数据量变小，提高检索性能。

垂直分表

**目的：将冷数据（访问频次较低的数据）提取出来，提高热数据（访问频次较高的数据）的操作效率。**

**将一个表按照字段分成多表，每个表存储其中一部分字段。**

> 垂直分表分割列数据，生成新的表，会改变表结构
>
> 充分发挥热门数据的操作效率，商品信息的操作的高效率不会被商品描述的低效率所拖累

分库

**水平分库是把同一个表的数据按一定规则拆到不同的数据库中，每个库可以放在不同的服务器上。**

**水平分库是把不同表拆到不同数据库中，它是对数据行的拆分，不影响表结构**

**垂直分库是指按照业务将表进行分类，分布到不同的数据库上面，每个库可以放在不同的服务器上，它的核心理念是专库专用。**

### mysql 执行计划

`id , type , key , rows , extra`

#### id

select 查询的序列号，包含一组数字，表示查询中执行 select 子句或操作表的顺序
1、id 相同：执行顺序由上至下

2、id 不同：如果是子查询，id 的序号会递增，id 值越大优先级越高，越先被执行
3、id 相同又不同（两种情况同时存在）：id 如果相同，可以认为是一组，从上往下顺序执行；在所有组中，id 值越大，优先级越高，越先执行

#### type

访问类型，sql 查询优化中一个很重要的指标，结果值从好到坏依次是：

```
system > const > eq_ref > ref > fulltext > ref_or_null > index_merge > unique_subquery > index_subquery > range > index > ALL
```

`一般来说，好的 sql 查询至少达到 range 级别，最好能达到 ref`

1、system：表只有一行记录（等于系统表），这是 const 类型的特例，平时不会出现，可以忽略不计

2、const：表示通过索引一次就找到了，const 用于比较 primary key 或者 unique 索引。因为只需匹配一行数据，所有很快。如果将主键置于 where 列表中，mysql 就能将该查询转换为一个 const

3、eq_ref：唯一性索引扫描，对于每个索引键，表中只有一条记录与之匹配。常见于主键 或 唯一索引扫描。

4、ref：非唯一性索引扫描，返回匹配某个单独值的所有行。本质是也是一种索引访问，它返回所有匹配某个单独值的行，然而他可能会找到多个符合条件的行，所以它应该属于查找和扫描的混合体

5、range：只检索给定范围的行，使用一个索引来选择行。key 列显示使用了那个索引。一般就是在 where 语句中出现了 bettween、<、>、in 等的查询。这种索引列上的范围扫描比全索引扫描要好。只需要开始于某个点，结束于另一个点，不用扫描全部索引

6、index：Full Index Scan，index 与 ALL 区别为 index 类型只遍历索引树。这通常为 ALL 块，应为索引文件通常比数据文件小。（Index 与 ALL 虽然都是读全表，但 index 是从索引中读取，而 ALL 是从硬盘读取）

7、ALL：Full Table Scan，遍历全表以找到匹配的行

#### key

实际使用的索引，如果为 NULL，则没有使用索引。

查询中如果使用了覆盖索引，则该索引仅出现在 key 列表中

#### rows

 根据表统计信息及索引选用情况，大致估算出找到所需的记录所需要读取的行数

#### extra

不适合在其他字段中显示，但是十分重要的额外信息

1、Using filesort ：
mysql 对数据使用一个外部的索引排序，而不是按照表内的索引进行排序读取。也就是说 mysql 无法利用索引完成的排序操作成为“文件排序”

2、Using temporary：
使用临时表保存中间结果，也就是说 mysql 在对查询结果排序时使用了临时表，常见于 order by 和 group by

3、Using index：
表示相应的 select 操作中使用了覆盖索引（Covering Index），避免了访问表的数据行，效率高
如果同时出现 Using where，表明索引被用来执行索引键值的查找（参考上图）
如果没用同时出现 Using where，表明索引用来读取数据而非执行查找动作

4、Using where ：
使用了 where 过滤

5、Using join buffer ：
使用了链接缓存

6、Impossible WHERE：
where 子句的值总是 false，不能用来获取任何元祖

7、select tables optimized away：
在没有 group by 子句的情况下，基于索引优化 MIN/MAX 操作或者对于 MyISAM 存储引擎优化 COUNT（\*）操作，不必等到执行阶段在进行计算，查询执行计划生成的阶段即可完成优化

8、distinct：
优化 distinct 操作，在找到第一个匹配的元祖后即停止找同样值得动作

![img](https://img-blog.csdn.net/20170516093515092?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd3VzZXl1a3Vp/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

执行顺序
1（id = 4）、【select id, name from t2】：select_type 为 union，说明 id=4 的 select 是 union 里面的第二个 select。

2（id = 3）、【select id, name from t1 where address = ‘11’】：因为是在 from 语句中包含的子查询所以被标记为 DERIVED（衍生），where address = ‘11’ 通过复合索引 idx_name_email_address 就能检索到，所以 type 为 index。

3（id = 2）、【select id from t3】：因为是在 select 中包含的子查询所以被标记为 SUBQUERY。

4（id = 1）、【select d1.name, … d2 from … d1】：select_type 为 PRIMARY 表示该查询为最外层查询，table 列被标记为 “derived3”表示查询结果来自于一个衍生表（id = 3 的 select 结果）。

5（id = NULL）、【 … union … 】：代表从 union 的临时表中读取行的阶段，table 列的 “union 1, 4”表示用 id=1 和 id=4 的 select 结果进行 union 操作。

[^2]:[MySQL高级 之 explain执行计划详解 ](https://blog.csdn.net/wuseyukui/article/details/71512793)

## mysql 聚合函数

 avg() 平均值
​ count() 计数
​ max() 最大值
​ min() 最小值
​ sum() 求和

## MySQL 获取当前时间

current_timestamp()、now() 年月日时分秒 `2021-12-12 19:26:32`

current_time() ，curtime() 时分秒 `19:26:22`

current_date(), curdate() 年月日 `2021-12-12`

## 存储过程

### 创建和调用




### 存储过程的参数



### 存储过程的变量



### 注释

## sql常见问题

### mysql 中 order by 和 group by 同时使用

写的顺序：select ... from... where.... group by... having... order by..
执行顺序：from... where...group by... having.... select ... order by...

解决方案：

```sql
# substring_index（“待截取有用部分的字符串”，“截取数据依据的字符”，截取字符的位置N）
# group_concat([DISTINCT] 要连接的字段 [Order BY ASC/DESC 排序字段] [Separator '分隔符'])

SELECT
	*
FROM
	reward
WHERE
	id IN (
		SELECT
			substring_index(
				group_concat(id ORDER BY datatime DESC),
				',',
				1
			)
		FROM
			reward
		GROUP BY
			uid
	)
ORDER BY
datatime DESC;
```

### SQL 实现重复数据

```sql
select * from student s where s.stuName in (select s2.stuName from student s2 having count(s.stuName) > 1) ;

-- 查询某个字段重复
select s.stuName name1 , count(s.stuName) count1 from student s group by s.stuName having count(s.stuName) > 1 order by count1 desc ;

-- 查询多个字段都重复
select s.stuName , s.stuSex , count(*) from student s group by s.stuName , s.stuSex having count(*) > 1
```

### SQL 实现某个字段分组第一条数据

```sql
-- over(partition by stu_no order by score_prize desc) 函数说明
-- 首先对stu_no中相同的进行分区，再stu_no相同情况下对score_prize排序
-- row_number()从1开始，为每一条分组记录返回一个数字，这里的row_number() over(partition by stu_no order by score_prize desc)对stu_no中相同的进行分区，再stu_no相同情况下对score_prize排序,之后的每条记录返回一个序号
select
	*
from
	(
	select
		row_number() over(partition by stu_no
	order by
		score_prize desc) as rownum,
		stu_no,
		course_no
	from
		score s 
) T
where
	T.rownum = 1
```

### SQL实现行专列

原来的表格

<table border='1'>
    <tr>
    <td>学号</td>
    <td>课程号</td>
    <td>成绩</td>
    </tr>
	<tr>
    <td>0001</td>
    <td>0001</td>
    <td>80</td>
    </tr>
	<tr>
    <td>0001</td>
    <td>0002</td>
    <td>90</td>
    </tr>
	<tr>
    <td>0002</td>
    <td>0003</td>
    <td>100</td>
    </tr>
	<tr>
    <td>0002</td>
    <td>0003</td>
    <td>70</td>
    </tr>
	<tr>
    <td>0001</td>
    <td>0003</td>
    <td>60</td>
    </tr>
	<tr>
    <td>0003</td>
    <td>0003</td>
    <td>70</td>
    </tr>
	<tr>
    <td>0002</td>
    <td>0003</td>
    <td>80</td>
    </tr>
	<tr>
    <td>0003</td>
    <td>0003</td>
    <td>10</td>
    </tr>
</table>

使用sql实现将该表行转列为下面的表结构

<table>
    <tr>
    <td>学号</td>    <td>课程号0001</td>    <td>课程号0002</td>    <td>课程号0003</td>
    </tr>
        <tr>
    <td>0001</td>    <td>80</td>    <td>90</td>    <td>99</td>
    </tr>
    <tr>
    <td>0002</td>    <td>80</td>    <td>90</td>    <td>99</td>
    </tr>
    <tr>
    <td>0003</td>    <td>80</td>    <td>90</td>    <td>99</td>
    </tr>
</table>

*实现*

1.**使用常量列输出目标表的结构**

```sql
select
	stu_no,
	'课程号0001',
	'课程号0002',
	'课程号0003'
from
	score;
```

3.**分组，并使用最大值函数max取出上图每个方块里的最大值**

```sql
select stu_no,
max(case course_no when '0001' then course_prize else 0 end) as '课程号0001',
max(case course_no when '0002' then course_prize else 0 end) as '课程号0002',
max(case course_no when '0003' then course_prize else 0 end) as '课程号0003'
from score
group by stu_no;
```

### 索引失效的情况

where 子句中对字段进行 null 值判断
where 子句中使用!=或<>操作符
where 子句中使用 or 来连接条件
in 和 not in 也要慎用，对于连续的数值使用between
like %aa%
where 子句中的“=”左边进行函数、算术运算或其他表达式运算,例如：select id from t where num/2=100 可改为 select id from t where num=100*2

*优化*

1. exists 代替in
2. 不要写select *
3. 尽量使用数字型字段；
4. 尽可能的使用 varchar 代替 char
5. 避免频繁创建和删除临时表，以减少系统表资源的消耗。

### 数据库表死锁如何解决（重点）

**如何查看死锁**

```sql
SHOW OPEN TABLES WHERE IN_USE > 0
```

**解决**

1、检测死锁；

```sql
SELECT * FROM INFORMATION_SCHEMA.INNODB_TRX;
```

2、杀死死锁进程；

```sql
kill id
```

3、检测现在的死锁状态；

```sql
SELECT * FROM INFORMATION_SCHEMA.INNODB_TRX;
```

4、重新执行需要正确执行的代码，运行成功，成功解决死锁；

```sql
select * from user where a > 2 for update ;
```

**第二种方案**

```sql
show processlist;
找到锁进程，kill id ;
```

### Java代码获取当前系统时间

一. 获取当前系统时间和日期并格式化输出

```java
SimpleDateFormat df = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");//设置日期格式
System.out.println(df.format(new Date()));// new Date()为获取当前系统时间
```

二. 在数据库里的日期只以年-月-日的方式输出

1、用 convert()转化函数

```
String sqlst = "select convert(varchar(10),bookDate,126) as convertBookDate from roomBook where bookDate between '2007-4-10' and '2007-4-25'";

System.out.println(rs.getString("convertBookDate"));
```

2、利用 SimpleDateFormat 类

```java
java中获取当前日期和时间的方法

import java.util.Date;
import java.util.Calendar;

import java.text.SimpleDateFormat;

public class TestDate{
public static void main(String[] args){
Date now = new Date();
SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy/MM/dd HH:mm:ss");//可以方便地修改日期格式


String hehe = dateFormat.format( now );
System.out.println(hehe);

Calendar c = Calendar.getInstance();//可以对每个时间域单独修改




int year = c.get(Calendar.YEAR);
int month = c.get(Calendar.MONTH);
int date = c.get(Calendar.DATE);
int hour = c.get(Calendar.HOUR_OF_DAY);
int minute = c.get(Calendar.MINUTE);
int second = c.get(Calendar.SECOND);
System.out.println(year + "/" + month + "/" + date + " " +hour + ":" +minute + ":" + second);
}
}
```



[mysql三大范式](https://www.cnblogs.com/gongcheng-/p/10901824.html#_label2)
[mysql优化，主从复制，读写分离，分库分表](https://blog.csdn.net/qq_21644303/article/details/118655850?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522163929617116780255289996%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=163929617116780255289996&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_click~default-2-118655850.pc_search_all_es&utm_term=mysql%E4%B8%BB%E4%BB%8E%E5%A4%8D%E5%88%B6&spm=1018.2226.3001.4187)

[什么是悲观锁 ](https://www.jianshu.com/p/8a70a4af7eac)
[MySQL表死锁问题的产生和解决 ](https://blog.csdn.net/xcc1974494/article/details/112842044)
[MySQL 存储过程 ](https://www.runoob.com/w3cnote/mysql-stored-procedure.html)