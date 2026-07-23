# Redis
## 说明
Redis 的底层数据类型（也称为 “对象类型”）是其核心设计的一部分，决定了数据在内存中的存储结构、编码方式及支持的操作。与用户直接使用的 上层数据结构（如 String、List、Hash 等） 不同，底层数据类型更关注 “如何高效存储”，同一上层数据结构可能根据数据特征选择不同的底层编码。

Redis 底层核心数据类型可分为 基础类型 和 特殊类型，以下是详细分类及解析：

## 一、基础底层数据类型
基础类型是 Redis 最核心、最常用的底层实现，直接对应上层数据结构的主流编码方式，包括 5 种：  

1. 简单动态字符串（Simple Dynamic String, SDS）  
核心作用：Redis 中字符串（String）类型的默认底层实现（替代 C 语言原生字符串），同时也是 Hash、List 等类型中 “值” 的存储载体。  

设计特点：  
动态扩容：预分配冗余空间（避免频繁内存分配），例如字符串长度小于 1MB 时，扩容后容量为原长度的 2 倍；大于 1MB 时，每次额外增加 1MB。  
二进制安全：不依赖 \0 标识字符串结束，可存储图片、视频等二进制数据。  
记录长度：结构体中直接存储字符串长度（len）和未使用空间（free），获取长度的时间复杂度为 O(1)（C 语言原生字符串为 O (n)）。  
结构体示例：  
```
    struct sdshdr {
        int len;    // 已使用字节数
        int free;   // 未使用字节数
        char buf[]; // 存储字符串的字节数组
    };
```
2. 双向链表（Doubly Linked List）  
核心作用：Redis 中 List 类型的底层实现之一（当 List 中元素较多或元素为大字符串时使用），同时用于实现 Redis 的发布订阅、慢查询日志等功能。  
设计特点：  
双向节点：每个节点包含 prev（前驱指针）和 next（后继指针），支持双向遍历，方便在链表两端（头部 / 尾部）执行插入 / 删除操作（时间复杂度 O (1)）。  
记录元数据：链表结构体中存储 head（头节点）、tail（尾节点）、len（链表长度），可快速获取链表长度和首尾元素。  
无环结构：头节点的 prev 和尾节点的 next 均指向 NULL，避免遍历死循环。  
结构体示例：
    ```
    // 链表节点
    typedef struct listNode {
        struct listNode *prev;
        struct listNode *next;
        void *value; // 节点值（可存储任意类型数据）
    } listNode;

    // 链表本身
    typedef struct list {
        listNode *head;
        listNode *tail;
        unsigned long len; // 链表长度
        // 自定义函数（用于节点值的复制、释放、比较）
        void *(*dup)(void *ptr);
        void (*free)(void *ptr);
        int (*match)(void *ptr, void *key);
    } list;
    ```
3. 压缩列表（ZipList）
核心作用：Redis 中 List、Hash、Sorted Set 类型的 “紧凑存储” 实现（当数据量小时使用，优先于双向链表、哈希表），目的是节省内存。
设计特点：
连续内存：用一块连续的内存存储多个元素，避免链表的指针开销。
无指针：元素间通过 “长度字段” 定位，每个元素包含 “前元素长度” 和 “自身长度”，支持从两端遍历。
适合小数据：仅当元素数量少（如 List 元素数 < 512）且元素体积小（如字符串长度 < 64 字节）时使用，否则会转为其他编码（如双向链表、哈希表）。

结构示例（从左到右）：
zlbytes：4 字节，记录压缩列表总长度（用于扩容）。
zltail：4 字节，记录尾元素偏移量（用于快速定位尾元素）。
zllen：2 字节，记录元素个数（超过 65535 时需遍历获取）。
entryX：N 个元素，每个元素包含 “前元素长度”“元素类型”“元素值”。
zlend：1 字节，标记压缩列表结束（固定值 0xFF）。
4. 哈希表（Hash Table）
核心作用：Redis 中 Hash 类型的默认底层实现（当数据量较大时），同时也是 Redis 数据库（dict 结构体）的核心存储结构（存储键值对）。
设计特点：
数组 + 链表：哈希表主体是数组（table），每个数组元素是一个 “哈希桶”，桶内通过链表解决哈希冲突（链地址法）。
动态扩容 / 缩容：当 “负载因子”（used / size，used 是已使用桶数，size 是数组长度）超过阈值（默认 1）时，触发扩容（数组长度翻倍）；当负载因子低于阈值（默认 0.1）时，触发缩容（数组长度减半）。
渐进式 rehash：扩容 / 缩容时不一次性迁移所有数据，而是分多次（每次处理一个桶）迁移，避免阻塞主线程（Redis 单线程模型的关键优化）。
结构体示例：
// 哈希表节点
typedef struct dictEntry {
    void *key;          // 键
    union {             // 值（支持多类型）
        void *val;
        uint64_t u64;
        int64_t s64;
        double d;
    } v;
    struct dictEntry *next; // 下一个节点（解决哈希冲突）
} dictEntry;

// 哈希表
typedef struct dictht {
    dictEntry **table;  // 数组（存储哈希桶指针）
    unsigned long size; // 数组长度（2 的幂次）
    unsigned long sizemask; // 掩码（size-1，用于计算哈希值的桶索引）
    unsigned long used; // 已使用的节点数
} dictht;

// 字典（管理两个哈希表，用于渐进式 rehash）
typedef struct dict {
    dictht ht[2];       // ht[0] 是主表，ht[1] 是扩容/缩容表
    long rehashidx;     // rehash 进度（-1 表示未进行）
    unsigned long iterators; // 当前正在遍历的迭代器数量
} dict;
5. 跳表（Skip List）
核心作用：Redis 中 Sorted Set（有序集合）类型的底层实现之一（当 Sorted Set 中元素较多或元素为大字符串时使用），用于高效支持 “按分数排序” 和 “范围查询”。
设计特点：
多层索引：基于有序链表，额外建立多层 “索引链表”（如第 1 层索引每 2 个节点取一个，第 2 层每 4 个取一个），实现类似 “二分查找” 的效率。
随机层数：每个节点插入时，通过随机算法决定其层数（默认最大层数 32），避免索引层级过高导致的内存浪费。
高效操作：插入、删除、查找、范围查询（如 ZRANGEBYSCORE）的时间复杂度均为 O(log n)，优于普通有序链表（O (n)）。
结构体示例：
    ```
    // 跳表节点
    typedef struct zskiplistNode {
        sds ele;            // 元素值（字符串）
        double score;       // 分数（用于排序）
        struct zskiplistNode *backward; // 前驱节点（仅最底层有）
        // 每层的前进指针和跨度（跨度=当前节点到下一个节点的距离）
        struct zskiplistLevel {
            struct zskiplistNode *forward;
            unsigned long span;
        } level[];
    } zskiplistNode;

    // 跳表本身
    typedef struct zskiplist {
        struct zskiplistNode *header, *tail; // 头节点、尾节点
        unsigned long length;                // 节点总数（不包含头节点）
        int level;                           // 当前跳表的最大层数
    } zskiplist;
    ```
## 二、特殊底层数据类型
特殊类型用于特定场景的优化存储，并非所有上层数据结构都会用到，主要包括 2 种：

1. 整数集合（IntSet）
核心作用：Redis 中 Set 类型的底层实现之一（当 Set 中所有元素都是整数，且元素数量较少时使用）。  
设计特点：  
紧凑存储：用一块连续的内存存储整数，按从小到大排序，无冗余空间。  
动态类型：支持 3 种整数类型（int16_t、int32_t、int64_t），当插入的整数超出当前类型范围时，会自动 “升级” 类型（如从 int16_t 升级到 int32_t），但不支持降级。  
适合小整数集：仅当元素全为整数且数量 < 512 时使用，否则转为哈希表存储。  
结构体示例：  
    typedef struct intset {
        uint32_t encoding;  // 编码方式（INTSET_ENC_INT16/32/64）
        uint32_t length;    // 元素个数
        int8_t contents[];  // 存储整数的连续内存（实际类型由 encoding 决定）
    } intset;
2. 快速列表（Quick List）  
核心作用：Redis 3.2 版本后，替代 “双向链表 + 压缩列表”，成为 List 类型的 唯一底层实现，结合了两者的优点（节省内存 + 高效操作）。  
设计特点：  
链表 + 压缩列表：快速列表是一个 “双向链表”，但链表的每个节点不是单个元素，而是一个 压缩列表（即 “将多个小元素打包成一个压缩列表节点”）。  
平衡内存与性能：既避免了双向链表的指针开销（用压缩列表打包元素），又解决了压缩列表 “元素过多时遍历慢” 的问题（用链表拆分压缩列表节点）。  
可配置节点大小：通过 list-max-ziplist-size 配置压缩列表节点的最大容量，平衡内存占用和操作效率。  
## 三、上层数据结构与底层类型的映射关系
理解上层数据结构（用户直接使用的类型）如何选择底层类型，是掌握 Redis 性能优化的关键。以下是核心映射关系表：

| 上层数据结构 | 底层类型(编码方式)     | 适用场景(触发条件)                                             |
| ------------ | ---------------------- | -------------------------------------------------------------- |
| String       | SDS                    | 所有场景(String 仅依赖 SDS 实现)                               |
| List         | 快速列表(Quick List)   | 所有场景(Redis 3.2+唯一实现)                                   |
| Hash         | 压缩列表(ZipList)      | 元素数<512 且所有键值对长度<64 字节                            |
| Hash         | 哈希表(Hash Table)     | 元素数 ≥512 或存在键值对长度≥64 字节                           |
| Set          | 整数集合(IntSet)       | 元素全为整数且数量<512                                         |
| Set          | 哈希表(Hash Table)     | 元素包含非整数或数量 ≥512                                      |
| Sorted Set   | 压缩列表(ZipList)      | 元素数<128 且所有元素长度<64 字节                              |
| Sorted Set   | 跳表(Skip List)+哈希表 | 元素数 ≥128 或存在元素长度≥64 字节(跳表排序，哈希表快速查分数) |

总结
Redis 底层数据类型的设计核心是 “空间与时间的平衡”：

当数据量小时，优先使用 压缩列表、整数集合 等紧凑结构，节省内存；
当数据量增大时，自动转为 哈希表、跳表、快速列表 等高效结构，保证操作性能；
所有底层类型均为 Redis 单线程模型优化（如渐进式 rehash、无阻塞遍历），避免操作阻塞主线程。