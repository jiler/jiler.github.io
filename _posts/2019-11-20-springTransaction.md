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
4. 持久性（Durability）：事务完成之后，就对数据库进行了修改。不可回滚。
>注：我一开始学习的时候，不明白`持久性`这一特性，比如转账完成，转账转错了，就不能回滚吗？这时候，事务已经COMMIT，结束了，当然不能回滚。但是能重新提交另外一个事务来“抵消”错误的转账。持久性是针对一个事务的概念。要想撤回这个事务对数据库产生的影响，只能用另一个事务来抵消

### 事务并发的问题：
1. 丢失更新：两个事务T1和T2读入同一个数据并修改，T2提交的结果覆盖了T1提交的结果，导致T1的修改被丢失。
2. 脏读：事务A读取了事务B更新的数据，如果B回滚，那么A读到的数据就是脏数据。事务A的两次读操作分别在事务B的修改之后和回滚之后。
3. 不可重复读：事务A多次读取同一数据，事务B在A多次读取期间，对数据进行了更改并提交，导致A多次读取到的结果不一致。事务B写操作，在事务A的两个读操作之间完成。（侧重于修改和删除，通过行锁就能解决）。
4. 幻读：事务A将所有人的年龄置空，事务B同时插入一条数据。事务A执行结束，发现还有一条数据年龄不为空。（侧重新增，需要锁住表才能解决）

### 数据库的锁：
- 根据锁的粒度 分为 页级锁 行级锁 和 表级锁；
- 根据锁的类型 分为 共享锁 和 排他锁。
- 根据使用机制 分为

共享锁 会防止 排他锁 来加锁，但是允许其他 共享锁 来加锁。  
排他锁 会防止其他 排他锁 和 共享锁 来加锁。

### 事务的隔离级别：
用户通过 设置隔离级别 这个入口，来控制锁。通过设置合适的隔离级别，数据库会自动使用合适的锁。用不同的隔离级别会解决不同的事务并发问题：

隔离级别|丢失更新|脏读|不可重复读|幻读
--|:--:|:--:|:--|--:
读未提交（read uncommited）|不允许|允许|允许|允许
读已提交（read commited）|不允许|不允许|允许|允许
可重复读（repeatable read）|不允许|不允许|不允许|允许
可串行化（serializable）|不允许|不允许|不允许|不允许

读未提交：通过 排他写锁 来实现。两个事务不可同时写同一条数据。
读已提交：大多数数据库的默认级别就是这个，比如Sql Server，Oracle
可重复读：可以通过“共享读锁”和“排他写锁”实现。这是MYSQL默认的隔离级别。


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
