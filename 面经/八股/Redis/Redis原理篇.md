# 数据结构

## ***动态字符串SDS

定义了5种字符串类型，最常用的以8个比特位的字符串结构体

字符串具备**动态扩容**能力

- 如果新字符串小于1M，则新空间为拓展后字符串长度的两倍 + 1；
- 如果新字符串大于1M，则新空间为拓展后字符串长度 + 1M + 1.称为**内存预分配**

## IntSet 整数集合

IntSet是Redis种set集合的一种实现方式，基于整数数组来实现，并且具备长度可变、有序等特征。

一个整数数组，会自动扩容。

## ***Dict 字典

Dict 由三部分组成，分别为：**哈希表**（DictHashTable）、**哈希节点**（DictEntry）、字典（Dict）

当向Dict添加键值对时，Redis首先根据 key 计算出 hash 值 （h），然后利用 h & sizemask 来计算应该存储到数组中的哪个索引位置。

![image-20240718222922387](C:\Users\wilzsn\AppData\Roaming\Typora\typora-user-images\image-20240718222922387.png)

生成dictht 哈希表，每次插入键值对时，生成一个dictEntry，再计算哈希值插入到对应的数组角标位置。

## ZipList 压缩列表

申请的连续的内存空间。优点：1. 节省内存空间，2. 访问效率快。（有头尾节点信息的连续数组）

如果内存占用较多，申请内存效率较低，容易造成碎片化的内存空间。

![image-20240719124630328](C:\Users\wilzsn\AppData\Roaming\Typora\typora-user-images\image-20240719124630328.png)

![image-20240719124713463](C:\Users\wilzsn\AppData\Roaming\Typora\typora-user-images\image-20240719124713463.png)

## QuickList

是一个双端链表，只不过链表中的每个节点都是一个ziplist

为了避免QuickList中的每个 ZipList 中 entry 过多， Redis 提供了一个配置项：list-max-ziplist-size 来限制。



## ***SkipList 跳表

Redis跳表最高支持32级指针，跨度为 2^31 

## 基本数据类型

### String

- 基本编码方式是 **RAW**，基于简单动态字符串 (SDS) 实现，存储上限为 512 mb
- 如果存储的 SDS 长度小于44字节，则采用 **EMBSTR** 编码，此时 object head 与 SDS 是一段连续空间。申请内存时只需要调用一次内存分配函数，效率更高。
- 如果存储的字符串是整数值，并且大小在LONG _MAX 范围内，则会采用 **INT** 编码：直接将数据保存在RedisObject 的ptr指针位置

![image-20240719130851944](C:\Users\wilzsn\AppData\Roaming\Typora\typora-user-images\image-20240719130851944.png)

![image-20240719131202692](C:\Users\wilzsn\AppData\Roaming\Typora\typora-user-images\image-20240719131202692.png)

（ EMBSTR编码， 连续内存，head 和 SDS 在连续的空间内，刚好可以占用一个内存分片，不会产生碎片情况）

![image-20240719131636957](C:\Users\wilzsn\AppData\Roaming\Typora\typora-user-images\image-20240719131636957.png)



### List 

用 ziplist 和 quickList 来实现

Redis中 list 用 quickList 来实现， ziplist 和 linkedlist 来实现

![image-20240719132636356](C:\Users\wilzsn\AppData\Roaming\Typora\typora-user-images\image-20240719132636356.png)



### Set

有无序、保证元素唯一、求交集、并集、差集

利用 hash 来实现

当存储的所有数据都是整数，并且元素数量不超过set-max-intset-entries时，Set会采用 IntSet 编码。

后来会转换为 hash。

![image-20240719133429019](C:\Users\wilzsn\AppData\Roaming\Typora\typora-user-images\image-20240719133429019.png)



![image-20240719133721336](C:\Users\wilzsn\AppData\Roaming\Typora\typora-user-images\image-20240719133721336.png)



### ZSet

利用 hash + SkipList 来实现

![image-20240719141845531](C:\Users\wilzsn\AppData\Roaming\Typora\typora-user-images\image-20240719141845531.png)

利用Dict 来实现键值存储，键必须唯一特点

利用SkipList 来实现可排序特点



当元素数量不多时，zset会采用 ZipList 结构来节省内存。



### Hash

- Hash结构默认采用ZipList编码，用以节省内存。ZipList中相邻的两个entry分别保存field和value
- 当数据量较大，Hash结构会转为 Dict

# 网络模型

 ## 用户空间和内核空间

Linux 系统为了提高IO效率，会在用户空间和内核空间都加入缓冲区

- 写数据时，要把用户缓冲数据拷贝到内核缓冲区，然后写入设备
- 读数据时，要从设备读取数据到内核缓冲区，然后再拷贝到用户缓冲区

![image-20240719155342839](C:\Users\wilzsn\AppData\Roaming\Typora\typora-user-images\image-20240719155342839.png)

要优化 IO 的流程用时，有5种 IO 模型

## 阻塞IO

## 非阻塞IO

![image-20240719155702240](C:\Users\wilzsn\AppData\Roaming\Typora\typora-user-images\image-20240719155702240.png)

没什么用，用户进程在等待数据时没有干其他事，而且反复调用recvfrom函数容易导致CPU资源消耗。

## IO多路复用

IO多路复用：是利用单个线程来同时监听多个FD，并在某个FD可读、可写时得到通知，从而避免无效的等待，充分利用CPU资源。

### select

select 轮询 fd 数组，并阻塞等待数据。

![image-20240719160851043](C:\Users\wilzsn\AppData\Roaming\Typora\typora-user-images\image-20240719160851043.png)



![image-20240719162808512](C:\Users\wilzsn\AppData\Roaming\Typora\typora-user-images\image-20240719162808512.png)

涉及两次 fd_set 的复制，从用户态复制到内核态，再从内核态复制回用户态，再从用户态遍历已就绪的 fd ，读取其中的数据。

select 模式存在的问题：

- 需要将整个fd_set从用户空间拷贝到内核空间，select结束还要再次拷贝回用户空间
- select无法得知具体哪个fd就绪，需要遍历整个fd_set
- fd_set监听的fd数量不能超过1024。（因为fd_set数组为1024bit位数组）

### poll

![image-20240719163023032](C:\Users\wilzsn\AppData\Roaming\Typora\typora-user-images\image-20240719163023032.png)

与select相比：

- select模式中 fd_set 大小固定为1024，而pollfd 在内核中采用链表，理论上无上限。
- 监听FD越多，每次遍历消耗时间也越久，性能反而会下降。

### epoll

select 和 poll 只会通知用户进程有 FD 就绪，但不确定具体是哪个 FD ，需要用户进程逐个遍历FD来确认。

epoll 则会在通知用户进程 FD 就绪的同时，把已就绪的 FD 写入用户空间。

epoll把FD放入到红黑树，并设置回调函数。当数据就绪，则将FD假如到rdlist这个就绪列表中。

![image-20240719164221341](C:\Users\wilzsn\AppData\Roaming\Typora\typora-user-images\image-20240719164221341.png)

![image-20240719165121314](C:\Users\wilzsn\AppData\Roaming\Typora\typora-user-images\image-20240719165121314.png)



### IO多路复用-事件通知机制

LevelTriggered、EdgeTriggered

![image-20240719170540385](C:\Users\wilzsn\AppData\Roaming\Typora\typora-user-images\image-20240719170540385.png)

![image-20240719171229070](C:\Users\wilzsn\AppData\Roaming\Typora\typora-user-images\image-20240719171229070.png)

## 信号驱动IO

![image-20240719171541140](C:\Users\wilzsn\AppData\Roaming\Typora\typora-user-images\image-20240719171541140.png)



## 异步IO

![image-20240719171804038](C:\Users\wilzsn\AppData\Roaming\Typora\typora-user-images\image-20240719171804038.png)

异步IO 两个阶段都是非阻塞的

## Redis网络模型

Redis在核心业务部分（命令处理），是单线程执行

Redis在整个业务部分，就是多线程

![image-20240719172446386](C:\Users\wilzsn\AppData\Roaming\Typora\typora-user-images\image-20240719172446386.png)



![image-20240719173955336](C:\Users\wilzsn\AppData\Roaming\Typora\typora-user-images\image-20240719173955336.png)

# 通信协议

1. 客户端向服务端发送一条命令
2. 服务端解析并执行命令，返回响应结果给客户端

因此客户端发送命令的格式、服务端响应结果的格式必须有一个规范，这个规范就是通信协议。

在Redis中采用的是 **RESP** (Redis Serialization Protocol)

### RESP协议-数据类型

- 单行字符串：首字符为 ‘ + ’, 后面跟上字符串，以（“\r \n”）结尾。例如返回“ OK ”：“ + OK \r\n”
- 错误： 首字符为 ‘ - ’, 后面跟上字符串，以（“\r \n”）结尾。例如返回“ 错误信息 ”：“ - Error message \r\n”
- 数值：首字符为 ‘ ： ’, 后面跟上数字格式字符串，以（“\r \n”）结尾。
- 多行字符串：首字节是 ' $ '，表示二进制安全的字符串，最大支持512M![image-20240720082833644](C:\Users\wilzsn\AppData\Roaming\Typora\typora-user-images\image-20240720082833644.png)
- 数组：首字节是' * '，后面跟上数组元素个数，元素数据类型不限。![image-20240720082851916](C:\Users\wilzsn\AppData\Roaming\Typora\typora-user-images\image-20240720082851916.png)





# 内存策略

## 过期策略

用 expire 命令给Redis的 key 设置 TTL（存活时间）

在Redis的结构体中，有两个Dict：一个用来记录key-value；另一个用来记录key-TTL（利用两个 Dict 分别记录key-value，key-TTL，可以判断当前key是否过期）

TTL到期会采用 **惰性删除** 和 **周期删除**

惰性删除：当TTL过期时不是立即删除，而是再次访问该key的时候，再判断当前key的存活时间，如果过期才执行删除

周期删除：通过一个定时任务，周期性的抽样部分过期的key。执行周期有两种：

- Redis 会执行一个定时任务 serverCron( )，按照 server.hz的频率来执行过期 key 的清理，模式为 Slow （时间较久）（1秒中执行10次）
- Redis的每个事件循环前会调用beforeSleep( )函数，执行过期 key清理，模式为 Fast

## 淘汰策略

当Redis内存使用达到设置的阈值时，Redis主动挑选部分key删除以释放更多内存的流程。

