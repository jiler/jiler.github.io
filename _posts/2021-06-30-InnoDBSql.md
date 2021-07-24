---
title: InnoDB SQL分析
---

## 语法
DDL语法

``` sql
-- 新增列
ALTER TABLE student ADD COLUMN `class_id` int(11) DEFAULT NULL COMMENT '班级id' after `student_name`;
-- 修改列
ALTER TABLE student MODIFY COLUMN `class_name` varchar(20) DEFAULT NULL COMMENT '班级名称';
-- 新建索引
ALTER TABLE student ADD INDEX idx_class_id(class_id);
-- 创建表时，带索引
CREATE TABLE `config_info` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '代理自增主键',
  `config_id` int(11) NOT NULL COMMENT '配置id',
  PRIMARY KEY (`id`),
  KEY `idx_config_id` (`config_id`),
  KEY `idx_other_id` (`other_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='配置表';
-- update 错误写法
update student set age = 1 and student_name = 'lilei' where id = 1;
-- 上述sql意思是：age值是bool表达式，如果name是lilei，则age=1，否则age=0
update student set age = (1 and student_name = 'lilei') where id = 1;
-- update正确写法
update student set age = 1, student_name = 'lilei' where id = 1;
-- select错误写法(sql5.6不报错class_name随机，sql5.7报错，sql8报错)
select class_id, count(id), class_name from student group by class_id;
-- select正确写法
select class_id, count(id), class_name from student group by class_id, class_name;

```

## explain

``` sql
explain select * from student where id = 10
```

上述sql执行结果如下：  

id|select_type|table|partitions|type|possible_keys|key|key_len|ref|rows|filtered|Extra
--|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|--:
1|SIMPLE|student|NULL|const|PRIMARY|PRIMARY|8|const|1|100.00|NULL

字段|含义
--|--
id|数字越大越优先执行，如果数字一样大，从上到下执行
select_type|simple表示不用union或不包含子查询，还有union、subquery
table|查询表名，如果用到别名，则显示别名
partitions|使用分区表时出现
**type**|后面详细说明
possible_keys|查询可能使用到的索引，都列出来
**key**|真正使用的索引
key_len|用于处理查询的索引长度
ref|如果使用常数等值查询，则显示const，如果是连接查询，则被驱动表的这个位置 显示驱动表的关联字段
**rows**|预估扫描行数，越小越好
filtered|存储引擎返回的数据在server层过滤后，剩下多少满足查询要求的记录数据比例，越大越好。
Extra|using filesort、using index、using join bugger、using union、using temporary、using where

Extra说明：filesort，说明要在server层做一次额外排序  

type字段类型|含义
--|--
ALL|全表扫描
index|扫描全索引
range|索引，范围<>
ref|索引查找，匹配单值
const|主键或唯一索引，只匹配一行数据

方向：
1. 利用索引，减少数据扫描范围，减少排序
2. 避免不正确、低效、复杂的sql
3. 防止单表容量、流量多大，设计要合理
4. 使用其他手段比如缓存、其他引擎、业务逻辑优化



## 优化
1. 不能在索引上做运算，比如`select * from user where id + 1= 10`，不会走索引。
2. 查询类型要一致，否则不走索引。比如，一个字段是字符串类型，如果执行`where number = 123`，实际执行的是`where cast(number as int) = 123`，这时候不走索引，
3. 避免使用前缀匹配模糊查询
4. 慎用count，`count(1) = count(*) = count(id)`，如果name有空值，则`count(1) != count(name)`
5. 不写复杂sql
6. limit偏移量不要写过大，
7. 避免负向查询：not、!=、<>、!<、!>、not exists、not in、not like等

## 并发控制
并发控制机制有两种实现：锁 和 MVCC（Multi-Version Concurrency Control多版本并发控制）
对于读多写少的场景，采用MVCC更合适，每个数据项都有多个副本，每个副本都有一个时间戳，根据协议内容，维护各版本。
乐观锁：读取数据项时，不加锁（乐观的认为不会有冲突），在更新时，知道最后提交才会检查冲突。这么做是为了提高并发度。为了方式ABA问题，引入版本号的概念。其实现就是MVCC，类似于CAS。
悲观锁：适用于读少写多的场景，悲观的认为修改会有冲突，因此通过锁来解决问题。

书籍推荐
MySQL必知必会
https://book.douban.com/subject/3354490/
高性能MySQL(5.5)
https://book.douban.com/subject/23008813/
MySQL技术内幕InnoDB存储引擎
https://book.douban.com/subject/24708143/
