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
 



