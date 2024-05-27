# Redis
学习一下Redis的原理

## 简单动态字符串SDS
C语言的字符串存在问题：
 - 获取字符串长度需要通过**运算**。
 - 非二进制安全。C语言的字符串，要从字符串的起始地址开始读，直到读到'\0'就结束。但是如果字符串本身就需要含有'\0'，这就出问题了。
 - 不可修改。

Redis 构建了一种简单动态字符串SDS。
 - SDS的一个实现源码（C语言实现，最大只能2<sup>8</sup>-1(255)个字节）：
```
struct __attribute__ ((__packed__)) sdshdr8{
    uint8_t len;         //buf已保存的字符串字节数，不包含结束标识
    uint8_t alloc;       //buf申请的总的字节数，不包含结束标识
    unsigned char flags; //不同SDS的头类型，用来控制SDS的头大小
    char buf[];
} sdshdr8
```
 - **动态扩容**.如果新字符串小于1M,则新空间为扩展后字符串长度的两倍+1;如果大于1M,则新空间为扩展后字符串长度+1M+1,称为内存预分配。
 - **减少内存分配次数**.
 - **二进制安全**.
 - **获取字符串长度的时间复杂度为O(1)**.

## IntSet
IntSet是Redis中set集合的一种实现方式，基于整数数组来实现，并且具备**长度可变**、**有序**等特征  
 - 结构如下  
```
typedef struct intset{
    uint32_t encoding;  //编码方式，支持存放16位、32位、64位整数
    uint32_t length;    //元素个数
    char contents[];    //整数数组，保存集合数据
} intset
```
 - 寻址地址：startPtr + (sizeof(int16) * index)。其中startPtr是起始地址,index是数组下标。
 - 长度可变：如果输入的整数超过了本身定义的编码方式，比如现在将50000插入到了int16_t的intset中，50000>2<sup>16</sup>，它会自动升级编码刀合适的大小。
   - 1.升级编码为INTSET_ENC_INT32，每个整数占4字节，并按照新的编码方式以及元素个数对数组进行扩容。
   - 2.**倒序**依次将数组中的元素拷贝到扩容后的正确位置。使用倒序是因为，如果使用正序，第一个元素扩容之后就会覆盖后面的元素，导致后面的元素丢失。如果从最后一个元素开始扩容，则不会有这个问题，因为最后一个元素之后没有元素，接着确定元素的起始地址，然后插入。  
     <img width="1167" alt="image" src="https://github.com/hhhhby/Redis/assets/113978854/9cbaf993-2d1f-4d66-98a7-c5cb4f6db237">
     <img width="1174" alt="image" src="https://github.com/hhhhby/Redis/assets/113978854/8b25320a-2f8a-4bcd-918b-d95e31e7455c">
     <img width="1181" alt="image" src="https://github.com/hhhhby/Redis/assets/113978854/b7c612f5-ccf1-4f4e-940b-a594719c3144">
   - 3.将待添加的元素放入数组末尾。
     <img width="1178" alt="image" src="https://github.com/hhhhby/Redis/assets/113978854/00ebe846-584c-4e66-b465-e9d433defec2">
   - 4.最后，将inset的encoding和length属性进行修改。
     <img width="939" alt="image" src="https://github.com/hhhhby/Redis/assets/113978854/a3015161-dbfb-4721-b105-314313607b34">
## Dict

     
## ZipList
 - 压缩列表可以看做一种连续内存空间的“双向链表”（不是真的链表，其地址分配都是连续的）
 - 列表的节点之间不是通过指针连接，而是记录上一节点和本节点长度来寻址，内存占用较低。
 - 如果列表数据过多，导致列表过长，可能影响查询性能。
 - 增加或删除较大数据时可能发生连续更新问题。



### ZipListEntry
ZipListEntry中的encoding编码分为字符串和整数两种：

 - **字符串**：如果encoding是以“00”、“01”或者“10”开头，则证明content是字符串。
<img width="1068" alt="image" src="https://github.com/hhhhby/Redis/assets/113978854/83fc7bb0-fb66-4de6-80c0-37288128a3ff">

   - 保存“ab”和“bc”
     - 确定encoding：“ab”占两个字节，即选择00开头，剩余的6位表示字节的大小，即000010，即encoding为00000010
     - previous_entry_length：“ab”是第一个entry，因此前一个entry长度为0，则它的值为00000000
     - content: a对应97（01100001），b对应98（01100010），即0110 0001 0110 0010
     - “ab”保存为0x00 0x02 0x61 0x62
   - 保存“bc”
     - 确定encoding：“bc”占两个字节，即选择00开头，剩余的6位表示字节的大小，即000010，即encoding为00000010
     - previous_entry_length：“bc”的前一个entry为“ab”,encoding为1个字节，previous_entry_length为1个字节，content长度为2个字节，则它的值为00000100
     - content: b对应98（01100010），b对应99（01100011），即0110 0010 0110 0011
     - “bc”保存为0x04 0x02 0x62 0x63
   - “ab” “bc”保存为0x000x020x610x620x040x020x620x63
<img width="1162" alt="image" src="https://github.com/hhhhby/Redis/assets/113978854/179f670a-8462-4597-b4a7-5c2de6a56e0b">

 - **整数**：如果encoding是以“11”开始，则证明content是整数，且encoding固定只占用1个字节。
   <img width="1092" alt="image" src="https://github.com/hhhhby/Redis/assets/113978854/97bc869a-ef99-44e1-a762-78091c253cd3">
 - 保存2和5
   <img width="959" alt="image" src="https://github.com/hhhhby/Redis/assets/113978854/348c7e51-a89c-4a4c-9393-b63a726a9797">
 - ZipList只能从两端进行遍历，如果保存的节点过多，如果要找的节点在中间，则需要遍历很多个节点才能找到。
 - 因此ZipList需要对节点的个数进行限制。
### ZipList的连锁更新问题

 - ZipList连续多次空间扩展操作称之为**连锁更新（Cascade Update）**，新增、删除都可能导致连锁更新的发生。 

## QuickList
 - Q1：ZipList虽然节省内存，但申请内存必须是连续空间，如果**内存占用较多，申请内存效率较低**，怎么办？
 - A1：限制ZipList的长度和entry的大小
 - Q2：但是就要存储大量数据，**超出了ZipList最佳上限**该怎么办？
 - A2：创建多个ZipList来分片存储数据
 - Q3：数据拆分比较**分散**，不方便管理和查找，这多个ZipList如何建立联系？
 - A3：Redis在3.2版本引入了新的数据结构**QucikList**，它是一个双端链表，只不过链表中的每个节点都是一个ZipList。
 - **QuickList结构**
   <img width="1283" alt="image" src="https://github.com/hhhhby/Redis/assets/113978854/5da8268b-2e36-4076-abc8-082dc88015d7">
### list-max-ziplist-size：防止entry过多
 - 为了避免QuickList中的每个ZipList中entry过多，Redis提供了一个配置项：list-max-ziplist-size来限制
   - 如果值为正，则代表ZipList的允许的entry个数的最大值
   - 如果值为负，则代表ZipList的最大内存大小，分5种情况：
     - -1：每个ZipList的内存占用不能超过4kb
     - -2：每个ZipList的内存占用不能超过8kb
     - -3：每个ZipList的内存占用不能超过16kb
     - -4：每个ZipList的内存占用不能超过32kb
     - -5：每个ZipList的内存占用不能超过64kb
   - 其默认值为-2

### list-compress-depth：对节点的ZipList做压缩
 - 通过配置list-compress-depth来控制。首尾不压缩。这个参数是控制首尾不压缩的节点个数：
   - 0：特殊值，代表不压缩
   - 1：表示QuickList的首尾各有一个节点不压缩，中间节点压缩
   - 2：表示QuickList的首尾各有两个节点不压缩，中间节点压缩
   - 默认值是0

### QuickList内部结构
 - <img width="1218" alt="image" src="https://github.com/hhhhby/Redis/assets/113978854/a688f656-2a04-46e6-81d3-253b4bdd5ce3">
 - **特点**
   - 是一个节点为ZipList的双端链表
   - 节点采用ZipList，解决了传统链表的内存占用问题
   - 控制ZipList大小，解决连续内存空间申请效率问题
   - 中间节点可压缩，进一步节省了内存
 
## SkipList
SkipList（跳表）首先是链表，但与传统链表相比有几点差异：
 
 - 元素按照升序排列存储
 - 节点可能包含多个指针，指针跨度不同
  <img width="1303" alt="image" src="https://github.com/hhhhby/Redis/assets/113978854/724a80cc-896f-4dc2-b500-a65ac5c31a63">

 - 特点
   - 跳表是一个双向链表，每个节点都包含score和ele值
   - 节点按照score值排序，score值一样则按照ele字典排序
   - 每个节点都可以包含多层指针，层数是1到32之间的随机数
   - 不同层指针到下一节点的跨度不同，层级越高，跨度越大
   - 增删改查效率与红黑树基本一致，实现却更简单

## RedisObject
Redis中的任意数据类型的键和值都会被封装为一个RedisObject，也叫做Redis对象。

 - 源码实现
   <img width="1133" alt="image" src="https://github.com/hhhhby/Redis/assets/113978854/ac89a536-48b9-4a5e-9018-b3fecf37e505">
 - String类型的内存占用较大，每个String对象都会含有一个头信息，但是一个List对象中包含多个数据，只含有一个头信息，这样会节省很多空间。
 - 11种编码方式
<img width="992" alt="image" src="https://github.com/hhhhby/Redis/assets/113978854/94b253c2-f68f-4958-ae49-f0fd1117d0a6">
 
 - 数据类型对应的编码方式
<img width="1023" alt="image" src="https://github.com/hhhhby/Redis/assets/113978854/48e7b1a7-9e31-4f0e-bb5b-350fb4584b21">


## String（字符串类型）

String是Redis中最常见的数据存储类型
 
 - 其基本编码方式是**RAW**，基于简单动态字符串（SDS）实现，存储上限为512mb。
   ![image](https://github.com/hhhhby/Redis/assets/113978854/e87d99a1-830f-4c57-ab4a-45f879e0e3e0)

 - 如果存储的SDS**长度小于44字节**，则会采用**EMBSTR编码**，此时object head与SDS是一段连续空间。申请内存时**只需要调用一次内存分配函数**，效率更高。
   ![image](https://github.com/hhhhby/Redis/assets/113978854/0d40284a-c9cc-46cb-a1c4-a6e2fd350eb0)

 - 如果存储的字符串是**整数值**，并且大小在LONG_MAX范围内，则会采用**INT编码**：直接将数据保存在RedisObject的ptr指针位置（刚好8字节），不再需要SDS了。
   ![image](https://github.com/hhhhby/Redis/assets/113978854/7276a81e-cd7c-4d2f-ab86-d4e60365fcf9)



## List
Redis的List类型可以从**首、尾**操作列表中的元素。
 
 - 符合上述特征的数据结构有
   - **LinkedList**：普通链表，可以从双端访问，**内存占用较高，内存碎片较多**
   - **ZipList**：压缩列表，可以从双端访问，它申请的内存是一片**连续**的地址，**内存占用低**，但是**存储上限低**。
   - **QuickList**：LinkedList+ZipList，可以从双端访问，每个节点都是一个ZipList,每个ZipList之间用指针链接，因此**内存占用较低**，包含多个ZipList，**存储上限高**。
 - 3.2版本之前，分别采用ZipList和LinkedList来实现List，如果元素数量小于512并且元素大小小于64字节时采用ZipList编码，超过则采用LinkedList编码。
 - 3.2版本之后，Redis统一采用QuickList来实现List。


## Set
Set是Redis中的单列集合，满足下列特点：

 - 不保证有序性
 - 保证元素唯一（可以判断元素是否存在）
 - 求交集、并集、差集

 - Set对**查询元素的效率要求非常高**
 - HashTable满足要求，也就是Redis中的Dict，Dict是双列集合（可以存键值对）
 - SkipList查询的效率也很高，但是它的一个特点是根据得分（score）排序，它是**有序**的。而Set不保证有序。

 - 为了查询效率和唯一性，set采用HT编码（Dict）。Dict中的key用来存储元素，value统一为null。**内存占用较多（指针多）**
 - 当存储的所有数据都是整数，并且元素数量不超过set-max-intset-entries时，Set会采用IntSet编码，以节省内存。每插入一个元素，都会判断是否是整数，如果不是整数，则编码方式将从OBJ_ENCODING_INTSET改为OBJ_ENCODING_HT

## ZSet
ZSet也就是SortedSet，其中每一个元素都需要指定一个score值和member值：
 
 - 可以根据score值排序
 - member必须唯一
 - 可以根据member查询分数  
因此，zset底层数据结构必须满足**键值存储、键必须唯一、可排序**这几个需求。
 - SkipList：可以排序，并且可以同时存储score和ele值（member），满足可排序
 - HT（Dict）：可以键值存储，并且可以根据key找value
 - 由于ZSet采用了两种数据结构，因此其内存占用较高（含有大量的指针）
 - 当元素不多时，HT和SkipList的优势不明显，而且更耗内存。因此ZSet还会采用ZipList结构来节省内存，不过需要同时满足两个条件：
   - 元素数量小于zset-max-ziplist-entries,默认值为128
   - 每个元素都小于zset-max-ziplist-value字节，默认值64
 - ZipList本身没有排序功能，而且没有键值对的概念，因此需要有zset通过编码实现：
   - ZipList是连续内存，因此score和element是紧挨在一起的两个entry，element在前，score在后
   - score越小越接近队首，score越大越接近队尾，按照score值升序排列。



## Hash
Hash结构与ZSet非常相似

 - 都是键值对存储
 - 都需求根据键获取值
 - 键必须唯一  

区别如下：
 
 - zset的键是member,值是score；hash的键和值都是任意值
 - zset要根据score排序；hash无需排序  

 - 因此Hash默认采用ZipList编码，用以节省内存。ZipList中相邻的两个entry分别保存field和value
 - 当数据量较大时，Hash结构会转为HT编码，也就是Dict，触发条件有两个：
   - ZipList中的元素数量超过了hash-max-ziplist-entry(默认512)
   - ZipList中的任意entry大小超过了hash-max-ziplist-value（默认64）
