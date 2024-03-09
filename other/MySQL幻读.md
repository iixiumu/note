# MySQL幻读

<https://dev.mysql.com/doc/refman/5.7/en/innodb-next-key-locking.html>

<https://dev.mysql.com/doc/refman/8.0/en/innodb-next-key-locking.html>

The so-called phantom problem occurs within a transaction when the same query produces different sets of rows at different times. For example, if a SELECT is executed twice, but returns a row the second time that was not returned the first time, the row is a “phantom” row.

To prevent phantoms, InnoDB uses an algorithm called next-key locking that combines index-row locking with gap locking. InnoDB performs row-level locking in such a way that when it searches or scans a table index, it sets shared or exclusive locks on the index records it encounters. Thus, the row-level locks are actually index-record locks. In addition, a next-key lock on an index record also affects the “gap” before that index record. That is, a next-key lock is an index-record lock plus a gap lock on the gap preceding the index record. If one session has a shared or exclusive lock on record R in an index, another session cannot insert a new index record in the gap immediately before R in the index order.


幻读有很多种出现形式，简单的SELECT不加条件的查询在RR下肯定是读不到隔壁事务提交的数据的。但是仍然可能在执行INSERT/UPDATE时遇到幻读现象。因为SELECT 不加锁的快照读行为是无法限制其他事务对新增重合范围的数据的插入的。

引用一个 github 上面的评论 地址：

Mysql官方给出的幻读解释是：只要在一个事务中，第二次select多出了row就算幻读。
a事务先select，b事务insert确实会加一个gap锁，但是如果b事务commit，这个gap锁就会释放（释放后a事务可以随意dml操作），a事务再select出来的结果在MVCC下还和第一次select一样，接着a事务不加条件地update，这个update会作用在所有行上（包括b事务新加的），a事务再次select就会出现b事务中的新行，并且这个新行已经被update修改了，实测在RR级别下确实如此。

如果这样理解的话，Mysql的RR级别确实防不住幻读

有道友回复 地址：

在快照读读情况下，mysql通过mvcc来避免幻读。
在当前读读情况下，mysql通过next-key来避免幻读。
select * from t where a=1;属于快照读
select * from t where a=1 lock in share mode;属于当前读

不能把快照读和当前读得到的结果不一样这种情况认为是幻读，这是两种不同的使用。所以我认为mysql的rr级别是解决了幻读的。

幻读，并不是说两次读取获取的结果集不同，幻读侧重的方面是某一次的 select 操作得到的结果所表征的数据状态无法支撑后续的业务操作。更为具体一些：select 某记录是否存在，不存在，准备插入此记录，但执行 insert 时发现此记录已存在，无法插入，此时就发生了幻读。
