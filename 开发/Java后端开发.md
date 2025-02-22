[TOC]

# Java 基础

## 知识体系

## Questions

### 1. `HashMap` 的底层原理，1.8与1.7的区别

#### (1) 扩容因子默认为什么是0.75

#### (2)为什么链表长度为8要转化为红黑树

### 2. `String` 、`StringBuffer` 、`StringBuilder`

### 3.强引用、软引用、弱引用、虚引用有什么区别

### 4.`CAS` 和 `ABA` 问题

### 5.原子类的实现原理

### 6.说一下 `CurrentHashMap` 如何实现线程安全的

### 7.代理模式、适配器模式、桥接模式、装饰器模式，本质区别是什么

### 8.`cglib` 和 `jdk` 动态代理的原理，有什么区别

### 9.大数据量的分页查询怎么优化

### 10.分库分表怎么做，可能会遇到什么问题

### 11.反射机制

### 12.`==`和 `equals` 区别

### 13.`Object` 类的 `hashcode` 方法的作用

# Java并发

## 知识体系

## Questions

### 1.线性池了解吗？参数有哪些？任务到达线程池的过程？线程池的大小如何设置？

#### 线程池参数

|           参数           |                             作用                             |
| :----------------------: | :----------------------------------------------------------: |
|       corePoolSize       |                        核心线程池大小                        |
|     maximumPoolSize      |                        最大线程池大小                        |
|      keepAliveTime       | 线程池中超过 corePoolSize 数目的空闲线程最大存活时间；可以allowCoreThreadTimeOut(true) 使得核心线程有效时间 |
|         TimeUnit         |                    keepAliveTime 时间单位                    |
|        workQueue         |                         阻塞任务队列                         |
|      threadFactory       |                         新建线程工厂                         |
| RejectedExecutionHandler | 拒绝策略。当提交任务数超过 maxmumPoolSize+workQueue 之和时，任务会交给RejectedExecutionHandler 来处理 |

#### 线程处理任务过程

* 当线程池小于 `corePoolSize`，新提交任务将创建一个新线程执行任务，即使此时线程池中存在空闲线程
* 当线程池达到 `corePoolSize` 时，新提交任务将被放入 `workQueue ` 中，等待线程池中任务调度执行
* 当 `workQueue` 已满，且 `maximumPoolSize` 大于 `corePoolSize` 时，新提交任务会创建新线程执行任务
* 当提交任务数超过 `maximumPoolSize` 时，新提交任务由 `RejectedExecutionHandler` 处理
* 当线程池中超过 `corePoolSize` 线程，空闲时间达到 `keepAliveTime` 时，关闭空闲线程 

#### 线程池的大小设置

* `CPU` 密集型
  * `CPU` 密集的意思是该任务需要大量的运算，而没有阻塞，`CPU` 一直全速运行
  * `CPU` 密集型任务尽可能的少的线程数量，一般为 `CPU` 核数 +1 个线程的线程池

* `IO` 密集型
  * 由于 `IO` 密集型任务线程并不是一直在执行任务，可以多分配一点线程数，如 `CPU * 2`
  * 也可以使用公式：`CPU` 核数 / (1 - 阻塞系数)；其中阻塞系数在 0.8 ～ 0.9 之间

### 2.`Java`乐观锁机制，`CAS`思想？缺点？是否原子性？

#### `Java`乐观锁机制

乐观锁体现的是悲观锁的反面。它是一种积极的思想，它总是认为数据是不会被修改的，所以是不会对数据上锁的。但是乐观锁在更新的时候会去判断数据是否被更新过。乐观锁的实现方案一般有两种（版本号机制和 `CAS`）。乐观锁适用于读多写少的场景，这样可以提高系统的并发量。在 `Java` 中 `java.util.concurrent.atomic`下的原子变量类就是使用了乐观锁的一种实现方式 `CAS` 实现的。

乐观锁，大多是基于数据版本 (`Version`)记录机制实现。即为数据增加一个版本标识，在基于数据库表的版本解决方案中，一般是通过为数据库表增加一个 `“version”` 字段来 实现。 读取出数据时，将此版本号一同读出，之后更新时，对此版本号加一。此时，将提 交数据的版本数据与数据库表对应记录的当前版本信息进行比对，如果提交的数据 版本号大于数据库表当前版本号，则予以更新，否则认为是过期数据。

#### `CAS`思想

`CAS` 就是 `compare and swap`（比较交换），是一种很出名的无锁的算法，就是可以不使用锁机制实现线程间的同步。使用CAS线程是不会被阻塞的，所以又称为非阻塞同步。`CAS` 算法涉及到三个操作：需要读写内存值 `V`； 进行比较的值 `A`； 准备写入的值 `B`。当且仅当 `V` 的值等于 `A` 的值等于 `V` 的值的时候，才用 `B` 的值去更新 `V` 的值，否则不会执行任何操作（比较和替换是一个原子操作 `A` 和 `V` 比较，`V` 和 `B` 替换），一般情况下是一个自旋操作，即不断重试。缺点：高并发的情况下，很容易发生并发冲突，如果 `CAS` 一直失败，那么就会一直重试，浪费 `CPU` 资源。

#### 原子性

功能限制 `CAS` 是能保证单个变量的操作是原子性的，在 `Java` 中要配合使用 `volatile` 关键字来保证线程的安全；当涉及到多个变量的时候 `CAS` 无能为力；除此之外 `CAS` 实现需要硬件层面的支持，在 `Java` 的普通用户中无法直接使用，只能借助 `atomic` 包下的原子类实现，灵活性受到了限制。

### 3.`ReenTrantLock` 使用方法？底层实现？和 `synchronized` 区别？

由于 `ReentrantLock` 是 `java.util.concurrent` 包下提供的一套互斥锁，相比 `Synchronized`，`ReentrantLock`类提供了一些高级功能，主要有以下三项：

* 等待可中断，持有锁的线程长期不释放的时候，正在等待的线程可以选择放弃等待，这相当于 `Synchronized` 来说可以避免出现死锁的情况。通过 `lock.lockInterruptibly()`来实现这个机制
* 公平锁，多个线程等待同一个锁时，必须按照申请锁的时间顺序获得锁，`Synchronized` 锁非公平锁，`ReentrantLock` 默认的构造函数是创建的非公平锁，可以通过参数 `true` 设为公平锁，但公平锁表现的性能不是很好
* 锁绑定多个条件，一个 `ReentrantLock` 对象可以同时绑定对个对象。`ReenTrantLock` 提供了一个 `Condition`（条件）类，用来实现分组唤醒需要唤醒的线程们，而不是像 `synchronized` 要么随机唤醒一个线程要么唤醒全部线程

#### 使用方法

 基于 `API` 层面的互斥锁，需要 `lock()` 和 `unlock()` 方法配合 `try/finally` 语句块来完成

#### 底层实现

` ReenTrantLock` 的实现是一种自旋锁，通过循环调用 `CAS` 操作来实现加锁。它的性能比较好也是因为避免了使线程进入内核态的阻塞状态。想尽办法避免线程进入内核的阻塞状态是我们去分析和理解锁设计的关键钥匙

#### `ReenTrantLock`和`synchronized` 的区别

 1、**底层实现**上来说，`synchronized` 是 `JVM` 层面的锁，是 `Java` 关键字，通过 `monitor` 对象来完成（`monitorenter与monitorexit`），对象只有在同步块或同步方法中才能调用 `wait/notify` 方法；`ReentrantLock` 是从 `jdk1.5` 以来（`java.util.concurrent.locks.Lock`）提供的 `API` 层面的锁。`synchronized` 的实现涉及到锁的升级，具体为无锁、偏向锁、自旋锁、向OS申请重量级锁；`ReentrantLock` 实现则是通过利用 `CAS` （`CompareAndSwap`）自旋机制保证线程操作的原子性和 `volatile` 保证数据可见性以实现锁的功能

 2、**是否可手动释放：`**synchronized` 不需要用户去手动释放锁，`synchronized` 代码执行完后系统会自动让线程释放对锁的占用； `ReentrantLock` 则需要用户去手动释放锁，如果没有手动释放锁，就可能导致死锁现象。一般通过`lock()` 和 `unlock()` 方法配合 `try/finally` 语句块来完成，使用释放更加灵活

 3、**是否可中断** `synchronized` 是不可中断类型的锁，除非加锁的代码中出现异常或正常执行完成； `ReentrantLock` 则可以中断，可通过 `trylock(long timeout,TimeUnit unit)` 设置超时方法或者将 `lockInterruptibly()` 放到代码块中，调用 `interrupt` 方法进行中断。

4、**是否公平锁 **`synchronized` 为非公平锁 `ReentrantLock` 则即可以选公平锁也可以选非公平锁，通过构造方法`new ReentrantLock` 时传入 `boolean` 值进行选择，为空默认 `false` 非公平锁，`true` 为公平锁

### 4.介绍一下 `Java` 的内存模型

`Java` 内存模型（`Java Memory Model，JMM`）就是一种符合内存模型规范的，屏蔽了各种硬件和操作系统的访问差异的，保证了 `Java` 程序在各种平台下对内存的访问都能保证效果一致的机制及规范。`JMM` 是一种规范，是解决由于多线程通过共享内存进行通信时，存在的本地内存数据不一致、编译器会对代码指令重排序、处理器会对代码乱序执行等带来的问题。目的是保证并发编程场景中的原子性、可见性和有序性。所以，`Java` 内存模型，除了定义了一套规范，还提供了一系列原语，封装了底层实现后，供开发者直接使用。我们前面提到，并发编程要解决**原子性、有序性和一致性**的问题。

* **原子性：** 在 `Java` 中，为了保证原子性，提供了两个高级的字节码指令 `Monitorenter` 和 `Monitorexit`。这两个字节码，在 `Java` 中对应的关键字就是 `Synchronized`。因此，在 `Java` 中可以使用 `Synchronized` 来保证方法和代码块内的操作是原子性的
* **可见性：**` Java` 内存模型是通过在变量修改后将新值同步回主内存，在变量读取前从主内存刷新变量值的这种依赖主内存作为传递媒介的方式来实现的。`Java` 中的 `Volatile` 关键字修饰的变量在被修改后可以立即同步到主内存。被其修饰的变量在每次使用之前都从主内存刷新。因此，可以使用 `Volatile` 来保证多线程操作时变量的可见性。除了 `Volatile`，`Java` 中的 `Synchronized` 和 `Final` 两个关键字也可以实现可见性。只不过实现方式不同
* **有序性：**在 `Java` 中，可以使用 `Synchronized` 和 `Volatile` 来保证多线程之间操作的有序性。区别：`Volatile` 禁止指令重排。`Synchronized` 保证同一时刻只允许一条线程操作

### 5.`volatile` 作用？底层实现？单例模式中 `volatile` 的作用？

#### 作用

保证数据的“可见性”：被 `volatile` 修饰的变量能够保证每个线程能够获取该变量的最新值，从而避免出现数据脏读的现象。 禁止指令重排：在多线程操作情况下，指令重排会导致计算结果不一致。

#### 底层实现

观察加入 `volatile` 关键字和没有加入 `volatile` 关键字时所生成的汇编代码发现，加入 `volatile` 关键字时，会多出一个 `lock` 前缀指令， `lock` 前缀指令实际上相当于一个内存屏障（也成内存栅栏），内存屏障会提供3个功能：

* 它确保指令重排序时不会把其后面的指令排到内存屏障之前的位置，也不会把前面的指令排到内存屏障的后面；即在执行到内存屏障这句指令时，在它前面的操作已经全部完成
* 它会强制将对缓存的修改操作立即写入主存
* 如果是写操作，它会导致其他 `CPU` 中对应的缓存行无效

#### 单例模式中 `volatile` 的作用

防止代码读取到 `instance` 不为 `null` 时，`instance` 引用的对象有可能还没有完成初始化

```java
class Singleton{
    private volatile static Singleton instance = null;   //禁止指令重排
    private Singleton() {        
    }
    public static Singleton getInstance() {
        if(instance==null) {
            synchronized (Singleton.class) {
                if(instance==null)
                    instance = new Singleton();
            }
        }
        return instance;
    }
}
```

### 6.`ThreadLocal` 原理

# JVM

## 知识体系

## Questions

### 1.`JVM` 内存划分

### 2.`GC` 垃圾收集器

### 3.垃圾收集算法，为什么新生代用标记-复制老年代用标记-整理

### 4.类加载流程

### 5.双亲委派机制，怎么样会打破双亲委派模型



# 数据库

## 知识体系

## Questions

###  1.`MySQL`的引擎了解吗？默认的是哪个？`InnoDB `和 `myISAM` 的区别？

`myISAM`:  支持表锁，适合读密集的场景，不支持外键，不支持事务，索引与数据在不同的文件，非聚簇索引

`InnoDB`:支持行、表锁，默认为行锁，适合并发场景，支持外键，支持事务，索引与数据同一文件，聚簇索引

### 2.介绍 `MVCC`

`MVCC` 是一种多版本并发控制机制，在大多数情况下代替行级锁，使用 `MVCC`，能降低其系统开销。`MVCC` 是通过保存数据在某个时间点的快照来实现的.。不同存储引擎的 `MVCC` 实现是不同的，典型的有乐观并发控制和悲观并发控制。`InnoDB` 的 `MVCC` 是通过在每行记录后面保存两个隐藏的列来实现的，这两个列，分别保存了这个行的创建时间，一个保存的是行的删除时间。这里存储的并不是实际的时间值，而是系统版本号(可以理解为事务的 `ID`)，每开始一个新的事务，系统版本号就会自动递增，事务开始时刻的系统版本号会作为事务的 `ID`。`InnoDB` 只会查找版本早于当前事务版本的数据行(也就是行的系统版本号小于或等于事务的系统版本号)，这样可以确保事务读取的行，要么是在事务开始前已经存在的，要么是事务自身插入或者修改过的。

### 3.`MySQL` 中一条 `SQL` 语句的执行过程

以查询语句为例

```mysql
select * from tb_student  A where A.age='18' and A.name='张三';
```

* 先检查该语句是否有权限，如果没有权限，直接返回错误信息，如果有权限，在 `MySQL8.0` 版本以前，会先查询缓存，以这条 `sql` 语句为 `key` 在内存中查询是否有结果，如果有直接缓存，如果没有，执行下一步
* 通过分析器进行词法分析，提取 `sql` 语句的关键元素，比如提取上面这个语句是查询 `select`，提取需要查询的表名为 `tb_student`,需要查询所有的列，查询条件是这个表的 `id='1'`。然后判断这个 `sql` 语句是否有语法错误，比如关键词是否正确等等，如果检查没问题就执行下一步
* 接下来就是优化器进行确定执行方案，上面的 `sql` 语句，可以有两种执行方案： a.先查询学生表中姓名为“张三”的学生，然后判断是否年龄是18。 b.先找出学生中年龄18岁的学生，然后再查询姓名为“张三”的学生。 那么优化器根据自己的优化算法进行选择执行效率最好的一个方案（优化器认为，有时候不一定最好）。那么确认了执行计划后就准备开始执行了
* 进行权限校验，如果没有权限就会返回错误信息，如果有权限就会调用数据库引擎接口，返回引擎的执行结果

### 4.如何查看 `sql` 语句的执行计划？用哪个关键字？使用这个关键字可以得到哪些信息？

用关键字 `explain` 来查看执行计划

![](https://i.loli.net/2021/07/01/xPIajcdQrysEJON.png)

![](https://i.loli.net/2021/07/01/k8TmrSEaK4M1lpc.png)

### 5.聚簇索引和非聚簇索引的区别，非聚簇索引是如何查询的？

**`MyISAM`的是非聚簇索引**，`B+Tree`的叶子节点上的 `data`，并不是数据本身，而是数据存放的地址。主索引和辅助索引没啥区别，只是主索引中的 `key` 一定得是唯一的。这里的索引都是非聚簇索引。非聚簇索引的两棵 `B+` 树看上去没什么不同，节点的结构完全一致只是存储的内容不同而已，主键索引 `B+` 树的节点存储了主键，辅助键索引 `B+` 树存储了辅助键。表数据存储在独立的地方，这两颗 `B+` 树的叶子节点都使用一个地址指向真正的表数据，对于表数据来说，这两个键没有任何差别。由于索引树是独立的，通过辅助键检索无需访问主键的索引树。`InnoDB` 的数据文件本身就是索引文件，`B+Tree` 的叶子节点上的 `data` 就是数据本身，`key` 为主键，这是聚簇索引。聚簇索引，叶子节点上的 `data` 是主键(所以聚簇索引的 `key`，不能过长)。 

**`InnoDB`使用的是聚簇索引**，将主键组织到一棵 `B+` 树中，而行数据就储存在叶子节点上，若使用 `where id = 14` 这样的条件查找主键，则按照 `B+` 树的检索算法即可查找到对应的叶节点，之后获得行数据。若对 `Name` 列进行条件搜索，则需要两个步骤：第一步在辅助索引 `B+` 树中检索 `Name`，到达其叶子节点获取对应的主键。第二步使用主键在主索引 `B+` 树种再执行一次 `B+` 树检索操作，最终到达叶子节点即可获取整行数据。

###  6.`MySQL`的 `ACID`，分别解释一下

**原⼦性：** 事务是最⼩的执⾏单位，不允许分割。事务的原⼦性确保动作要么全部完成，要么全不执行

**一致性：** 执⾏事务前后，数据保持⼀致，多个事务对同⼀个数据读取的结果是相同的

 **隔离性：** 并发访问数据库时，⼀个⽤户的事务不被其他事务所⼲扰，各并发事务之间数据库是独⽴的

 **持久性：** ⼀个事务被提交之后。它对数据库中数据的改变是持久的，即使数据库发⽣故障也不应该对其有任何影响

### 7.数据库索引的实现原理

`MySQL` 中默认的引擎为 `InnoDB`，`innoDB` 的索引使用的是 `B+` 树实现

**`B+` 树对比 `B树` 的好处**

* `IO` 次数少：`B+` 树的中间结点只存放索引，数据都存在叶结点中，因此中间结点可以存更多的数据，让索引树更加矮胖

* 范围查询效率更高：`B` 树需要中序遍历整个树，`B+` 树只需要遍历叶结点中的链表

* 查询效率更加稳定：每次查询都需要从根结点到叶结点，路径长度相同，所以每次查询的效率都差不多

**使用 `B` 树索引和哈希索引的比较**

哈希索引能以 `O(1)` 时间进行查找，但是只支持精确查找，无法用于部分查找和范围查找，无法用于排序与分组；`B` 树索引支持大于小于等于查找，范围查找。哈希索引遇到大量哈希值相等的情况后查找效率会降低。哈希索引不支持数据的排序

### 8.联合索引，最左前缀匹配规则，`sql`优化器在其应用

### 9.索引怎么优化

### 10.隔离级别

### 11.慢 `sql` 分析优化

### 12.`bin`、`redo`、`undo log`的区别

### 13.`MySQL` 分页查询是怎么实现的

# Spring

## 知识体系

## Questions

### 1.`Sping IOC AOP` 的实现原理

### 2.`Spring` 事物的实现原理

### 3.`bean` 的生命周期


# Redis

## 知识体系

## Questions

### 1.基于 `redis` 的分布式锁是如何实现的

![](https://i.loli.net/2021/07/01/WwxOC1ULHkAdIfF.png)

**实现思想**：获取锁的时候，使用 `setnc` 加锁，并使用 `expire` 命令为锁添加一个超时时间，超过该时间则自动释放锁，锁的 `value` 值为一个随机生成的 `UUID`，通过此在释放锁的时候进行判断；获取锁的时候还设置一个获取的超时时间，若超过这个时间则放弃获取锁；释放锁的时候，通过 `UUID` 判断是不是该锁，若是该锁，则执行 `delete` 进行锁释放。因为 `redis` 是单线程的，这样的话，如果4个请求打过来的话，只有一个请求能获得锁，获得锁即为 `key` 设置一个值，如果这个 `key` 存在就设置失败，也就是只有一个请求可以设置成功，这个请求处理完之后，会把这个 `key` 释放掉，那么剩下等着的3个请求一定会有一个请求重新对这个 `key` 设置一个值再次获得锁，依次类推。这样即当不同机子上的请求打过来的时候能够保证某一时刻只能有一个请求去消费资源，间接地形成一种加锁的机制。

#### `Redisson` 实现 `Redis` 分布式锁

* 加锁机制：客户端1面对分布式集群下，首先根据 `hash` 节点选择一台机器，发送一段 `lua` 脚本（保证复杂业务的原子性），将 `key `加锁（如果 `key `不存在就进行加锁）、设置过期时间、客户端 `id`

* 锁互斥机制：如果客户端2执行同样的lua脚本，代码中则会判断该锁 `key` 已存在，紧接着判断锁 `key` 的 `hash` 数据结构中是否包含客户端2的 `id`，如果不包含则会获得一个返回的数字，代表这个锁 `key` 的剩余生存时间，紧接着客户端2会进入 `while` 循环不断尝试加锁

* `watch dog` 自动延期机制：客户端1加锁有默认生存时间，如果想继续持有，可以在加锁成功时启动一个 `watch dog`看门狗（后台线程），每10秒检查一下，如果客户端1还持有锁 `key`，就会不断的延长锁 `key` 的生存时间。**(是对1中如何设置有效期的优化)**

* 可重入加锁机制： `hash` 数据结构中的客户端 `id` 加锁次数+1**(是对锁只能加一次，不可重入的优化)** 

* 释放锁机制：每次对数据结构中的加锁次数-1，当加锁次数为0，说明该客户端不持有锁，此时会从 `redis` 里删除这个 `key` ，另外的客户端可以尝试完成加锁。

**缺点**：如果采用这种方案，对某个 `redis master` 实例，写入 `mylock` 这种锁的 `value`，此时异步复制给对应的 `master slave` 实例，这个过程中一旦 `redis master` 宕机，`redis slave` 就变成了 `redis master` 会导致客户端2尝试加锁时，在新的`redis master `上完成了加锁，而客户端1也以为自己成功加了锁。导致多个客户端对一个分布式锁完成了加锁，导致各种脏数据产生。【`redis` 主从架构的主从异步复制导致 `redis` 分布式锁的最大缺陷】

### 2.跳表数据结构，`redis` 中哪里用到了跳表

### 3.出现缓存雪崩、击穿、穿透的情况及解决方法

### 4.持久化策略原理、实现细节及应用场景

### 5.主从复制原理及故障恢复

### 6.数据库缓存一致性

### 7.`AOF` 和 `RDB` 有啥优缺点

### 8.`redis` 的 `key` 的过期策略

### 9.`redis` 是单线程的吗，为什么这么快

### 10.`zset` 底层实现









