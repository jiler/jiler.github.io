---
title: Spring事务简介
---
## 事务
### 事务的概念：
事务是一个操作序列，这些操作要么都执行，要么都不执行，它是一个不可分割的工作单位。
### 事务的四个基本要素（ACID）：
1. 原子性（Atomicity）：事务开始后，所有操作要么全部做完，要么全部不做。事务是一个不可分割的整体。
2. 一致性（Consistency）：事务开始前和结束后，数据库的完整性约束没有被破坏。从一种一致状态，达到另一种一致状态。A给B转账，A扣了钱，B加了钱。总钱数在事务前后是一致的。
3. 隔离性（Isolation）：不同事务之间的操作，互不干扰。A给B转账100，C给A转账200，这两件事是互不干扰的。
4. 持久性（Durability）：事务完成之后，就对数据库进行了修改，不能取消。
>注：我一开始学习的时候，不明白`持久性`这一特性，比如转账完成，转账转错了，就不能回滚吗？这时候，事务已经COMMIT，结束了，当然不能回滚。但是能重新提交另外一个事务来“抵消”错误的转账。持久性是针对一个事务的概念。要想撤回这个事务对数据库产生的影响，只能用另一个事务来抵消

### 事务并发的问题：
1. 丢失更新：两个事务T1和T2读入同一个数据并修改，T2提交的结果覆盖了T1提交的结果，导致T1的修改被丢失。
2. 脏读：事务A读取了事务B更新的数据，如果B回滚，那么A读到的数据就是脏数据。事务A的两次读操作分别在事务B的修改之后和回滚之后。
3. 不可重复读：事务A多次读取同一数据，事务B在A多次读取期间，对数据进行了更改并提交，导致A多次读取到的结果不一致。事务B写操作，在事务A的两个读操作之间完成（侧重于修改和删除，通过行锁就能解决）。
4. 幻读：事务A将所有人的年龄置空，事务B同时插入一条数据。事务A执行结束，发现还有一条数据年龄不为空（侧重新增，需要锁住表才能解决）。但是Innodb对于幻读有解决方案，对于在默认"可重复读"的隔离级别下，普通查询是“快照读”，是看不到别的事务插入数据的，只有在“当前读”的情况下才会出现幻读。

### 事务的隔离级别：
用不同的隔离级别会解决不同的事务并发问题：

隔离级别|丢失更新|脏读|不可重复读|幻读
--|:--:|:--:|:--|--:
读未提交（read uncommited）|不允许|允许|允许|允许
读已提交（read commited）|不允许|不允许|允许|允许
可重复读（repeatable read）|不允许|不允许|不允许|允许
可串行化（serializable）|不允许|不允许|不允许|不允许

用户通过 设置隔离级别 这个入口，来控制大多数的锁。比较少的用显示调用。通过设置合适的隔离级别，数据库会自动使用合适的锁。

读未提交：通过 排他写锁 来实现。两个事务不可同时写同一条数据。  
读已提交：大多数数据库的默认级别就是这个，比如Sql Server，Oracle  
可重复读：可以通过“共享读锁”和“排他写锁”实现。这是MYSQL默认的隔离级别。

### 数据库的锁：
- 加锁方式：两阶段协议锁（2PL：Two-Phase Locking）
具体含义是相对于“一次性锁协议”而言的，所谓“一次性锁协议”指的是事务开始时，一次性申请所有锁，之后不再申请任何锁，如果某个锁不可用，则申请不成功，事务不执行，在事务尾端，一次性释放所有锁。这样不会产生死锁问题。两阶段锁协议指的是，整个事务分为两个阶段，前一个阶段是加锁，后一个阶段是解锁。在加锁阶段，事务只能加锁，也可以操作数据，但是不能解锁。直到事务释放第一个锁，就进入解锁阶段，这时候事务只能解锁，也可以操作数据，但是不能加锁。这么做，让解锁不必发生在事务尾端，提高了事务并发度。但是可能会造成死锁。
- 锁的类型：共享锁（S Lock）、排它锁(X Lock)

- 根据锁的粒度 分为 页级锁 行级锁 和 表级锁；
- 根据锁的模式 分为 共享锁（S锁）、排他锁（X锁）、意向共享锁（IS锁）、意向排它锁（IX锁）
- 表级锁：表锁（分S锁、X锁）、MDL锁、意向锁（分S锁、X锁）
- 行级锁：记录锁（record-lock 分S锁、X锁）、间隙锁（Gap lock）、临键锁（Next-key lock）、插入意向锁

共享锁 会防止 排他锁 来加锁，但是允许其他 共享锁 来加锁。  
排他锁 会防止其他 排他锁 和 共享锁 来加锁。

简单记忆：意向锁之间不冲突、排它锁与其他锁都冲突、S锁与 IS锁/S锁 不冲突）

#### 表级锁
1. MDL锁：
全称metadata lock，又叫元数据锁。不需要显示使用，方为一个表时，会被自动加上。对一个表进行增删改查时，会加上MDL读锁；修改一个表时，会加上MDL写锁。

2. 表锁：
语法示例

``` sql
-- 锁定student表
lock tables student read; -- 可以执行，此时其他会话，可以读，会被阻塞；不能写，报错
select * from student where id = 1; -- 可以执行
select * from teacher where id = 1; -- 不能执行，只能读锁定的表
insert into student (id, name) values (10, 'studentName'); -- 不能执行，只能读，不能添删改
unlock tables; -- 可以执行，解锁

lock tables student write; -- 可以执行，此时其他会话，可以读写，会被阻塞
unlock tables; -- 可以执行，解锁
```

3. 意向锁：
表锁与行锁冲突，如果加表锁时，要遍历每行是否加了行锁，效率很低，因此首先加意向锁，就简单了。

#### 行级锁：
1. 记录锁（Record Lock）：单行记录的锁，锁加在索引上。
2. 间隙锁（Gap Lock）：范围锁，锁定一个范围，不包含记录本身。RR级别下才会有这个锁。
3. 临键锁（Next-Key Lock）：锁定一个范围，也锁定记录本身。RR级别下才会有这个锁。
4. 插入意向锁（Insert Intention Lock）：特殊的间隙锁，简称II Gap，代表有插入意向，只有insert才会有这个锁。

### 快照读 与 当前读：
一致性非锁定读（快照读）：普通的select，不包含如下sql:
``` sql
select * from student where id = 1 lock in share mode;
select * from student where id = 1 for update;
```
一致性锁定读（当前读）：除了上述sql之外的，还有insert、update、delete。
RC级别下，快照读是 读取 被锁定行数据 最新已提交的快照。  
RR级别下，快照读是 读取 事务开始时的快照。  

### 加锁机制
#### RC级别：
只有记录锁
#### RR级别：
原则1：加锁单位都是next-key lock，锁定的是前开后闭的区间
原则2：访问到的对象才会加锁；
优化1：索引上的等值查询，给唯一索引加锁的时候，next-key lock会退化成行锁。
优化2：索引上的等值查询，向右遍历 且 最后一个值不满足等值条件的时候，next-key lock会退化成间隙锁。

## Spring的事务
所有的数据库访问技术都有事务处理机制，这些技术提供了API用来开启事务、提交事务来完成数据操作，或发生错误时回滚数据。

Spring的事务机制是用统一的机制来处理不同数据访问技术的事务处理。它提供了一个PlatformTransactionManager接口，不同的数据库访问技术提供不同的实现。

数据访问技术|实现
--|--
JDBC|DataSourceTransactionManager
JPA|JpaTransactionManager
Hibernate|HibernateTransactionManager
JDO|JdoTransactionManager
分布式事务|JtaTransactionManager

### 声明式事务
用`@Transactional`注解的方式选择需要使用事务的方法就是声明式事务，这是AOP的另外一个应用。当此方法被调用时，Spring开启一个新的事务，无异常运行结束时，提交这个事务。
> 这个注解来自于org.springframework.transaction.annotation包，而不是javax.transaction包。

#### `@Transactional`的属性说明
<table>

  <tr><th>属性</th><th>含义</th><th>默认值</th></tr>
  <tr>
    <td rowspan="7" >propagation</td>
    <td>Propagation.REQUIRED，方法A被调用时，如果没有事务则新建一个事务。当方法A调用方法B时，方法B使用和A相同的事务。方法B有异常需要回滚时，整个事务回滚。</td>
    <td rowspan="7" >Propagation.REQUIRED</td>
  </tr>
  <tr><td>Propagation.REQUIRES_NEW，当方法A调用方法B时，无论是否当前有事务都去新建。这样，方法B有异常需要回滚时，A不回滚。</td></tr>
  <tr><td>Propagation.NESTED，跟REQUIRES_NEW类似，但是只支持JDBC，不支持JPA和Hibernate。</td></tr>
  <tr><td>Propagation.SUPPORTS，方法调用时，有事务就用事务，没有就不用。</td></tr>
  <tr><td>Propagation.NOT_SUPPORTED，强制不在事务中执行，若有事务，则在方法调用到结束的阶段，事务都会被挂起。</td></tr>
  <tr><td>Propagation.NEVER，有事务抛出异常。</td></tr>
  <tr><td>Propagation.MANDATORY，强制在事务中执行，若无事务抛出异常。</td></tr>

  <tr>
    <td rowspan="5" >isolation</td>
    <td>Isolation.READ_UNCOMMITTED</td>
    <td rowspan="5" >Isolation.DEFAULT</td>
  </tr>
  <tr><td>Isolation.READ_COMMITTED</td></tr>
  <tr><td>Isolation.REPEATABLE_READ</td></tr>
  <tr><td>Isolation.SERIALIZABLE</td></tr>
  <tr><td>Isolation.DEFAULT，使用当前数据库的隔离级别，Mysql默认是REPEATABLE_READ，Oracle和SQLServer默认是READ_COMMITTED。</td></tr>

  <tr>
    <td>timeout</td>
    <td>过期时间</td>
    <td>默认-1，不过期（当前数据库的事务过去时间）</td>
  </tr>
  <tr>
    <td>readOnly</td>
    <td>是否是只读事务</td>
    <td>false</td>
  </tr>
  <tr>
    <td>rollBackFor</td>
    <td>指定哪些异常可以引起回滚</td>
    <td>Throwable的子类</td>
  </tr>
  <tr>
    <td>noRollBackFor</td>
    <td>指定哪些异常，不引起回滚</td>
    <td>Throwable的子类</td>
  </tr>
</table>




>上述问题，其实是在一个事务内完成的，跟事务之间的执行顺序没关系。
由于对数据库的事务，锁，隔离级别已经忘记了，为了加深理解，做如下探索：



>
最近遇到遇到一个需求：
在一个请求（http接口请求）中完成下面两个过程
过程1：用户的操作产生了一些数据，需要将这些数据，写入mysql数据库不同的表。
过程2：把用户的这个操作产生的数据（比如过程1产生的自增ID），从数据库中查出来，进行二次加工，用到其他地方。

需求1：过程1执行完毕，再执行过程2，如果过程1执行出错（error），则不执行过程2；
这个需求很简单，只要是按照顺序写的同步代码，问题不大。

设计：过程1产生的数据是一些数据，存储到数据库之后，过程2再从数据库取数。

问题1：过程1的事务能控制，保证用户产生的非结构化数据，全部写入数据库或者全部不写入数据库。
如果加入了过程2，在同一个事务内，能否存入数据库后，直接查到？

问题2：springboot和mysql的默认情况，对是如何处理的？

带着以上问题，我搜索并实践，最终得到如下结果：
