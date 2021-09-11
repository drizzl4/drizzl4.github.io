---
title: "InnoDB"
date: 2021-09-07T21:51:03+08:00
# aliases: ["/first"]
author: "drizzl4"
# author: ["Me", "You"] # multiple authors
TocOpen: false
draft: false
hidemeta: false
canonicalURL: "https://canonical.url/to/page"
disableHLJS: true # to disable highlightjs
disableShare: false
disableHLJS: false
hideSummary: false
ShowReadingTime: true
ShowBreadCrumbs: false
ShowPostNavLinks: true
cover:
    image: "<image path/url>" # image path/url
    alt: "<alt text>" # alt text
    caption: "<text>" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: true # only hide on current single page
editPost:
    URL: "https://github.com/drizzl4/drizzl4.github.io/blob/master/content"
    Text: "Edit" # edit text
    appendFilePath: true # to append file path to Edit link

# weight: 1
tags: ["Mysql","InnoDB"]
categories: ["Mysql","InnoDB"]
showToc: true
comments: true
description: ""
searchHidden: false
summary: "About InnoDB"
---

# InnoDB note

## InnoDB体系架构

![img](/images/55bb6bb1N19e97000.jpg)

### 后台线程

InnoDB存储引擎是多线程的模型，因此其后台有多个不同的后台线程，负责处理不同的任务。  

#### 1.Master Thread

Master Thread是一个非常核心的后台线程，主要负责将缓冲池中的数据异步刷新到磁盘，保证数据的一致性，包括脏页的刷新、合并插入缓冲（INSERT BUFFER）、UNDO页的回收等。  

#### 2.IO Thread

在InnoDB存储引擎中大量使用了AIO（Async IO）来处理写IO请求，这样可以极大提高数据库的性能。而IO Thread的工作主要是负责这些IO请求的回调（call back）处理。

##### 查看InnoDB 版本

```mysql
SHOW VARIABLES LIKE'innodb_version'
```

##### 查看/设置IO Thread数量

```mysql
SHOW VARIABLES LIKE'innodb_%io_threads'
```

##### 观察InnoDB中的IO Thread

```mysql
SHOW ENGINE INNODB STATUS
```
#### 3.Purge Thread

事务被提交后，其所使用的undolog可能不再需要，因此需要PurgeThread来回收已经使用并分配的undo页。  

```mysql
SHOW VARIABLES LIKE 'innodb_purge_threads'
```

#### 4.Page Cleaner Thread

Page Cleaner Thread是在InnoDB 1.2.x版本中引入的。其作用是将之前版本中脏页的刷新操作都放入到单独的线程中来完成。而其目的是为了减轻原Master Thread的工作及对于用户查询线程的阻塞，进一步提高InnoDB存储引擎的性能。

### 内存

#### 1.缓冲池

> InnoDB存储引擎是基于磁盘存储的，并将其中的记录按照页的方式进行管理。因此可将其视为基于磁盘的数据库系统（Disk-base Database）。在数据库系统中，由于CPU速度与磁盘速度之间的鸿沟，基于磁盘的数据库系统通常使用缓冲池技术来提高数据库的整体性能。
缓冲池简单来说就是一块内存区域，通过内存的速度来弥补磁盘速度较慢对数据库性能的影响。在数据库中进行读取页的操作，首先将从磁盘读到的页存放在缓冲池中，这个过程称为将页“FIX”在缓冲池中。下一次再读相同的页时，首先判断该页是否在缓冲池中。若在缓冲池中，称该页在缓冲池中被命中，直接读取该页。否则，读取磁盘上的页。
对于数据库中页的修改操作，则首先修改在缓冲池中的页，然后再以一定的频率刷新到磁盘上。这里需要注意的是，页从缓冲池刷新回磁盘的操作并不是在每次页发生更新时触发，而是通过一种称为Checkpoint的机制刷新回磁盘。同样，这也是为了提高数据库的整体性能。

缓冲池配置：

```mysql
SHOW VARIABLES 'innodb_buffer_poll_size'
```

缓冲池内存结构：

![img](/images/55bb6bb3N9ca9bc2f.jpg)

> 从InnoDB 1.0.x版本开始，允许有多个缓冲池实例。每个页根据哈希值平均分配到不同缓冲池实例中。这样做的好处是减少数据库内部的资源竞争，增加数据库的并发处理能力。可以通过参数innodb_buffer_pool_instances来进行配置，该值默认为1。

#### 2.LRU List、Free List和Flush List

> 通常来说，数据库中的缓冲池是通过LRU（Latest Recent Used，最近最少使用）算法来进行管理的。即最频繁使用的页在LRU列表的前端，而最少使用的页在LRU列表的尾端。当缓冲池不能存放新读取到的页时，将首先释放LRU列表中尾端的页。

> 在InnoDB存储引擎中，缓冲池中页的大小默认为16KB，同样使用LRU算法对缓冲池进行管理。稍有不同的是InnoDB存储引擎对传统的LRU算法做了一些优化。在InnoDB的存储引擎中，LRU列表中还加入了midpoint位置。新读取到的页，虽然是最新访问的页，但并不是直接放入到LRU列表的首部，而是放入到LRU列表的midpoint位置。这个算法在InnoDB存储引擎下称为midpoint insertion strategy。在默认配置下，该位置在LRU列表长度的5/8处。midpoint位置可由参数innodb_old_blocks_pct控制

```mysql
mysql＞SHOW VARIABLES LIKE'innodb_old_blocks_pct'\G;
***************************1.row***************************
Variable_name:innodb_old_blocks_pct
Value:37
1 row in set(0.00 sec)
```

参数innodb_old_blocks_pct默认值为37，表示新读取的页插入到LRU列表尾端的37%的位置（差不多3/8的位置)，midpoint之前的列表称为new列表，midpoint之后的列表称为old表    

此外，还有innodb_old_blocks_time参数，用于表示页读取到mid位置后需要等待多久才会被加入到LRU列表的热端。   

<hr>

通过命令SHOW ENGINE INNODB STATUS来观察LRU列表及Free列表的使用情况和运行状态：

```mysql
mysql＞SHOW ENGINE INNODB STATUS\G;
***************************1.row***************************
Type:InnoDB
Name:
Status:
=====================================
120725 22:04:25 INNODB MONITOR OUTPUT
=====================================
Per second averages calculated from the last 24 seconds
……
Buffer pool size 327679
Free buffers 0
Database pages 307717
Old database pages 113570
Modified db pages 24673
Pending reads 0
Pending writes:LRU 0,flush list 0,single page 0
Pages made young 6448526,not young 0
48.75 youngs/s,0.00 non-youngs/s
Pages read 5354420,created 239625,written 3486063
55.68 reads/s,81.74 creates/s,955.88 writes/s
Buffer pool hit rate 1000/1000,young-making rate 0/1000 not 0/1000
……
```

pages made young显示了LRU列表中页移动到前端的次数，因为该服务器在运行阶段没有改变innodb_old_blocks_time的值，因此not young为0。youngs/s、non-youngs/s表示每秒这两类操作的次数。这里还有一个重要的观察变量——Buffer pool hit rate，表示缓冲池的命中率  

<hr>

InnoDB存储引擎从1.0.x版本开始支持压缩页的功能，即将原本16KB的页压缩为1KB、2KB、4KB和8KB。而由于页的大小发生了变化，LRU列表也有了些许的改变。对于非16KB的页，是通过unzip_LRU列表进行管理的  

```mysql
mysql＞SHOW ENGINE INNODB STATUS\G;
……
Buffer pool hit rate 999/1000,young-making rate 0/1000 not 0/1000
Pages read ahead 0.00/s,evicted without access 0.00/s,Random read ahead 0.00/s
LRU len:1539,unzip_LRU len:156
I/O sum[0]:cur[0],unzip sum[0]:cur[0]
……
```

可以看到LRU列表中一共有1539个页，而unzip_LRU列表中有156个页。这里需要注意的是，LRU中的页包含了unzip_LRU列表中的页。  

关于unzip_LRU从缓存池中分配内存，例如从缓冲池中申请页为4KB的大小：

1）检查4KB的unzip_LRU列表，检查是否有可用的空闲页；

2）若有，则直接使用；

3）否则，检查8KB的unzip_LRU列表；

4）若能够得到空闲页，将页分成2个4KB页，存放到4KB的unzip_LRU列表；

5）若不能得到空闲页，从LRU列表中申请一个16KB的页，将页分为1个8KB的页、2个4KB的页，分别存放到对应的unzip_LRU列表中。

<hr>

> 在LRU列表中的页被修改后，称该页为脏页（dirty page），即缓冲池中的页和磁盘上的页的数据产生了不一致。这时数据库会通过CHECKPOINT机制将脏页刷新回磁盘，而Flush列表中的页即为脏页列表。需要注意的是，脏页既存在于LRU列表中，也存在于Flush列表中。LRU列表用来管理缓冲池中页的可用性，Flush列表用来管理将页刷新回磁盘，二者互不影响。

#### 3.重做日志缓冲

InnoDB存储引擎首先将重做日志信息先放入到这个缓冲区，然后按一定频率将其刷新到重做日志文件。重做日志缓冲一般不需要设置得很大，因为一般情况下每一秒钟会将重做日志缓冲刷新到日志文件，因此用户只需要保证每秒产生的事务量在这个缓冲大小之内即可。

#### 4.额外的内存池

额外的内存池通常被DBA忽略，他们认为该值并不十分重要，事实恰恰相反，该值同样十分重要。在InnoDB存储引擎中，对内存的管理是通过一种称为内存堆（heap）的方式进行的。在对一些数据结构本身的内存进行分配时，需要从额外的内存池中进行申请，当该区域的内存不够时，会从缓冲池中进行申请。例如，分配了缓冲池（innodb_buffer_pool），但是每个缓冲池中的帧缓冲（frame buffer）还有对应的缓冲控制对象（buffer control block），这些对象记录了一些诸如LRU、锁、等待等信息，而这个对象的内存需要从额外内存池中申请。因此，在申请了很大的InnoDB缓冲池时，也应考虑相应地增加这个值。

## Checkpoint技术

缓冲池是为了协调CPU速度与磁盘速度的差距，因此对页的操作首先是在缓冲池中完成的，如果一条DML语句，入Update或Delete改变了页中的记录，那么此时页是脏的   

缓冲池中的页比磁盘的新，缓冲池将新版本的页从缓冲池刷新到磁盘  

> 倘若每次一个页发生变化，就将新页的版本刷新到磁盘，那么这个开销是非常大的。若热点数据集中在某几个页中，那么数据库的性能将变得非常差。同时，如果在从缓冲池将页的新版本刷新到磁盘时发生了宕机，那么数据就不能恢复了。为了避免发生数据丢失的问题，当前事务数据库系统普遍都采用了Write Ahead Log策略，即当事务提交时，先写重做日志，再修改页。当由于发生宕机而导致数据丢失时，通过重做日志来完成数据的恢复。这也是事务ACID中D（Durability持久性）的要求。

{{< collapse summary="为何会有Checkpoint技术" >}}

思考下面的场景，如果重做日志可以无限地增大，同时缓冲池也足够大，能够缓冲所有数据库的数据，那么是不需要将缓冲池中页的新版本刷新回磁盘。因为当发生宕机时，完全可以通过重做日志来恢复整个数据库系统中的数据到宕机发生的时刻。但是这需要两个前提条件：  

1. 缓冲池可以缓存数据库中所有的数据

2. 重做日志可以无限增大

对于第一个前提条件，有经验的用户都知道，当数据库刚开始创建时，表中没有任何数据。缓冲池的确可以缓存所有的数据库文件。然而随着市场的推广，用户的增加，产品越来越受到关注，使用量来越大。这时负责后台存储的数据库的容量必定会不断增大。当前3TB的MySQL数据库已并不少见，但是3 TB的内存却非常少见。目前Oracle Exadata旗舰数据库一体机也就只有2 TB的内存。因一个假设对于生产环境应用中的数据库是很难得到保证的。再来看第二个前提条件：重做日志可以无限增大。也许是可以的，但是这对成本的要求太高，同时不便于运维。DBA或SA不能知道什么时候日志是否已经接近于磁盘可使用空间的阈值，并且要让存储设备支持可动态扩展也是需要一定的技巧和设备支持的。好的，即使上述两个条件都满足，那么还有一个情况需要考虑：宕机后数据库的恢间。当数据库运行了几个月甚至几年时，这时发生宕机，重新应用重做日志的时间会非常久，此时恢复的代价也会非常大。

{{< /collapse >}}

Checkpoint（检查点）技术的目的是解决以下几个问题：

 1. 缩短数据库的恢复时间
 2. 缓冲池不够用时，将脏页刷新到磁盘
 3. 重做日志不可用时，刷新脏页

