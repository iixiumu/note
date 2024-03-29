多版本并发控制(Multi-Version Concurrency Control, MVCC)是MySQL中基于乐观锁理论实现隔离级别的方式，用于实现读已提交和可重复读取隔离级别的实现。

实现(隔离级别为可重复读)
在说到如何实现前先引入两个概念：

系统版本号：一个递增的数字，每开始一个新的事务，系统版本号就会自动递增。

事务版本号：事务开始时的系统版本号。

在MySQL中，会在表中每一条数据后面添加两个字段：

创建版本号：创建一行数据时，将当前系统版本号作为创建版本号赋值

删除版本号：删除一行数据时，将当前系统版本号作为删除版本号赋值

SELECT
select时读取数据的规则为：创建版本号<=当前事务版本号，删除版本号为空或>当前事务版本号。

创建版本号<=当前事务版本号保证取出的数据不会有后启动的事物中创建的数据。这也是为什么在开始的示例中我们不会查出后来添加的数据的原因

删除版本号为空或>当前事务版本号保证了至少在该事物开启之前数据没有被删除，是应该被查出来的数据。

INSERT
insert时将当前的系统版本号赋值给创建版本号字段。

UPDATE
插入一条新纪录，保存当前事务版本号为行创建版本号，同时保存当前事务版本号到原来删除的行，实际上这里的更新是通过delete和insert实现的。

DELETE
删除时将当前的系统版本号赋值给删除版本号字段，标识该行数据在那一个事物中会被删除，即使实际上在位commit时该数据没有被删除。根据select的规则后开启懂数据也不会查询到该数据。

MVCC真的解决了幻读？
从最开始我们的测试示例和上面的理论支持来看貌似在MySQL中通过MVCC就解决了幻读的问题，那既然这样串行化读貌似就没啥意义了，带着疑问继续测试。

如何解决幻读
很明显可重复读的隔离级别没有办法彻底的解决幻读的问题，如果我们的项目中需要解决幻读的话也有两个办法：

使用串行化读的隔离级别
MVCC+next-key locks：next-key locks由record locks(索引加锁) 和 gap locks(间隙锁，每次锁住的不光是需要使用的数据，还会锁住这些数据附近的数据)
实际上很多的项目中是不会使用到上面的两种方法的，串行化读的性能太差，而且其实幻读很多时候是我们完全可以接受的。
