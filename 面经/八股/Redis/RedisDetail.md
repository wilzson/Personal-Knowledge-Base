# Redis 有序集合zset

只有当长度和排序集的元素个数，同时满足以下两个条件时使用ziplist（listpack）作为其内部表示，具体条件如下：

1. 元素数量少，集合中的元素数量少于配置的阈值64
2. 插入数据长度：当zset插入的任意一个数据的长度超过64的时候

## 手写跳表 

```java
package skiplist;

/**
 * 跳表的一种实现方法
 * 跳表中存储的是正整数，并且存储的是不重复的
 */

public class SkipList {
    public class Node {

        private int data = -1;

        // 每一个节点，都带着一个指针数组，最大16层的一个跳表结构，多级索引结构
        private Node forwards[] = new Node[MAX_LEVEL];

        private int maxLevel = 0; // 该节点的层数

        @Override
        public String toString() {
            StringBuilder builder = new StringBuilder();
            builder.append("{data:");
            builder.append(data);
            builder.append("; levels: ");
            builder.append(maxLevel);
            builder.append(" }");
            return builder.toString();
        }
    }
    private static final int MAX_LEVEL = 16;
    private int levelCount = 1;

    private Node head = new Node();
    // 定义好跳表节点


    public void insert(int value) {
        int level = randomLevel();  // 首先生成一个随机的层数
        Node newNode = new Node();
        newNode.data = value;
        newNode.maxLevel = level;
        Node update[] = new Node[level];
        for(int i = 0; i < level; ++i) {
            update[i] = head;
        }

        //1. 记录每一层的前驱节点
        Node p = head;
        // 从最高层开始往下数
        for(int i = level - 1; i >= 0; --i) {
            while(p.forwards[i] != null && p.forwards[i].data < value) {
                p = p.forwards[i];
            }
            update[i] = p; // 记录前驱节点
        }
        // 每一层的前驱节点，插入当前节点
        for(int i = 0; i < level; i++) {
            newNode.forwards[i] = update[i].forwards[i];
            update[i].forwards[i] = newNode;
        }
        //更新节点深度
        if(levelCount < level)
            levelCount = level;
    }

    // 跳表查询
    public Node find(int value) {
        Node p = head;
        // 从上层开始，一层一层往下找
        for(int i = levelCount - 1; i >= 0; i--) {
            while(p.forwards[i] != null && p.forwards[i].data < value) {
                p = p.forwards[i];
            }
        }
        // 如果当前节点的下一个节点为目标节点，
        if(p.forwards[0] != null && p.forwards[0].data == value) {
            return p.forwards[0];
        }else {
            return null;
        }
    }


    // 理论来讲，一级索引中元素个数应该占原始数据的 50%，二级索引中元素个数占 25%，三级索引12.5% ，一直到最顶层。
    // 因为这里每一层的晋升概率是 50%。对于每一个新插入的节点，都需要调用 randomLevel 生成一个合理的层数。
    // 该 randomLevel 方法会随机生成 1~MAX_LEVEL 之间的数，且 ：
    //        50%的概率返回 1
    //        25%的概率返回 2
    //      12.5%的概率返回 3 ...
    private int randomLevel() {
        int level = 1;

        while (Math.random() < SKIPLIST_P && level < MAX_LEVEL)
            level += 1;
        return level;
    }

}

```







# 集群的session共享问题

多台Tomcat并不共享session存储空间，当请求切换到不同tomcat会导致数据丢失的问题

所以要采用redis来替换session实现分布式需求。

基于redis实现共享session登录

# Token的引入：
Token是在客户端频繁向服务端请求数据，服务端频繁的去数据库查询用户名和密码并进行对比，判断用户名和密码正确与否，并作出相应提示，在这样的背景下，Token便应运而生。

## Token的定义
Token是服务端生成的一串字符串，以作客户端进行请求的一个令牌，当第一次登录后，服务器生成一个Token便将此Token返回给客户端，以后客户端只需带上这个Token前来请求数据即可，无需再次带上用户名和密码。

## Token的目的
Token的目的是为了减轻服务器的压力，减少频繁的查询数据库，使服务器更加健壮。

## 如何使用Token
### 用mac地址作为Token
客户端：客户端在登录的时候获取mac地址，并作为参数传递给服务端。

服务端：接收到该参数后，用一个变量来接收并同时将其作为Token保存在数据库中，并将该Token设置到session中客户端每次请求的时候都要统一拦截，并将客户端传递的token和服务器端session中的token进行对比，如果相同则放行，不同则拒绝。

### 用session值作为Token

# 缓存更新策略

先操作数据库，再删除缓存的方案
确保数据库和缓存操作的原子性

# 缓存穿透
是指客户端请求的数据在缓存中和数据库中都不存在，这样缓存永远不会生效，这些请求都会打到数据库中，大量的请求可能会导致数据库崩溃
常见解决方法：
- 缓存空对象
- 布隆过滤器
经典加一层，优点：内存占用较少，没有多余key
缺点：实现复杂，存在误判可能（Redis里面自带布隆过滤器）

# 缓存雪崩
同一时间段内大量的缓存key同时失效或者redis服务宕机，导致大量请求达到数据库

# 缓存击穿
一个被高并发访问并且缓存重建业务较复杂的key突然失效了，无数的请求访问会在瞬间在数据库带来巨大的冲击。
- 互斥锁
缺点：等待时间过长，只有一个线程可以抢占资源，其他线程都要等待
- 逻辑过期


# 优惠券秒杀
## 超卖问题
典型的多线程安全问题，常见的解决问题就是锁

- 悲观锁
  认为线程安全问题一定发生，因此在操作数据之前先获取锁，确保线程串行执行。
- 乐观锁（应用于读多写少的情况）
  认为线程安全问题不一定会发生，因此不加锁，只是在更新数据是去判断有没有其他线程对数据做了修改
  - 如果没有修改则认为是安全的，自己才更新数据
  - 如果已经被其他线程修改说明发生了安全问题，此时可以重试或异常

乐观锁
- 版本号法，加多了一个字段version，在进行更新操作的时候，判断version是否为对应的数字
- CAS法， 先对比，对比没有错误，再进行更改



# Redis分布锁

分布式锁：满足分布式系统或集群模式下**多进程可见**并且**互斥**的锁

分布式锁需要满足的条件：多进程可见、满足互斥、高可用、高并发、安全性

## 获取锁

- 互斥：确保只有一个线程获取锁
- 

setnx "key" thread1

expire "key" 5 # 添加一个超时时间

## 释放锁

- 手动释放
- 超时释放：获取锁是添加一个超时时间

del "key"

## 保证分布式锁的原子性

要保证分布式锁的原子性，即获取锁和释放锁的一致性。

Redis提供了Lua脚本功能，在一个脚本中编写多条Redis命令，确保多条命令执行时的原子性

#执行redis命令

redis.call('命令名称'，’key‘，'其他参数'....)

在redis执行lua命令，要使用EVAL关键字

```sql
EVAL "return redis.call('set', 'name', 'Jack')" 0
// Lua脚本中索引以1开头，后面放对应参数
EVAL "return redis.call('set', KEYS[1], ARGV[1])" 1 name Rose 
```



## Redis 分布式锁误删操作

```sql
# 添加锁， NX是互斥，EX是设置超时时间
SET lock thread1 NX EX 10
```



1. 因为业务1阻塞导致超时释放锁，业务2抢占锁执行业务，业务1执行完后直接del key，导致业务2锁被误删

- 改进Redis的分布式锁
  1. 获取锁时存入线程标示（可以用UUID标示）
  2. 在释放锁时先获取锁中线程标示，判断是否与当前线程标示一致，然后再释放锁。

1. 因为业务1的锁判断和释放锁是分开动作，所以会导致误删操作

- 改进方法：

  利用Redis的Lua脚本，在一个脚本中编写多条Redis命令，确保多条命令执行的原子性。









## 基于Redis的分布式锁优化

1. 锁不可重入 
2. 不可重试：获取锁只尝试一次就结束锁的获取（watch dog延续时长）
3. 超时释放优化
4. 主从复制

使用Redission分布式框架来实现多种优化






# 优惠券秒杀

## 超卖问题

典型的多线程安全问题，常见的解决问题就是锁

- 悲观锁
  认为线程安全问题一定发生，因此在操作数据之前先获取锁，确保线程串行执行。
- 乐观锁（应用于读多写少的情况）
  认为线程安全问题不一定会发生，因此不加锁，只是在更新数据是去判断有没有其他线程对数据做了修改
  - 如果没有修改则认为是安全的，自己才更新数据
  - 如果已经被其他线程修改说明发生了安全问题，此时可以重试或异常

乐观锁

- 版本号法，加多了一个字段version，在进行更新操作的时候，判断version是否为对应的数字
- CAS法， 先对比，对比没有错误，再进行更改





# 消息队列

## Redis消息队列实现异步秒杀

最简单消息队列模型包含3个角色：消息队列、生产者(发送消息到消息队列)、消费者(从消息队列获取信息并处理信息)



# BitMap实现用户签到

二进制哈希表

Redis中利用string类型数据结构实现bitmap，因此最大上限是512M，转换为bit则是2^32个bit位。

```sql
SETBIT:
GETBIT:
BITCOUNT:统计bitmap中值为1的bit位的数量
BITFIELD:（查询、修改）bitmap中bit数组中的指定位置的值
BITFIELD_RO:获取bitmap中bit数组，并以十进制形式返回
BITOP：将多个BitMap的结果做位运算
BITPOS：查找bit数组中指定范围内第一个0/1出现的位置
```





# 分布式缓存

用于集群redis的实现

单点Redis的问题

1. 数据丢失问题：内存存储，服务重启可能会导致丢失数据（解决方案：利用AOF、RDB实现持久化）
2. 并发能力问题：单点Redis无法满足高并发场景（解决方案：实现主从集群，读写分离）
3. 存储能力问题：基于内存存储，存储空间不足（解决方案：搭建分片集群，利用插槽机制实现动态扩容）
4. 故障恢复问题：如果Redis宕机，则服务不可用，需要一种自动的故障恢复方案（解决方案：利用Redis哨兵，实现健康检测和自动恢复）

## Redis持久化

RDB：Redis Database backup file (redis数据备份文件)。构建快照。当Redis实例故障重启后，从磁盘读取快照文件。

```sql
127.0.0.1:6379 > save # 由Redis主进程来执行RDB，会阻塞所有命令 (不推荐该方法)

127.0.0.1:6379 > bgsave # 开启子进程来执行RDB，避免主进程受到影响
bgsave开始时会fork主进程得到子进程，子进程共享主进程的内存数据，完成fork后读取内容数据并写入RDB文件
fork采用的写时复制的技术
读时共享，写时复制
```

Redis关机前，会主动执行一次RDB



AOF： Append Only File（追加文件）。Redis处理的每一个**写命令**都会记录在AOF文件，可以看作是命令日志文件。

```sql
AOF的命令记录频率
appendfsync always # 每执行一次写命令，立即记录到AOF命令

appendfsync everysec # 写命令执行完先放入AOF缓冲区，然后每隔一秒将缓冲区数据写到AOF文件，默认方案
```

AOF文件比RDB文件大很多（因为AOF文件记录写命令，存在大量冗余数据）

执行 **bgrewriteaof** 命令，可以让aof命令执行重写功能。

公司企业里面主要为 RDB + AOF 混合使用， RDB在 save 的过程，使用 AOF 记录增量。



## Redis主从复制

 Redis主从复制原理：

主从第一次同步是全量同步

判断是否同步：

1. 从节点发送 replid 
2. 主节点判断请求replid是否一致
3. 是第一次，则返回主节点 replid 和 offset

主节点RDB文件发送给从节点（RDB + repl_backlog（记录RDB期间的所有命令））

从节点清空本地数据，加载RDB文件，再执行repl_backlog 



### 简述全量同步的流程

1. salve节点请求增量同步
2. master节点判断replid，发现不一致，拒绝增量同步
3. master将完整内存数据生成RDB，发送RDB到slave
4. slave清空本地数据，加载master的RDB
5. master将RDB期间的命令记录再repl_backlog，并持续将log中的命令发送给slave
6. slave执行接收到的命令，保持与master之间的同步

### 简述增量同步的流程

如果slave重启后同步，则执行增强同步

repl_backlog大小有上限，如果被覆盖，则进行全量同步

增量同步：slave提交自己的offset到master，master获取repl_baklog中从offset之后的命令给slave



## Redis哨兵机制

哨兵机制（sentinel）用来实现主从集群的自动故障恢复。（多机热备）
