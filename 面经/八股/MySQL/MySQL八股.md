# MySQL

## B+tree和跳表

相同点：都是多级索引+链表的结果

不同点：IO操作的单位不同，B+Tree是page(16k)，跳表是node节点，一个node几十个字节，读取的IO次数也是不一样。

## 增删改查的语句

```sql
select * from customerinfo where id = xxx [order by column_name asc | desc] limit 10;
update table_name set column1 = value1, column2 = value2 where id = xxx,
insert into table_name(coulumn1, coulumn2, column3) values (value1, value2, value3);
delete from table_name where condition;
```



## 主键索引和辅助索引

主键索引为聚集索引，数据库走主键索引的时候，根据id就能获取对应的数据信息

辅助索引也叫非聚集索引，辅助索引先找到主键，再根据主键索引获取数据。相当于辅助索引需要走两次索引

### 覆盖索引

在二级索引的B+Tree就能查询到结果的过程就叫做**覆盖索引**。

使用InnoDB存储引擎，可以通过辅助索引覆盖索引，直接获取查询记录，无需查询聚集索引中的记录。

使用覆盖索引可以减少大量的IO操作



### 联合索引

联合索引是指对表上的多个列进行索引。

联合索引的第二个好处是对第二个键值已经做了排序

https://www.php.cn/faq/550296.html



## 在数据库里面， where、group by、order by、limit的执行顺序

首先是where过滤数据，然后是group by进行数据分组，接下来是order by进行排序，最后是limit限制返回的记录数量。



## 1. 选取最适用的字段属性

ENUM类型被当作数值型数据来处理，而数值型数据被处理起来的速度比文本类型快的多。

## 2. 使用连接JOIN来代替子查询

子查询

```sql
select * from customerinfo where customerid 
not in (select curstomerid from salesinfo)
```

如果使用连接来完成查询工作，速度会快很多

```sql
select * from customerinfo
left join salesinfo on customerinfo.customerid = salesinfo.customerid
where salesinfo.customerid is null
```

使用子查询会在内存中创建临时表来保存子表，较耗时

## 使用联合(UNION)来代替手动创建的临时表





## 事务

当同时把某个数据插入两个相关联的表中，为保持数据库中数据的一致性和完整性。要使用事务。作用为要么语句块中每条语句都操作成功，要么都失败。

```sql
begin;
	insert into salesinfo set customerid = 14;
	update inventory set quantity = 11 where item = 'book';
commit;
```

事务的另一个重要作用是当**多个用户同时使用相同的数据源**时，它可以利用**锁定数据库**的方法来为用户提供一种安全的访问方式，这样可以保证用户的操作不被其它的用户所干扰。



## 锁定表

事务由于其独占性，在事务执行的过程中，数据库将会被锁定，因此其他用户请求只能暂时等待直到该事务结束。

```sql
lock table invertory write select quantity from inventory where item = 'book';
...
update invertory set quantity=11 where item='book';unlocktables;
```



## 建立索引

索引可以提高数据库性能，尤其是在查询语句当中包含有max(),min(),orderby这些命令的时候。

一般来说，索引应建立在那些使用**join**, **where**判断, **orderby**排序的字段上，尽量不要对数据库中某个含有大量重复的值的字段建立索引。如ENUM类型。



## 优化的查询语句

- 最好在相同类型的字段间进行比较。CHAR字段和VARCHAR字段大小相同的时候，可以比较

- 在建有索引的字段上尽量不要使用函数进行操作。

- 尽量减少通配符的使用

  ```sql
  select * from books where name like "MySQL%"
  
  select * from books where name >= "MySQL" and name < "MySQM"//方法2更快
  ```

  

# 优化MySQL数据库的八种方法

## 1. 创建索引

以查询占主要的应用来说，索引很重要。不加索引需要对全表扫描

## 2. 复合索引

如select * from users where area = 'beijing' and age = 22.

最佳左前缀特性。因此在创建复合索引的时候，应该将最常用作限制条件的列放在最左边，依次递减。

## 3. 索引不会包含有null的列

只要列中包含有NULL值都将不会被包含在索引中，复合索引中只要有一列含有NULL值，那么这一列对于此复合索引就是无效的。所以我们在数据库设计时不要让字段的默认值为NULL。

## 4. 使用短索引

对串列进行索引，如果可能应该指定一个前缀长度。例如，如果有一个CHAR(255)的 列，如果在前10 个或20 个字符内，多数值是惟一的，那么就不要对整个列进行索引。短索引不仅可以提高查询速度而且可以节省磁盘空间和I/O操作。

## 5. 排序的索引

mysql查询只使用一个索引，因此如果where子句中已经使用了索引的话，那么order by中的列是不会使用索引的。因此数据库默认排序可以符合要求的情况下不要使用排序操作；尽量不要包含多个列的排序，如果需要最好给这些列创建复合索引。

## 6. 尽量不使用like

## 7. 不要在列上进行运算

## 不使用not in和<>操作

NOT IN和<>操作都不会使用索引将进行全表扫描。NOT IN可以NOT EXISTS代替，id<>3则可使用id>3 or id<3来代替。

Exists可以使用索引，并且只要查到一行数据满足就会终止查询。

# MySQL常用的两种存储引擎

- 一个是MyISAM，不支持事务处理，读性能处理快，**表级别锁**
- 另一个InnoDB。支持事务处理(ACID)，设计目标是为处理大容量数据发挥最大化性能，**行级别锁**。



## 索引

常见的索引结果有B树、B+树、Hash、红黑树。

在MySQL中使用了B+树作为索引结构

## B树和B+树的区别

1. B树所有节点及存放键(key)，也存放数据（data），B+树只有叶子节点存放key和data，其他内节点只存放key
2. B+树的叶子节点，有一条链表串起来
3. B+树更适合范围查询



## MySQL的事务隔离级别详解

- 读取未提交，允许读取尚未提交的数据变更，会导致脏读、幻读或不可重复读。
- 读取已提交，允许读取并发事务已经提交的数据。可以阻止脏读，但是幻读或不可重复读仍有可能发生。
- 可重复读。对同一字段的多次读取结果是一致的。可以阻止脏读和不可重复读，但幻读仍有可能发生。
- 可串行读。事务各自单独处理，隔离程度最高

## 脏读、幻读、不可重复读

- 脏读就是指当一个事务正在访问数据，并且对数据进行了修改，而这种修改还没有提交到数据库中，这时，另外一个事务也访问这个数据，然后使用了这个数据。
- 不可重复读：是指在一个事务内，多次读同一数据。在这个事务还没有结束时，另外一个事务也访问该同一数据。那么，在第一个事务中的两次读数据之间，由于第二个事务的修改，那么第一个事务两次读到的的数据可能是不一样的。这样就发生了在一个事务内两次读到的数据是不一样的，因此称为是不可重复读。
- 幻读就是一个事务里原本没有读到的数据，另外一个事务插入了这条数据，再原事务想要插入的时候，发现这条数据又已经存在了。

# MySQL三大日志

## binlog 

binlog是逻辑日志，记录内容是语句的原始逻辑，只要发生了表数据更新，都会产生binlog日志。

`binlog`会记录所有涉及更新数据的逻辑操作，并且是顺序写。

两种方式：

1. row(指定具体的操作数据)， 
2. statement(简单的SQL语句)

写入机制

1. 事务执行时，先将日志写到binlog cache，事务提交的时候，再把binlong cache提交到binlong文件中



## 回表查询

指的是当MySQL数据库引擎在通过索引定位到数据行后，发现需要访问表中的其他列数据，而不是直接通过索引就能获取到所需的数据。

这种情况通常发生在查询语句中包含了索引无法覆盖的字段或者设计到了复杂的查询条件时。

### 怎样避免回表查询

1. 覆盖索引。确保查询所需的列都包含在一个索引中。如果你的查询只涉及索引中的列，而不需要回表获取其他列的值，就可以避免回表查询。

   ```sql
   create index idx_employed_id on employees(name, employee_id);
   ```

2. 尽量使用覆盖索引来满足查询的需求。意味着查询中**条件**和**选择列表中的列**都应该时索引的一部分，这样数据库引擎可以直接从索引中获取所需的信息。

3. 明确列出查询中所需的列，而不是使用select * 。这可以避免不必要的列被检索，从而减少回表的需求。



## Explain来查看查询语句

```sql
explain select * from student where age > 10 and age < 15
```



## 当表中有主键索引(id)和普通索引(name)

```sql
select id from test where id > 1 and name like 'i%';
```

这条查询语句的结果既可以使用主键索引，也可以使用普通索引，但是执行的效率会不同。这时，就需要优化器来决定使用哪个索引了。

很显然这条查询语句是**覆盖索引**，直接在二级索引就能查找到结果（因为二级索引的 B+ 树的叶子节点的数据存储的是主键值），就没必要在主键索引查找了，因为查询主键索引的 B+ 树的成本会比查询二级索引的 B+ 的成本大，优化器基于查询成本的考虑，会选择查询代价小的普通索引。



# MVCC（多版本并发控制）

## 读操作(select)

会使用快照读取，事务读取的是快照数据，因此不会对其他并发事务对数据行的修改产生影响

## 写操作

当一个事务执行写操作时，它会生成一个新的数据版本，并将修改后的数据写入数据库。具体工作情况如下：

- 对于写操作，事务会为要修改的数据行创建一个新的版本，并将修改后的数据写入新版本。
- 新版本的数据会带有当前事务的版本号，以便其他事务能够正确读取相应版本的数据。
- 原始版本的数据仍然存在，供其他事务使用快照读取，这保证了其他事务不受当前事务的写操作影响。



