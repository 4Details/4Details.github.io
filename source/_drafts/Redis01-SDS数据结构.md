---
title: Redis01 sds数据结构
tags: [Redis,sds]
category: []
---

Redis 底层使用c++语言实现，本文为基本数据结构 SDS（Simple Dynamic String）学习笔记。

> - Redis内部使用sds作为默认的字符串类型，sds在c的字符串前添加了额外的信息（sdshr）
>
> - sdshr中len字段记录了字符串长度，可以O(1)时间获得字符串长度
>
> - sds同时使用sdshr中的len字段来判断字符串是否结束，实现了二进制安全（c中字符串结束"\0"）
>
> - 修改sds时会按需分配大小，可以杜绝缓冲区溢出
>
> - sds空间扩展会进行空间预分配和惰性释放，可以减少内存分配次数。
>
>   空间预分配策略：
>
>   - 修改之后SDS长度<1M，每次为len*2（len和free各一半）
>   - 修改之后SDS长度>1M，每次为len+1M
>   - 注意，以上两种策略最后都会多加1byte存储空字符。
>
>   惰性释放：当sds缩短保存的字符串时，并不会立即重新分配内存回收的字节，而是使用free将这些字节的数量记录起来，等待下一次使用。
>
> - sds支持部分c++语言标准库中的字符串操作函数，在末尾设置空字符，即可重用string.h库定义的函数。

```c++
typedef char *sds;

/* Note: sdshdr5 is never used, we just access the flags byte directly.
 * However is here to document the layout of type 5 SDS strings. */
struct __attribute__ ((__packed__)) sdshdr5 {
    unsigned char flags; /* 3 lsb of type, and 5 msb of string length */
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr8 {
    uint8_t len; /* used */
    uint8_t alloc; /* excluding the header and null terminator */
    unsigned char flags; /* 3 lsb of type, 5 unused bits */
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr16 {
    uint16_t len; /* used */
    uint16_t alloc; /* excluding the header and null terminator */
    unsigned char flags; /* 3 lsb of type, 5 unused bits */
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr32 {
    uint32_t len; /* used */
    uint32_t alloc; /* excluding the header and null terminator */
    unsigned char flags; /* 3 lsb of type, 5 unused bits */
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr64 {
    uint64_t len; /* used */
    uint64_t alloc; /* excluding the header and null terminator */
    unsigned char flags; /* 3 lsb of type, 5 unused bits */
    char buf[];
};
```

```c++
sds sdsnewlen(const void *init, size_t initlen) { //使用init中长度为initlen的字符串，创建sdshdr
    void *sh;
    sds s;
    char type = sdsReqType(initlen);//根据字符串的长度获取要创建的sds的类型
    /* Empty strings are usually created in order to append. Use type 8
     * since type 5 is not good at this. */
    // 初始化使用SDS_TYPE_5，现已不适用， 默认使用SDS_TYPE_8
    if (type == SDS_TYPE_5 && initlen == 0) type = SDS_TYPE_8;
    int hdrlen = sdsHdrSize(type);//根据类型获取头的大小
    unsigned char *fp; /* flags pointer. */

    sh = s_malloc(hdrlen+initlen+1);//分配空间，总长度 = 头部+字符串长度+给字符串结尾的"\0"预留1字节
    if (sh == NULL) return NULL;
    if (init==SDS_NOINIT) //SDS_NOINIT是特殊情况，表示不初始化
        init = NULL;
    else if (!init)
        memset(sh, 0, hdrlen+initlen+1);//如果init是null，则直接分配空间
    s = (char*)sh+hdrlen; //s指向sdshdr的buf的位置
    fp = ((unsigned char*)s)-1;  //s - 1 其实是flags的位置， *fp就是 sdshdr->flags字段
    // 根据字符的类型初始化成员变量(len， alloc， flags)，SDS_HDR_VAR的作用是定义 
    // sh临时变量（sdshdr*）指向sdshdr的起始位置，用于后面取len和alloc赋值
    switch(type) {
        case SDS_TYPE_5: {
            *fp = type | (initlen << SDS_TYPE_BITS);
            break;
        }
        case SDS_TYPE_8: {
            SDS_HDR_VAR(8,s);
            sh->len = initlen;
            sh->alloc = initlen;
            *fp = type;
            break;
        }
        case SDS_TYPE_16: {
            SDS_HDR_VAR(16,s);
            sh->len = initlen;
            sh->alloc = initlen;
            *fp = type;
            break;
        }
        case SDS_TYPE_32: {
            SDS_HDR_VAR(32,s);
            sh->len = initlen;
            sh->alloc = initlen;
            *fp = type;
            break;
        }
        case SDS_TYPE_64: {
            SDS_HDR_VAR(64,s);
            sh->len = initlen;
            sh->alloc = initlen;
            *fp = type;
            break;
        }
    }
    if (initlen && init)
        memcpy(s, init, initlen);
    s[initlen] = '\0'; //将buf的最后一个字节置0
    return s;
}
```

