## 事务

事务是一组操作的集合，它是一个不可分割的工作单位，事务会把所有的操作作为一个整体一起向系统提交或撤销操作请求，即这些操作要么同时成功，要么同时失败。

- 原子性（Atomicity)：事务是不可分割的最小操作单元，要么全部成功，要么全部失败。
- 一致性（Consistency)：事务完成时，必须使所有的数据都保持一致状态。
- 隔离性(Isolation)：数据库系统提供的隔离机制，保证事务在不受外部并发操作影响的独立环境下运行。
- 持久性(Durability）：事务一旦提交或回滚，它对数据库中的数据的改变就是永久的。

```sql
CREATE TABLE account( 
id int auto_increment primary key comment 'ID',
name varchar(10) comment'姓名',
money int comment'余额') comment'账户表';

INSERT INTO account(id,name, money) VALUES (nuLl,'张三',2000),(nuLl, '李四',2000);

-- 查看事务的提交方式
SELECT @@autocommit;
-- 设置事务的提交方式
SET @@autocommit=0;

-- 手动开启事务
START TRANSACTION;
SELECT * FROM account WHERE name = '张三';
UPDATE account SET money = money - 1000 WHERE name = '张三';
UPDATE account SET money = money + 1000 WHERE name = '李四';

-- 提交事务
COMMIT;
-- 回滚事务
ROLLBACK;
```

### 迸发事务问题

[迸发事务问题演示](https://www.bilibili.com/video/BV1Kr4y1i7ru?p=55&vd_source=590cb3e34e1ca2883a1019e924794619)

- 脏读: 一个事务读到另外一个事务还没有提交的数据。
- 不可重复读: 一个事务先后读取同一条记录，但两次读取的数据不同，称之为不可重复读。
- 幻读 一个事务按照条件查询数据时，没有对应的数据行，但是在插入数据时，又发现这行数据已经存在，好像出现了” 幻影”。

隔离级别
- Read uncommitted
- Read committed
- Repeatable Read(默认)
- Serializable

```sql
-- 查询隔离级别
SELECT @@TRANSACTION_ISOLATION;

-- 设置事务隔离级别 
-- SET [SESSION | GLOBAL] TRANSACTION ISOLATION LEVEL [ READ UNCOMMITTED | READ COMMITTED | REPEATABLE READ | SERIALIZABLE ]
SET SESSION TRANSACTION ISOLATION LEVEL SERIALIZABLE;
```