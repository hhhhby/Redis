<img width="327" alt="image" src="https://github.com/hhhhby/Redis/assets/113978854/b62af4a5-5514-4863-979d-c052d05232bf"># Redis
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
```


