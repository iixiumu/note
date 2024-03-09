#   MySql事务隔离级别
![MySql事务隔离](https://i.loli.net/2019/05/05/5ccec4a37fccf.jpeg)

ISO 和 ANIS SQL 标准制定了四种事务隔离级别，而 InnoDB 遵循了 SQL:1992 标准中的四种隔离级别：READ UNCOMMITED、READ COMMITED、REPEATABLE READ 和 SERIALIZABLE；每个事务的隔离级别其实都比上一级多解决了一个问题：

RAED UNCOMMITED：使用查询语句不会加锁，可能会读到未提交的行（Dirty Read）；
READ COMMITED：只对记录加记录锁，而不会在记录之间加间隙锁，所以允许新的记录插入到被锁定记录的附近，所以再多次使用查询语句时，可能得到不同的结果（Non-Repeatable Read）；
REPEATABLE READ：多次读取同一范围的数据会返回第一次查询的快照，不会返回不同的数据行，但是可能发生幻读（Phantom Read）；
SERIALIZABLE：InnoDB 隐式地将全部的查询语句加上共享锁，解决了幻读的问题；
MySQL 中默认的事务隔离级别就是 REPEATABLE READ，但是它通过 Next-Key 锁也能够在某种程度上解决幻读的问题。
``` sql
CREATE TABLE `xiumu`.`money` (
  `id` INT NOT NULL AUTO_INCREMENT,
  `user` VARCHAR(45) NOT NULL,
  `money` INT NOT NULL,
  `update_at` TIMESTAMP NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`));
insert into xiumu.money(user, money) values("a", 100),("b", 200),("c",300),("d",400),("e", 500);
ALTER TABLE `xiumu`.`money`
RENAME TO  `xiumu`.`account` ;
```
The transaction isolation level. The default is REPEATABLE-READ.
GLOBAL
    SET GLOBAL transaction_isolation = 'READ-COMMITTED';
SESSION
    SET @@SESSION.transaction_isolation = value;
    SET SESSION transaction_isolation = value;
    SET transaction_isolation = value;
    SET @@transaction_isolation = value;
##  READ-UNCOMMITTED
``` sql
SET GLOBAL TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
SET transaction_isolation = 'READ-UNCOMMITTED';
SELECT @@transaction_isolation;
```
A|B
---|---
begin;|begin;
select * from account where id = 1;|select * from account where id = 1;
1 a 100|1 a 100
update account set money = money + 10 where id = 1;|~
select * from account where id = 1;|~
1 a 110|~
~|select * from account where id = 1;
~|1 a 110
rollabck|~
select * from account where id = 1;|~
1 a 100|~
~|select * from account where id = 1;
~|1 a 100
##  READ-COMMITTED（一般db默认）
``` sql
SET GLOBAL TRANSACTION ISOLATION LEVEL READ COMMITTED;
SET transaction_isolation = 'READ-COMMITTED';
SELECT @@transaction_isolation;
```
A|B
---|---
begin;|begin;
select * from account where id = 1;|select * from account where id = 1;
1 a 100|1 a 100
update account set money = money + 10 where id = 1;|~
select * from account where id = 1;|~
1 a 110|~
~|select * from account where id = 1;
~|1 a 100
commit|~
select * from account where id = 1;|~
1 a 110|~
~|select * from account where id = 1;
~|1 a 110
##  REPEATABLE-READ（MySQL默认）
``` sql
SET GLOBAL TRANSACTION ISOLATION LEVEL REPEATABLE READ;
SET transaction_isolation = 'REPEATABLE-READ';
SELECT @@transaction_isolation;
```
A|B
---|---
begin;|begin;
select * from account where id = 1;|select * from account where id = 1;
1 a 100|1 a 100
update account set money = money + 10 where id = 1;|~
select * from account where id = 1;|~
1 a 110|~
~|select * from account where id = 1;
~|1 a 100
commit|~
select * from account where id = 1;|~
1 a 110|~
~|select * from account where id = 1;
~|1 a 100

A|B
---|---
begin;|begin;
select * from account where money <= 200;|select * from account where money <= 200;
1 a 100;2 b 200|1 a 100;2 b 200
~|update account set money = 300 where id = 2;
select * from account where money <= 200;|select * from account where money <= 200;
1 a 100;2 b 200|1 a 100
~|update account set money = 200 where id = 3;
select * from account where money <= 200;|select * from account where money <= 200;
1 a 100;2 b 200|1 a 100;3 c 200
##  SERIALIZABLE
#   问题
##  脏读
事务A读取了事务B更新的数据，然后B回滚操作，那么A读取到的数据是脏数据。读未提交时发生。
##  不可重复读
事务A多次读取同一数据，事务B在事务A多次读取的过程中，对数据作了更新并提交，导致事务A多次读取同一数据时，结果不一致。读已提交时发生。
##  幻读
事务A对表中的数据进行了修改，涉及到表中的全部行。同时，事务B也修改这个表中的数据，向表中插入一行新数据。那么，事务A发现表中还有自己没有修改的行，就好象发生了幻觉一样。可重复读时发生，但是MySQL innodb 通过MVCC解决了。
# 隔离级别的实现
事务的机制是通过视图（read-view）来实现的并发版本控制（MVCC），不同的事务隔离级别创建读视图的时间点不同。

读未提交是不创建，直接返回记录上的最新值

读已提交是每条 SQL 创建读视图，在每个 SQL 语句开始执行的时候创建的。隔离作用域仅限该条 SQL 语句。

可重复读是每个事务重建读视图，整个事务存在期间都用这个视图。

串行化隔离级别下直接用加锁的方式来避免并行访问。

这里的视图可以理解为数据副本，每次创建视图时，将当前已持久化的数据创建副本，后续直接从副本读取，从而达到数据隔离效果。
# 快照读、当前读
在事务中，执行普通select查询之后，会创建快照，后面再执行相同的select语句时，查询的其实是前面生成的快照。这也就是为什么会有可重复读。

而如果执行
``` sql
select * from table where ? lock in share mode;
select * from table where ? for update;
insert into table values (…);
update table set ? where ?;
delete from table where ?;
```
会执行当前读，获取最新数据。回到前面的问题，如果事务B执行N-1操作，会触发当前读，读取事务A提交后的数据，也就是N-1，在此基础上执行-1操作，最终N变成N-2。

>   <= mysql5.7 tx_isolation
    >= mysql8.0
    [sysvar_transaction_isolation](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_transaction_isolation)
