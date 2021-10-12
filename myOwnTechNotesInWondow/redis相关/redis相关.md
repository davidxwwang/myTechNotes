## redis

redis作为缓存和数据源似乎是不同的：

- 作为缓存时候，可以根据LRU的策略随机的删除一些键（见https://redis.io/topics/lru-cache）；

- 但是如果作为数据源的话，任何键的删除都是不行的。

  ```
   需要看下
   config get maxmemory-policy 
   如果不是noeviction，是绝对不能作为数据源使用的，切记！！！
  ```

  



redis是单线程模式，预防多线程可能引起的竞争问题。

redis比较适合放热数据，

1. epoll模型
2. 多路复用
3. 单线程



redis 批量操作（pipeline）



redis客户端执行命令步骤：

发送命令，命令排队，命令执行，返回结果



redis持久化：

redis持久化是影响redis性能的主要因素之一。

rdb：存的是数据

AOF（append only file）：以独立的日志记录每次写命令。存的是命令。



redis事务：

redis事务不能支持回滚





高可用问题

主节点出现故障无法修复，

分布式问题



#### Redis内存问题

```
mem_fragmentation_ratio = used_memory_rss_human/used_memory_human
当mem_fragmentation_ratio > 1 说明redis中存在碎片
当mem_fragmentation_ratio < 1 说明操作系统把Redis内存交换（Swap）到硬盘了。

```

##### redis内存分配算法

内存分配器

ptmalloc

tcmalloc

Jemalloc

redis默认使用Jemalloc https://owent.net/2013/867.html   https://time.geekbang.org/column/article/9759



#### 高可用设计

##### Redis Sentinial：（故障发现，故障自动转移，配置中心，客户端通知）

会准实时感知每个redis节点的状态。

目的：（1）选集群的主节点。

​             （2）判断节点是否可达等。

使用Raft算法--》选出Sentinial集群的Leader做故障转移的工作



故障转移：





缓存雪崩： 缓存层由于某些原因不能提供服务，请求全部打到mysql上，可能造成服务级联不可用的情况。

此时要限流，要保护（Hystrix）



redis的编码：

#### RedisObject

redis中存的值都是redisObject类型

#### Bitmap类型：

特点：

（1）省内存，1个bit就可以表示一个id，那么1Mb就可以表示1024乘1024乘8个id，大约840万个id。

（2）对于那种只计算有和无的数据，特别有效，因为可以把坐标当作id来用。比如用户id，订单id（前提是他们的类型都是Integer或Long类型），但是对于那种String类型的id就不行了，估计要使用BloomFilter；

（3）如果id数值特别大，而且id的总量有非常小，反而使用bitmap不划算呢。

#### HyperLogLog:

目标： 使用极小的数据集，完成独立总数的统计。（基数统计）

缺点：有误差率。

#### GEO相关（地理位置相关）



##### 发布订阅

（1）无法对发布的消息持久化。

（2）不能支持重复消费消息。



#### redis中的数据结构

#### RedisObject对象

在redis.h文件中

```
typedef struct redisObject {

    // 类型
    unsigned type:4;

    // 编码
    unsigned encoding:4;

    // 对象最后一次被访问的时间
    unsigned lru:REDIS_LRU_BITS; /* lru time (relative to server.lruclock) */

    // 引用计数
    int refcount;

    // 指向实际值的指针
    void *ptr;

} robj;
```



#### RedisDB

```
typedef struct redisDb {

    // 数据库键空间，保存着数据库中的所有键值对
    dict *dict;                 /* The keyspace for this DB */

    // 键的过期时间，字典的键为键，字典的值为过期事件 UNIX 时间戳(带过期时间的keys)
    dict *expires;              /* Timeout of keys with a timeout set */

    // 正处于阻塞状态的键
    dict *blocking_keys;        /* Keys with clients waiting for data (BLPOP) */

    // 可以解除阻塞的键
    dict *ready_keys;           /* Blocked keys that received a PUSH */

    // 正在被 WATCH 命令监视的键，用于事务
    dict *watched_keys;         /* WATCHED keys for MULTI/EXEC CAS */

    struct evictionPoolEntry *eviction_pool;    /* Eviction pool of keys */

    // 数据库号码
    int id;                     /* Database ID */

    // 数据库的键的平均 TTL ，统计信息
    long long avg_ttl;          /* Average TTL, just for stats */

} redisDb;
```



##### 1.SDS（simple dynamic String）

```
struct sdshdr {
    long len;   // 记录buf中已使用字节数量
    long free;  // 记录buf中未使用字节数量
    char buf[]; // 字节数组，用于保存字符串
};
```

特点

- 常数复杂度获取字符串长度以及append字符串
- 对append这种操作，内存预先分配（降低内存分配次数，代价是占用了额外的内存，而且不会主动释放），杜绝缓冲区溢出

#### 2.链表（LIST的底层结构）

```
typedef struct listNode {
    struct listNode *prev; // 前置节点
    struct listNode *next; // 后置节点
    void *value;  // 数据
} listNode;

typedef struct list {
    listNode *head; // List头节点
    listNode *tail; // List尾节点
    void *(*dup)(void *ptr);
    void (*free)(void *ptr);
    int (*match)(void *ptr, void *key);
    unsigned int len;
} list;

typedef struct listIter {
    listNode *next;
    listNode *prev;
    int direction;
} listIter;
```

#### 3.字典

```
// hash表节点
typedef struct dictEntry {
    void *key;
    void *val;
    struct dictEntry *next;
} dictEntry;

typedef struct dictType {
    unsigned int (*hashFunction)(const void *key);
    void *(*keyDup)(void *privdata, const void *key);
    void *(*valDup)(void *privdata, const void *obj);
    int (*keyCompare)(void *privdata, const void *key1, const void *key2);
    void (*keyDestructor)(void *privdata, void *key);
    void (*valDestructor)(void *privdata, void *obj);
} dictType;

typedef struct dict {
    dictEntry **table;
    dictType *type;  // 类型特定函数
    unsigned int size;
    unsigned int sizemask; // hash表大小掩码，用于计算索引
    unsigned int used;
    void *privdata;
} dict;
```

哈希表用的hash算法是：

MurmurHash2 32 bit 算法：这种算法的分布率和速度都非常好



#### 跳表（ZSet的底层结构）

注意：它是有序链表的扩展。（类似给有序链表加多层索引）



#### ZipList（压缩结构，线性连续的内存结构）



#### redis中的阻塞队列（BLPOP，BRPUSH）

#### redis阻塞

（1）对超级大的map（几十万个field）进行hgetall等

（2）cpu饱和

### 重要的参考资料与文献

全面细致的介绍redis ：   http://redisbook.com/





### redis lua脚步

```javascript
KEYS[1] ARGV[1]

KEYS[1] 用来表示在redis 中用作键值的参数占位，主要用來传递在redis 中用作key值的参数。

ARGV[1] 用来表示在redis 中用作参数的占位，主要用来传递在redis中用做 value值的参数。


eg : EVAL script numkeys key [key ...] arg [arg ...]

EVAL "return {KEYS[1],KEYS[2],ARGV[1],ARGV[2]}" 2 key1 key2 first second(要指名KEYS的数目)

eval "return redis.call('set', KEYS[1], ARGV[1])" 1 "name_david" "gohomeLANZHOY" 
其中有1个key，name_david是KEYS[1], gohomeLANZHOY是values，是ARGV[1],
这句话的意思是 “set name_david gohomeLANZHOY”
```



#### 分布式锁

https://martin.kleppmann.com/2016/02/08/how-to-do-distributed-locking.html

