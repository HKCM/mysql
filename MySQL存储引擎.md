# 存储引擎

<!-- TOC -->
* [存储引擎](#存储引擎)
  * [MySQL体系结构](#mysql体系结构)
  * [存储引擎简介](#存储引擎简介)
  * [存储引擎特点](#存储引擎特点)
    * [InnoDB介绍](#innodb介绍)
    * [MyISAM](#myisam)
    * [Memory](#memory)
  * [存储引擎选择](#存储引擎选择)
  * [InnoDB引擎](#innodb引擎)
    * [逻辑存储结构](#逻辑存储结构)
      * [内存结构](#内存结构)
        * [BufferPool](#bufferpool)
        * [Change Buffer](#change-buffer)
        * [Adaptive Hash Index](#adaptive-hash-index)
        * [Log Buffer](#log-buffer)
      * [磁盘结构](#磁盘结构)
        * [System Tablespace](#system-tablespace)
        * [File-Per-Table Tablespaces](#file-per-table-tablespaces)
        * [General Tablespaces](#general-tablespaces)
        * [Undo Tablespaces](#undo-tablespaces)
        * [Temporary Tablespaces](#temporary-tablespaces)
        * [Doublewrite Buffer Files](#doublewrite-buffer-files)
        * [Redo Log](#redo-log)
      * [后台线程](#后台线程)
    * [事务原理](#事务原理)
      * [事务](#事务)
      * [特性](#特性)
      * [redo log](#redo-log-1)
      * [undo Log](#undo-log)
    * [MVCC实现原理](#mvcc实现原理)
      * [MVCC介绍](#mvcc介绍)
      * [当前读](#当前读)
      * [快照读](#快照读)
        * [记录中的隐藏字段](#记录中的隐藏字段)
        * [undo log日志](#undo-log日志)
        * [ReadView](#readview)
<!-- TOC -->

## MySQL体系结构

- 连接层: 最上层是一些客户端和链接服务，主要完成一些类似于连接处理、授权认证、及相关的安全方案。服务器也会为安全接入的每个客户端验证它所具有的操作权限。
- 服务层: 第二层架构主要完成大多数的核心服务功能，如SQL接口，并完成缓存的查询，SQL的分析和优化，部分内置函数的执行。所有跨存储引擎的功能也在这一层实现，如过程、函数等。
- 引擎层: 存储引擎真正的负责了MySQL中数据的存储和提取，服务器通过API和存储引擎进行通信。不同的存储引擎具有不同的功能，这样我们可以根据自己的需要，来选取合适的存储引擎。 (索引是在引擎层)
- 存储层: 主要是将数据存储在文件系统之上，并完成与存储引擎的交互。

连接层: Authentication,Thread Reuse,Connection Limits,Check Memory, Caches
服务层: SQL接口, 解析器, 查询优化器, 缓存
引擎层: InnoDB,MyISAM, NDB, Archive, Federated, Memory, Merge, Partner, Community, Custom
存储层: 系统文件,存储文件,日志文件

## 存储引擎简介

存储引擎就是存储数据、建立索引、更新/查询数据等技术的实现方式。**存储引擎是基于表的，而不是基于库的**，所以存储引擎也可被称为表类型。

```sql
-- 查询当前所支持的存储引擎
SHOW engines;

-- test.my_myisam definition
CREATE TABLE `my_myisam` (
  `id` int DEFAULT NULL,
  `name` varchar(10) DEFAULT NULL
) ENGINE=MyISAM DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- test.my_memory definition
CREATE TABLE `my_memory` (
  `id` int DEFAULT NULL,
  `name` varchar(10) DEFAULT NULL
) ENGINE=Memory DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;
```

## 存储引擎特点

### InnoDB介绍

InnoDB：是一种兼顾高可靠性和高性能的通用存储引擎，在MySQL5.5之后，InnoDB是默认的MySQL存储引擎。

特点:
- DML操作遵循ACID模型，**支持事务**；
- **行级锁**，提高并发访问性能；
- **支持外键**FOREIGN KEY约束，保证数据的完整性和正确性；

文件:
XXX.ibd：XXX代表的是表名，InnoDB引擎的每张表都会对应这样一个表空间文件，存储该表的表结构(frm、sdi)、数据和索引。

参数：innodb_file_per_table

表结构
- TableSpece:表空间
- Segment:段
- Extent:区 1M (一个区有64页)
- Page:页 16K
- ROW:行

```sql
-- 表文件
SHOW VARIABLES LIKE '%innodb_file_per_table%';
-- 数据存储位置
SHOW VARIABLES LIKE '%datadir%';
```

### MyISAM

MyISAM：是MySQL早期的默认存储引擎。

特点:
- 不支持事务，不支持外键
- 支持表锁，不支持行锁
- 访问速度快

文件:
- .MYD: 表数据
- .MYI: 索引
- .sdi: 表结构

### Memory

Memory引擎的表数据是存储在内存中的，由于受到硬件问题、或断电问题的影响，只能将这些表作为临时表或缓存使用。

特点:
- 内存存放
- hash索引（默认）

文件: .sdi：存储表结构信息

## 存储引擎选择

InnoDB：是Mysql的默认存储引擎，支持事务、外键。如果应用对事务的完整性有比较高的要求，在并发条件下要求数据的一致性,数据操作除了插入和查询之外，还包含很多的更新、删除操作，那么InnoDB存储引擎是比较合适的选择。

MyISAM：如果应用是以读操作和插入操作为主，只有很少的更新和删除操作，并且对事务的完整性、并发性要求不是很高，那么选择这个存储引擎是非常合适的。 (日志,评论)(MongoDB)

MEMORY：将所有数据保存在内存中，访问速度快，通常用于临时表及缓存。MEMORYE的缺陷就是对表的大小有限制，太大的表无法缓存在内存中，而且无法保障数据的安全性。(Redis)

## InnoDB引擎

### 逻辑存储结构

表结构:
- TableSpece(表空间): 表空间ibd文件，一个mysql实例可以对应多个表空间，用于存储记录、索引等数据。
- Segment(段):分为数据段(Leaf node segment)、索引段(Non-leaf node segment)、回滚段(Rollback segment),InnoDB 是索引组织表,数据段就是B+树的叶子节点,索引段即为B+树的非叶子节点.段用来管理多个Extent(区).
- Extent(区): 表空间的单元结构，每个区的大小为1M。默认情况下，InnoDB存储引擎页大小为16K，即一个区钟一共有64个连续的页。
- Page(页):InnoDB存储引擎磁盘管理的最小单元，每个页的大小默认为16KB。为了保证页的连续性，InnoDB存储引擎每次从磁盘申请4-5个区。
- ROW(行): InnoDB存储引擎数据是按行进行存放的。

#### 内存结构

##### BufferPool

BufferPool：缓冲池是主内存中的一个区域，里面可以缓存磁盘上经常操作的真实数据，在执行增删改查操作时，先操作缓冲池中的数据（若缓冲池没有数据，则从磁盘加载并缓存），然后再以一定频率刷新到磁盘，从而减少磁盘O，加快处理速度。

缓冲池以Page页为单位，底层采用链表数据结构管理Page。根据状态，将Page分为三种类型：
- free page：空闲page，未被使用。
- clean page：被使用page，数据没有被修改过。
- dirty page：脏页，被使用page，数据被修改过，也中数据与磁盘的数据产生了不一致。

##### Change Buffer

Change Buffer：更改缓冲区（针对于非唯一二级索引页），在执行DML语句时，如果这些数据Page 没有在Buffer Pool中，不会直接操作磁盘，而会将数据变更存在更改缓冲区Change Buffer中，在未来数据被读取时，再将数据合并恢复到Buffer Pool中，再将合并后的数据刷新到磁盘中。

##### Adaptive Hash Index

Adaptive Hash Index：自适应hash索引，用于优化对Buffer Pool数据的查询。InnoDB存储引擎会监控对表上各索引页的查询，如果观察到hash索引可以提升速度，则建立hash索引，称之为自适应hash 索引。

**自适应哈希索引，无需人工干预，是系统根据情况自动完成**

参数：adaptive，_hash_index

```sql
show variables like '%hash_index%';
```

##### Log Buffer

Log Buffer：日志缓冲区，用来保存要写入到磁盘中的log日志数据(redo log、undo log)，默认大小为16MB，日志缓冲区的日志会定期刷新到磁盘中。如果需要更新、插入或删除许多行的事务，增加日志缓冲区的大小可以节省磁盘/O。

参数：
- innodb_log_buffer_size：缓冲区大小
- innodb_flush_log_at_trx_commit：日志刷新到磁盘时机

```sql
show variables like '%log_buffer_size%';
show variables like '%flush_log%';
-- innodb_flush_log_at_trx_commit	1

-- 0：每秒将日志写入并刷新到磁盘一次。
-- 1：日志在每次事务提交时写入并刷新到磁盘 
-- 2：日志在每次事务提交后写入，并每秒刷新到磁盘一次。
```

#### 磁盘结构

##### System Tablespace

System Tablespace：系统表空间是更改缓冲区的存储区域。如果表是在系统表空间而不是每个表文件或通用表空间中创建的，它也可能包含表和索引数据。（在MySQL5.×版本中还包含InnoDB数据字典、undo log等)

参数：innodb_data_file_path

```sql
show variables like '%data_file_path%';
```

##### File-Per-Table Tablespaces

File-Per-Table Tablespaces:每个表的文件表空间包含单个InnoDB表的数据和索引,并存储在文件系统上的单个数据文件中.

参数:innodb_file_per_table

```sql
show variables like '%file_per_table%';
```

##### General Tablespaces

General Tablespaces：通用表空间，需要通过CREATE TABLESPACE语法创建通用表空间，在创建表时，可以指定该表空间。

```sql
-- 创建表空间
create tablespace itheima add datafile 'itheima.ibd' engine = innodb;
-- 创建数据表时指定表空间
create table a(id int primary key auto_increment, name varchar(50)) engine = innodb tablespace itheima;
```

##### Undo Tablespaces

Undo Tablespaces：撤销表空间，MySQL实例在初始化时会自动创建两个默认的undo表空间（初始大小 16M)，用于存储undo log日志。

##### Temporary Tablespaces

Temporary Tablespaces：InnoDB使用会话临时表空间和全局临时表空间。存储用户创建的临时表等数据。

##### Doublewrite Buffer Files

Doublewrite Buffer Files：双写缓冲区，innoDB引擎将数据页从Buffer Pooll刷新到磁盘前，先将数据页写入双写缓冲区文件中，便于系统异常时恢复数据。`ib_16384_0.dblwr`和`ib_16384_1.dblwr`

##### Redo Log

Redo Log：重做日志，是用来实现事务的持久性。该日志文件由两部分组成：重做日志缓冲 (redo log buffer)以及重做日志文件(redo log)，前者是在内存中，后者在磁盘中。当事务提交之后会把所有修改信息都会存到该日志中，用于在刷新脏页到磁盘时，发生错误时，进行数据恢复使用。

以循环方式写入重做日志文件，涉及两个文件`ib_logfile0`和`ib_logfile1`

#### 后台线程

1. Master Thread

核心后台线程，负责调度其他线程，还负责将缓冲池中的数据异步刷新到磁盘中，保持数据的一致性， 还包括脏页的刷新、合并插入缓存、udo页的回收。

2. IO Thread

在InnoDB存储引擎中大量使用了AIO来处理IO请求，这样可以极大地提高数据库的性能，而IO Thread主要负责这些IO请求的回调。
- Read thread,4,负责读操作
- Write thread,4,负责写操作
- Log thread,1,负责将日志缓冲区刷新到磁盘
- Insert buffer thread,1,负责将写缓冲区内容刷新到磁盘

```sql
show engine innodb status;
```

3. Purge Thread

主要用于回收事务已经提交了的undo log，在事务提交之后，undo log可能不用了，就用它来回收。

4. Page Cleaner Thread

协助Master Thread刷新脏页到磁盘的线程,它可以减轻Master Thread的工作压力,减少阻塞.

### 事务原理

#### 事务
事务是一组操作的集合，它是一个不可分割的工作单位，事务会把所有的操作作为一个整体一起向系统提交或撤销操作请求，即这些操作要么同时成功，要么同时炎败。

#### 特性

- 原子性(Atomicity)：事务是不可分割的最小操作单元，要么全部成功，要么全部失败。
- 一致性(Consistency)：事务完成时，必须使所有的数据都保持一致状态。
- 隔离性(Isolation)：数据库系统提供的隔离机制，保证事务在不受外部并发操作影响的独立环境下运行。
- 持久性(Durability)：事务一旦提交或回滚，它对数据库中的数据的改变就是永久的。

通过`redo log`和`undo log`保证了原子性,一致性和持久性,通过`锁`和`MVCC`保证隔离性

#### redo log

重做日志，记录的是事务提交时数据页的物理修改，是用来实现事务的持久性。

该日志文件由两部分组成：重做日志缓冲(redo log buffer)以及重做日志文件(redo log file)，前者是在内存中，后者在磁盘中。当事务提交之后会把所有修改信息都存到该日志文件中，用于在刷新脏页到磁盘，发生错误时，进行数据恢复使用。

原理:
1. 在一个事务中，执行多个增删改的操作时，InnoDB引擎会先操作缓冲池中的数据
2. 如果缓冲区没有对应的数据，会通过后台线程将磁盘中的数据加载出来，存放在缓冲区中
3. 然后将缓冲池中的数据修改，修改后的数据页我们称为脏页(缓冲池中的数据与磁盘中的数据不一致)
4. 脏页则会在一定的时机，通过后台线程刷新到磁盘中，从而保证缓冲区与磁盘的数据一致
5. 缓冲区的脏页数据并不是实时刷新的，而是一段时间之后将缓冲区的数据刷新到磁盘中
6. 刷新到磁盘的过程出错了，而提示给用户事务提交成功，而数据却没有持久化下来，这就出现问题了

redo log作用:
1. 当对缓冲区的数据进行增删改之后，会首先将操作的数据页的变化，记录在redo log buffer中
2. 在事务提交时，会将redo log buffer中的数据刷新到redo log磁盘文件中
3. 如果刷新缓冲区的脏页到磁盘时，发生错误，此时就可以借助于redo log进行数据恢复
4. 如果脏页成功刷新到磁盘或涉及到的数据已经落盘，此时redo log就没有作用了，就可以删除，所以存在的两个redo log文件是循环的。

那为什么每一次提交事务，要刷新redo log 到磁盘中呢，而不是直接将buffer pool中的脏页刷新到磁盘呢 ?

因为在业务操作中，我们操作数据一般都是随机读写磁盘的，而不是顺序读写磁盘。 而redo log在往磁盘文件中写入数据，由于是日志文件，所以都是顺序写的。顺序写的效率，要远大于随机写。 这种先写日志的方式，称之为 WAL（Write-Ahead Logging）。


#### undo Log

回滚日志，用于记录数据被修改前的信息，作用包含两个：提供回滚和MVCC(多版本并发控制)。

undo log和redo log记录物理日志不一样，它是逻辑日志。可以认为当delete一条记录时，undo log中会记录一条对应的insert记录，反之亦然，当update-一条记录时，它记录一条对应相反的update记录。当执行rollback时，就可以从undo log中的逻辑记录读取到相应的内容并进行回滚。

Undo log销毁：undo log在事务执行时产生，事务提交时，并不会立即删除undo log，因为这些日志可能还用于MVCC。

Undo log存储：undo log采用段的方式进行管理和记录，存放在前面介绍的rollback segment回滚段中，内部包含1024个undo log segment。

### MVCC实现原理

#### MVCC介绍

全称Multi-Version Concurrency Control，多版本并发控制。指维护一个数据的多个版本，使得读写操作没有冲突，快照读为MySQL实现MVCC提供了一个非阻塞读功能。MVCC的具体实现，还需要依赖于数据库记录中的三个隐式字段、undo log日志、ReadView。

- 通过隐式字段事务ID和回滚指针形成undo log版本链
- ReadView是具体的读取规则的体现,RR(可重复度)和RC(读已提交)有不同的ReadView

MVCC配合锁机制最终体现体现就是隔离性

#### 当前读

**读取的是记录的最新版本**，读取时还要保证其他并发事务不能修改当前记录，会对读取的记录进行加锁。对于我们日常的操作，如：`select...lock in share mode`(共享锁)，`select...for update、update、insert、delete`(排他锁)都是一种当前读。

例如同时开启A,B两个事务.
1. 在A事务中读取到id为1的行,姓名为张三
2. 在B事务中修改id为1的行,姓名变为李四
3. 此时无论B事务是否提交,在A事务中的查询得到的值都是张三,因为默认隔离级别是可重复读(快照读)
4. 在B事务提交后,该记录的最新版本在A事务中查不到,所以就不是`当前读`
5. 在A事务中想读最新版本需要`select * form user where id = 1 lock in share mode`

#### 快照读

简单的select(不加锁)就是快照读，快照读，读取的是记录数据的可见版本，有可能是历史数据，不加锁，是非阻塞读。
- Read Committed：每次select，都生成一个快照读。
- Repeatable Read：开启事务后第一个select语句才是快照读的地方。
- Serializable：快照读会退化为当前读。

##### 记录中的隐藏字段

- DB_TRX_ID:最近修改事务D，记录插入这条记录或最后一次修改该记录的事务D。
- DB_ROLL_PTR: 回滚指针，指向这条记录的上一个版本，用于配合undo log，指向上一个版本。
- DB_ROW_ID: 隐藏主键，如果表结构没有指定主键，将会生成该隐藏字段。

通过linux下`idb2sdi table.idb`命令可以看到表的字段

##### undo log日志

回滚日志，在insert、update、delete的时候产生的便于数据回滚的月志。

当insert的时候，产生的undo log日志只在回滚时需要，在事务提交后，可被立即删除。

而update、delete的时候，产生的undo log日志不仅在回滚时需要，在快照读时也需要，不会立即被删除。

**undo log版本链**

不同事务或相同事务对同一条记录进行修改，会导致该记录的undo log生成一条记录版本链表，链表的头部是最新的旧记录，链表尾部是最早的旧记录。v5指向v4, v4指向v3

##### ReadView

ReadView(读视图)是快照读SQL执行时MVCC提取数据的依据，记录并维护系统当前活跃的事务（未提交的）id。

不同的隔离级别,生成ReadView的时机不同:
- READ COMMITTEB:在事务中每一次执行快照读时生成ReadView.
- REPEATABLE READ:仅在事务中第一次执行快照读时生成ReadView,后续复用该ReadView.

ReadView中包含了四个核心字段：
- m_ids: 当前活跃的事务ID集合
- min_trx_id: 最小活跃事务ID
- max_trx_id: 预分配事务ID，当前最大事务ID+1(因为事务ID是自增的)
- creator_trx_id: ReadViewt创建者的事务ID

trx_id：代表是当前事务ID。
1. trx_id = creator_trx_id？可以访问该版本 > 成立，说明数据是当前这个事务更改的。
2. trx_id < min_trx_id？可以访问该版本 > 成立，说明数据已经提交了。
3. trx_id > max_trx_id？不可以访问该版本 > 成立，说明该事务是在ReadView生成后才开启。
4. min_trx_id <= trx_id <= max_trx_id？如果trx_id不在m_ids中是可以访问该版本的 > 成立，说明数据已经提交。
