---
description: redis想必大家都用过，但是一些底层实现的知识点还是需要好好看看，这篇博客就redis的数据结构为核心，总结网上的一些文章，当做学习笔记记录下来。
---

# 数据结构（详细）

### 一、Redis是什么？ <a href="#yi-redis-shi-shi-mo" id="yi-redis-shi-shi-mo"></a>

Redis是一个开源的，**基于内存的数据结构存储**，可用作于数据库、**缓存**、消息中间件。redis的常用命令可以去查阅官网，这里不做介绍。

### 二、为什么要用Redis <a href="#er-wei-shi-mo-yao-yong-redis" id="er-wei-shi-mo-yao-yong-redis"></a>

Redis是基于内存的，用于缓存的一种技术。而且通常是以kv的方式进行存储的，有人就问了，为啥不用数据结构的map呢？map不也差不多？下面是几点原因：

1. 数据结构的map是本地缓存，如果想用分布式缓存则做不到。多台机器缓存不具有**一致性。**
2. map是基于应用程序的，如果一个map特别大，那么运行起来会非常慢甚至程序会崩溃。而redis作为一个独立的缓存中间件，与应用程序解耦。
3. redis是专业做缓存的，不需要程序员去专门进行内存管理，而且可以将缓存数据存储到硬盘里，并且可以恢复，并且提供了多种数据结构及缓存过期等机制。

### 三、Redis的数据结构 <a href="#san-redis-de-shu-ju-jie-gou" id="san-redis-de-shu-ju-jie-gou"></a>

Redis 有 5 种基础数据结构，它们分别是：**string(字符串)、list(链表)、hash(哈希表)、set(集合) 和 zset(有序集合)**。要说明的是，这些数据结构表示的是redis key-value中“value”的类型，redis的key都是字符串。而redis底层并不是直接使用这些类型，而是构造了一个redisObject的类型。也就是说，我们在redis里创建一个key-value，会至少创建出两个对象，key对象和value对象。redisObject数据结构：

```c
typedef struct redisObject{

    // 对象的类型
    unsigned type 4:;

    // 对象的编码格式
    unsigned encoding:4;

    // 指向底层实现数据结构的指针
    void * ptr;

    //.....


}robj;
```

![](https://stormspirit22.github.io/images/redis/redis\_object.png)

（上面这张图现在看不懂没关系，等下面先介绍了几种数据结构再回过头来看就懂了。）

简单来说就是Redis对`key-value`封装成对象，key是一个对象，value也是一个对象。每个对象都有type(类型)、encoding(编码)、ptr(指向底层数据结构的指针)来表示。

下面分别就这几种类型的底层实现介绍一下。

#### 1、SDS <a href="#1sds" id="1sds"></a>

redis是用c写的，但是redis的string并不是c里的string直接拿来用的。redis的string类型是一个叫做\*\*简单动态字符串(Simple dynamic string,SDS)\*\*的数据结构，源码如下：

```c
struct sdshdr{

    // 字节数组，用于保存字符串
    char buf[];

    // 记录buf数组中已使用的字节数量，也是字符串的长度
    int len;

    // 记录buf数组未使用的字节数量
    int free;
}
```

可以看到sds的类型如图所示：

![](https://stormspirit22.github.io/images/redis/redis1-1.png)

那么sds与c的string相比有何优化的地方呢？

1. sdshdr数据结构中用len属性记录了字符串的长度。那么**获取字符串的长度时，时间复杂度只需要O(1)**。而c获取字符串长度都是O(N)的操作，每次都需要遍历整个数组。
2. SDS不会发生溢出的问题，如果修改SDS时，空间不足。先会扩展空间，再进行修改！(**内部实现了动态扩展机制**)。而对于c来说
3. SDS可以**减少内存分配的次数**(空间预分配机制)。在扩展空间时，除了分配修改时所必要的空间，还会分配额外的空闲空间(free 属性)。
4. SDS是**二进制安全的**，所有SDS API都会以处理二进制的方式来处理SDS存放在buf数组里的数据。

#### 2、链表 <a href="#2-lian-biao" id="2-lian-biao"></a>

链表这个数据结构大家应该也很熟悉了，在redis中的链表实现源码如下；

```c
typedef struct listNode {
    struct listNode *prev;
    struct listNode *next;
    void *value;
} listNode;
```

可以看到，多个 listNode 可以通过 `prev` 和 `next` 指针组成双向链表：

![](https://stormspirit22.github.io/images/redis/redis1-2.png)

使用listNode是可以组成链表了，Redis中**使用list结构来持有链表**：

```c
typedef struct list{

    //表头结点
    listNode  *head;

    //表尾节点
    listNode  *tail;

    //链表长度
    unsigned long len;

    //节点值复制函数
    void *(*dup) (viod *ptr);

    //节点值释放函数
    void  (*free) (viod *ptr);

    //节点值对比函数
    int (*match) (void *ptr,void *key);

}list
```

如下图所示：

![](https://stormspirit22.github.io/images/redis/redis1-3.png)

**Redis的链表有以下特性： #**

* 无环双向链表
* 获取表头指针，表尾指针，链表节点长度的时间复杂度均为O(1)
* 链表使用`void *`指针来保存节点值，可以保存各种不同类型的值

**链表的基本操作 #**

* `LPUSH` 和 `RPUSH` 分别可以向 list 的左边（头部）和右边（尾部）添加一个新元素；
* `LRANGE` 命令可以从 list 中取出一定范围的元素；
* `LINDEX` 命令可以从 list 中取出指定下表的元素，相当于 Java 链表操作中的 `get(int index)` 操作；

例子：

```bash
> rpush mylist A
(integer) 1
> rpush mylist B
(integer) 2
> lpush mylist first
(integer) 3
> lrange mylist 0 -1    # -1 表示倒数第一个元素, 这里表示从第一个元素到最后一个元素，即所有
1) "first"
2) "A"
3) "B"
```

list实现队列：

```bash
> RPUSH books python java golang
(integer) 3
> LPOP books
"python"
> LPOP books
"java"
> LPOP books
"golang"
> LPOP books
(nil)
```

list实现栈：

```bash
> RPUSH books python java golang
(integer) 3
> LPOP books
"python"
> LPOP books
"java"
> LPOP books
"golang"
> LPOP books
(nil)
```

#### 3、哈希表 <a href="#3-ha-xi-biao" id="3-ha-xi-biao"></a>

在Redis中，`key-value`的数据结构底层就是哈希表来实现的。Redis 中的哈希表相当于 Java 中的 **HashMap**，内部实现也差不多类似，都是通过 **“数组 + 链表”** 的链地址法来解决部分 **哈希冲突**，同时这样的结构也吸收了两种不同数据结构的优点。

Redis的Hash类型通常用于存储对象数据，hash 类型很像一个关系型数据库的数据表，hash 的 Key 是一个唯一值，Value 部分是一个 hashmap 的结构。操作上可以用hget key field来得到value，或者hgetall key来得到该key下所有的field和对应的value。如图：

![](https://stormspirit22.github.io/images/redis/redis1-4.png)

在Redis里边，哈希表使用dictht结构来定义：

```c
 typedef struct dictht{

        //哈希表数组
        dictEntry **table;  

        //哈希表大小
        unsigned long size;    

        //哈希表大小掩码，用于计算索引值
        //总是等于size-1
        unsigned long sizemark;     

        //哈希表已有节点数量
        unsigned long used;

    }dictht
```

dictEntry结构：

```c
  typedef struct dictEntry {

        //键
        void *key;

        //值
        union {
            void *value;
            uint64_tu64;
            int64_ts64;
        }v;    

        //指向下个哈希节点，组成链表
        struct dictEntry *next;

    }dictEntry;
```

哈希表最终dict结构：

```c
typedef struct dict {

    //类型特定函数
    dictType *type;

    //私有数据
    void *privdata;

    //哈希表
    dictht ht[2];

    //rehash索引
    //当rehash不进行时，值为-1
    int rehashidx;  

}dict;


//-----------------------------------

typedef struct dictType{

    //计算哈希值的函数
    unsigned int (*hashFunction)(const void * key);

    //复制键的函数
    void *(*keyDup)(void *private, const void *key);

    //复制值得函数
    void *(*valDup)(void *private, const void *obj);  

    //对比键的函数
    int (*keyCompare)(void *privdata , const void *key1, const void *key2)

    //销毁键的函数
    void (*keyDestructor)(void *private, void *key);

    //销毁值的函数
    void (*valDestructor)(void *private, void *obj);  

}dictType
```

最后，redis的哈希表结构：

![](https://stormspirit22.github.io/images/redis/redis1-5.png)

从代码实现和示例图上我们可以发现，**Redis中有两个哈希表**：

* ht\[0]：用于存放**真实**的`key-vlaue`数据
* ht\[1]：用于**扩容(rehash)**

**Rehash**

大字典的扩容是比较耗时间的，需要重新申请新的数组，然后将旧字典所有链表中的元素重新挂接到新的数组下面，这是一个 O(n) 级别的操作，作为单线程的 Redis 很难承受这样耗时的过程，所以 Redis 使用 **渐进式 rehash** 小步搬迁：

![](https://stormspirit22.github.io/images/redis/redis1-6.png)

渐进式 rehash 会在 rehash 的同时，保留新旧两个 hash 结构，如上图所示，查询时会同时查询两个 hash 结构，然后在后续的定时任务以及 hash 操作指令中，循序渐进的把旧字典的内容迁移到新字典中。当搬迁完成了，就会使用新的 hash 结构取而代之。具体过程：

* (1:在字典中维持一个索引计数器变量rehashidx，并将设置为0，表示rehash开始。
* (2:在rehash期间每次对字典进行增加、查询、删除和更新操作时，**除了执行指定命令外**；还会将ht\[0]中rehashidx索引上的值**rehash到ht\[1]**，操作完成后rehashidx+1。
* (3:字典操作不断执行，最终在某个时间点，所有的键值对完成rehash，这时**将rehashidx设置为-1，表示rehash完成**
* (4:在渐进式rehash过程中，字典会同时使用两个哈希表ht\[0]和ht\[1]，所有的更新、删除、查找操作也会在两个哈希表进行。例如要查找一个键的话，**服务器会优先查找ht\[0]，如果不存在，再查找ht\[1]**，诸如此类。此外当执行**新增操作**时，新的键值对**一律保存到ht\[1]**，不再对ht\[0]进行任何操作，以保证ht\[0]的键值对数量只减不增，直至变为空表。

**扩缩容的条件**

正常情况下，当 hash 表中 **元素的个数等于第一维数组的长度时**，就会开始扩容，扩容的新数组是 **原数组大小的 2 倍**。不过如果 Redis 正在做 `bgsave(持久化命令)`，Redis 尽量不去扩容，但是如果 hash 表非常满了，**达到了第一维数组长度的 5 倍了**，这个时候就会 **强制扩容**。

当 hash 表因为元素逐渐被删除变得越来越稀疏时，Redis 会对 hash 表进行缩容来减少 hash 表的第一维数组空间占用。所用的条件是 **元素个数低于数组长度的 10%**，缩容不会考虑 Redis 是否在做 `bgsave`。

#### 4、压缩列表（ziplist） <a href="#4-ya-suo-lie-biao-ziplist" id="4-ya-suo-lie-biao-ziplist"></a>

压缩列表(ziplist)是list和hash的底层实现之一。如果list的每个都是小整数值，或者是比较短的字符串，压缩列表(ziplist)作为list的底层实现。

压缩列表(ziplist)是Redis为了节约内存而开发的，是由一系列的**特殊编码的连续内存块**组成的**顺序性**数据结构。**压缩列表并不是以某种压缩算法进行压缩存储数据，而是它表示一组连续的内存空间的使用，节省空间**

![](https://stormspirit22.github.io/images/redis/redis1-7.png)

节点的结构图：

![](https://stormspirit22.github.io/images/redis/redis1-8.png)

> 压缩列表从表尾节点**倒序遍历**，首先指针通过zltail偏移量指向表尾节点，然后通过指向**节点记录的前一个节点的长度依次向前遍历访问整个压缩列表**。

#### 5、跳跃表（skiplist) <a href="#5-tiao-yue-biao-skiplist" id="5-tiao-yue-biao-skiplist"></a>

跳跃表(skiplist)是实现zset(**有序集合**)的底层数据结构之一，跳跃表是一种有序的数据结构，它通过每一个节点维持多个指向其它节点的指针，从而达到快速访问的目的。

skiplist由如下几个特点：

1. 有很多层组成，由上到下节点数逐渐密集，最上层的节点最稀疏，跨度也最大。
2. 每一层都是一个有序链表，只扫包含两个节点，头节点和尾节点。
3. 每一层的每一个每一个节点都含有指向同一层下一个节点和下一层同一个位置节点的指针。
4. 如果一个节点在某一层出现，那么该以下的所有链表同一个位置都会出现该节点

redis跳跃表具体的实现如图所示：

![](https://stormspirit22.github.io/images/redis/redis1-9.jpg)

在跳跃表的结构中有head和tail表示指向头节点和尾节点的指针，能后快速的实现定位。level表示层数，len表示跳跃表的长度，BW表示后退指针，在从尾向前遍历的时候使用。

BW下面还有两个值分别表示分值（score）和成员对象（各个节点保存的成员对象）。分值用来排序。跳跃表的实现中，除了最底层的一层保存的是原始链表的完整数据，上层的节点数会越来越少，并且跨度会越来越大。

跳跃表的上面层就相当于索引层，都是为了找到最后的数据而服务的，数据量越大，条表所体现的查询的效率就越高，和平衡树的查询效率相差无几。

#### 6、整数集合（intset） <a href="#6-zheng-shu-ji-he-intset" id="6-zheng-shu-ji-he-intset"></a>

整数集合是set(集合)的底层数据结构之一。当一个set(集合)**只包含整数值元素**，并且**元素的数量不多**时，Redis就会采用整数集合(intset)作为set(集合)的底层实现。

整数集合(intset)保证了元素是**不会出现重复**的，并且是**有序**的(从小到大排序)，intset的结构是这样子的：

```c
typeof struct intset {
        // 编码方式
        unit32_t encoding;
        // 集合包含的元素数量
        unit32_t lenght;
        // 保存元素的数组
        int8_t contents[];
} intset;
```

示例图：

![](https://stormspirit22.github.io/images/redis/redis1-10.png)

### 四、Redis五种数据结构的实现 <a href="#si-redis-wu-zhong-shu-ju-jie-gou-de-shi-xian" id="si-redis-wu-zhong-shu-ju-jie-gou-de-shi-xian"></a>

回到开始的那张图：

![](https://stormspirit22.github.io/images/redis/redis\_object.png)

可以看到在redis中，几种基本数据结构类型都有不同的编码方式，下面一一来分析一下。

#### 1、 字符串（String） <a href="#1-zi-fu-chuan-string" id="1-zi-fu-chuan-string"></a>

在Redis中，你如果设置数字进去，比如set num 123，那么redis会以int来编码，在redis中会生成一个redisObject对象，如下图所示：

![](https://stormspirit22.github.io/images/redis/redis1-11.jpg)

这个的意思是redis构造了一个类型为redis string、编码为int，指针指向123的对象。即上图的第一条。而如果你存储的不是数字类型，而是字符串类型，那么redis会使用前面介绍过的SDS方式进行存储，而这种方式有两种编码，如果字符串长度**大于32**字节，使用raw编码，否则使用embstr方式编码。

embstr和raw都是由SDS动态字符串构成的。唯一区别是：raw是分配内存的时候，redisobject和 sds 各分配一块内存，而embstr是redisobject和raw在一块儿内存中。

embstr和raw的**区别**：

* raw分配内存的时候，redisObject和SDS各分配一块内存，所以分配和释放内存的次数是两次。而embstr编码的数据是保存在一块连续的内存里，所以次数都是一次。

编码之间的**转换**：

* int类型如果存的**不再是一个整数值**，则会从int转成raw
* embstr是只读的，在修改的时候回从embstr转成raw

#### 2、列表（List） <a href="#2-lie-biao-list" id="2-lie-biao-list"></a>

在上面的图我们知道list类型有两种**编码格式**：

* ziplist：字符串元素的长度都小于64个字节`&&`总数量少于512个
* linkedlist：字符串元素的长度大于64个字节`||`总数量大于512个

编码之间的**转换：**

* 原本是ziplist编码的，如果保存的数据长度太大或者元素数量过多，会转换成linkedlist编码的。

#### 3、哈希（Hash） <a href="#3-ha-xi-hash" id="3-ha-xi-hash"></a>

在上面的图我们知道hash类型有两种**编码格式**：

* ziplist：key和value的字符串长度都小于64字节`&&`键值对总数量小于512
* hashtable：key和value的字符串长度大于64字节`||`键值对总数量大于512

编码之间的**转换：**

* 原本是ziplist编码的，如果保存的数据长度太大或者元素数量过多，会转换成hashtable编码的。

#### 4、集合（Set） <a href="#4-ji-he-set" id="4-ji-he-set"></a>

在上面的图我们知道set类型有两种**编码格式**：

* intset：保存的元素全都是整数`&&`总数量小于512
* hashtable：保存的元素不是整数`||`总数量大于512

编码之间的**转换：**

* 原本是intset编码的，如果保存的数据不是整数值或者元素数量大于512，会转换成hashtable编码的。

#### 5、有序集合（ZSet） <a href="#5-you-xu-ji-he-zset" id="5-you-xu-ji-he-zset"></a>

在上面的图我们知道set类型有两种**编码格式**：

* ziplist：元素长度小于64`&&`总数量小于128
* skiplist：元素长度大于64`||`总数量大于128

有序集合对象**同时采用skiplist和哈希表来实现**：

* skiplist能够达到插入的时间复杂度为O(logn)，根据成员查分值的时间复杂度为O(1)

编码之间的**转换：**

* 原本是ziplist编码的，如果保存的数据长度大于64或者元素数量大于128，会转换成skiplist编码的。

### 最后 <a href="#zui-hou" id="zui-hou"></a>

没想到redis的数据结构就这么一大篇，还只是简单介绍，所以看起来平时特别常用的东西，其实内部实现光是学习也许要花不少时间的。之后会介绍redis其他的一些功能特性。

### 参考 <a href="#can-kao" id="can-kao"></a>

[《【3y】从零单排学Redis【青铜】》](https://mp.weixin.qq.com/s?\_\_biz=MzI4Njg5MDA5NA==\&mid=2247484359\&idx=1\&sn=0994c6246990b7ad42a2d3f294042316\&chksm=ebd742c6dca0cbd0a826ace13f4d4eeff282052f4a97b31654ef1b3b32f991374f5c67a45ae9\&token=1834317504\&lang=zh\_CN\&scene=21#wechat\_redirect) [《Redis(1)——5种基本数据结构》](https://www.wmyskxz.com/2020/02/28/redis-1-5-chong-ji-ben-shu-ju-jie-gou/)
