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
     




