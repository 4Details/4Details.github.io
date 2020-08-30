---
title: 讨论社区24  -  Redis入门
date: 2019-07-25 23:12:41
tags: [Redis,NoSQL]
category: [讨论社区项目,编程笔记]
---

- `Redis`是一款基于键值对的`NoSQL`数据库，它的值支持多种数据结构：
  字符串(strings)、哈希(hashes)、列表(lists)、集合(sets)、有序集合(sorted sets)等。
> redis是一个key-value存储系统。和Memcached类似，它支持存储的value类型相对更多，包括string(字符串)、list(链表)、set(集合)、zset(sorted set --有序集合)和hash（哈希类型）。
>
> 这些数据类型都支持push/pop、add/remove及取交集并集和差集及更丰富的操作，而且这些操作都是原子性的。在此基础上，redis支持各种不同方式的排序。与memcached一样，为了保证效率，数据都是缓存在内存中。区别的是redis会周期性的把更新的数据写入磁盘或者把修改操作写入追加的记录文件，并且在此基础上实现了master-slave(主从)同步。

- `Redis`将所有的数据都存放在内存中，所以它的读写性能十分惊人。
  同时，`Redis`还可以将内存中的数据以快照或日志的形式保存到硬盘上，以保证数据的安全性。

> RDB  体积小。以快照形式存储到硬盘比较耗时，也会对进程造成阻塞，故可以隔一段时间执行
>
> AOF  可以实时存储。以日志形式存储`Redis`命令，是以追加日志的方式，占磁盘空间，恢复则是重复执行所有的记录命令，恢复速度较差。

- `Redis`典型的应用场景包括：缓存、排行榜、计数器、社交网络、消息队列等。



#  Linux安装

1. 官网下载压缩包安装  http://download.redis.io/releases/redis-3.0.0.tar.gz
2. 通过wget命令下载安装

```shell
# 下载之后解压到/usr/local路径，进入redis文件夹，make编译安装
wget http://download.redis.io/releases/redis-3.0.0.tar.gz
tar -xvzf redis-3.0.0.tar.gz /usr/local
cd redis-3.0.0
make PREFIX=/usr/local/redis install
```

启动redis服务端

```shell
# 前台启动，终端关闭，服务端就关闭
./redis-server
# 推荐使用下面的后台启动，终端关闭，服务端不会关闭
# 修改conf文件，将daemonize改为yes
vim /usr/local/redis/bin/redis.conf
# 启动redis服务端
cd /usr/local/redis
./bin/redis-server ./redis.conf
```

启动redis客户端

```shell
/usr/local/redis/bin/redis-cli 
```

#  Windows 安装

https://github.com/microsoftarchive/redis/releases/download/win-3.2.100/Redis-x64-3.2.100.msi

一路点击下一步即可安装，建议将安装路径添加到环境变量。

cmd  ` $ redis-cli `  启动redis客户端

#  使用

##  redis键值对的管理和操作

```shell
1 DEL key
该命令用于在 key 存在时删除 key。
2 DUMP key
序列化给定 key ，并返回被序列化的值。
3 EXISTS key
检查给定 key 是否存在。
4 EXPIRE key seconds
为给定 key 设置过期时间。
5 EXPIREAT key timestamp
EXPIREAT 的作用和 EXPIRE 类似，都用于为 key 设置过期时间。 不同在于 EXPIREAT 命令接受的时间参数是 UNIX 时间戳(unix timestamp)。
6 PEXPIRE key milliseconds
设置 key 的过期时间以毫秒计。
7 PEXPIREAT key milliseconds-timestamp
设置 key 过期时间的时间戳(unix timestamp) 以毫秒计
8 KEYS pattern
查找所有符合给定模式( pattern)的 key 。
9 MOVE key db
将当前数据库的 key 移动到给定的数据库 db 当中。
10 PERSIST key
移除 key 的过期时间，key 将持久保持。
11 PTTL key
以毫秒为单位返回 key 的剩余的过期时间。
12 TTL key
以秒为单位，返回给定 key 的剩余生存时间(TTL, time to live)。
13 RANDOMKEY
从当前数据库中随机返回一个 key 。
14 RENAME key newkey
修改 key 的名称
15 RENAMENX key newkey
仅当 newkey 不存在时，将 key 改名为 newkey 。
16 TYPE key
返回 key 所储存的值的类型。
```

##  redis哈希的操作

```
HDEL key field2 [field2] 
删除一个或多个哈希表字段
HEXISTS key field 
查看哈希表 key 中，指定的字段是否存在。
HGET key field 
获取存储在哈希表中指定字段的值。
HGETALL key 
获取在哈希表中指定 key 的所有字段和值
HINCRBY key field increment 
为哈希表 key 中的指定字段的整数值加上增量 increment 。
HINCRBYFLOAT key field increment 
为哈希表 key 中的指定字段的浮点数值加上增量 increment 。
HKEYS key 
获取所有哈希表中的字段
HLEN key 
获取哈希表中字段的数量
HMGET key field1 [field2] 
获取所有给定字段的值
HMSET key field1 value1 [field2 value2 ] 
同时将多个 field-value (域-值)对设置到哈希表 key 中。
HSET key field value 
将哈希表 key 中的字段 field 的值设为 value 。
HSETNX key field value 
只有在字段 field 不存在时，设置哈希表字段的值。
HVALS key 
获取哈希表中所有值HSCAN key cursor [MATCH pattern] [COUNT count] 迭代哈希表中的键值对。
```

## redis列表操作

```
BLPOP key1 [key2 ] timeout 
移出并获取列表的第一个元素， 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。
BRPOP key1 [key2 ] timeout 
移出并获取列表的最后一个元素， 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。
BRPOPLPUSH source destination timeout 
从列表中弹出一个值，将弹出的元素插入到另外一个列表中并返回它； 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。
LINDEX key index 
通过索引获取列表中的元素
LINSERT key BEFORE|AFTER pivot value 
在列表的元素前或者后插入元素
LLEN key 
获取列表长度
LPOP key 
移出并获取列表的第一个元素
LPUSH key value1 [value2] 
将一个或多个值插入到列表头部
LPUSHX key value 
将一个或多个值插入到已存在的列表头部
LRANGE key start stop 
获取列表指定范围内的元素
LREM key count value 
移除列表元素
LSET key index value 
通过索引设置列表元素的值
LTRIM key start stop 
对一个列表进行修剪(trim)，就是说，让列表只保留指定区间内的元素，不在指定区间之内的元素都将被删除。
RPOP key 
移除并获取列表最后一个元素
RPOPLPUSH source destination 
移除列表的最后一个元素，并将该元素添加到另一个列表并返回
RPUSH key value1 [value2] 
在列表中添加一个或多个值
RPUSHX key value 
为已存在的列表添加值
```

##  redis集合

```
SADD key member1 [member2] 
向集合添加一个或多个成员
SCARD key 
获取集合的成员数
SDIFF key1 [key2] 
返回给定所有集合的差集
SDIFFSTORE destination key1 [key2] 
返回给定所有集合的差集并存储在 destination 中
SINTER key1 [key2] 
返回给定所有集合的交集
SINTERSTORE destination key1 [key2] 
返回给定所有集合的交集并存储在 destination 中
SISMEMBER key member 
判断 member 元素是否是集合 key 的成员
SMEMBERS key 
返回集合中的所有成员
SMOVE source destination member 
将 member 元素从 source 集合移动到 destination 集合
SPOP key 
移除并返回集合中的一个随机元素
SRANDMEMBER key [count] 
返回集合中一个或多个随机数
SREM key member1 [member2] 
移除集合中一个或多个成员
SUNION key1 [key2] 
返回所有给定集合的并集
SUNIONSTORE destination key1 [key2] 
所有给定集合的并集存储在 destination 集合中
SSCAN key cursor [MATCH pattern] [COUNT count] 
迭代集合中的元素
```

##  有序集合

```shell
ZADD key score1 member1 [score2 member2] 
向有序集合添加一个或多个成员，或者更新已存在成员的分数
ZCARD key 
获取有序集合的成员数
ZCOUNT key min max 
计算在有序集合中指定区间分数的成员数
ZINCRBY key increment member 
有序集合中对指定成员的分数加上增量 increment
ZINTERSTORE destination numkeys key [key ...] 
计算给定的一个或多个有序集的交集并将结果集存储在新的有序集合 key 中
ZLEXCOUNT key min max 
在有序集合中计算指定字典区间内成员数量
ZRANGE key start stop [WITHSCORES] 
通过索引区间返回有序集合成指定区间内的成员
ZRANGEBYLEX key min max [LIMIT offset count] 
通过字典区间返回有序集合的成员
ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT] 
通过分数返回有序集合指定区间内的成员
ZRANK key member 
返回有序集合中指定成员的索引
ZREM key member [member ...] 
移除有序集合中的一个或多个成员
ZREMRANGEBYLEX key min max 
移除有序集合中给定的字典区间的所有成员
ZREMRANGEBYRANK key start stop 
移除有序集合中给定的排名区间的所有成员
ZREMRANGEBYSCORE key min max 
移除有序集合中给定的分数区间的所有成员
ZREVRANGE key start stop [WITHSCORES] 
返回有序集中指定区间内的成员，通过索引，分数从高到底
ZREVRANGEBYSCORE key max min [WITHSCORES] 
返回有序集中指定分数区间内的成员，分数从高到低排序
ZREVRANK key member 
返回有序集合中指定成员的排名，有序集成员按分数值递减(从大到小)排序
ZSCORE key member 
返回有序集中，成员的分数值
ZUNIONSTORE destination numkeys key [key ...] 
计算给定的一个或多个有序集的并集，并存储在新的 key 中
ZSCAN key cursor [MATCH pattern] [COUNT count] 
迭代有序集合中的元素（包括元素成员和元素分值）
```

##  redis事务的实现

```
DISCARD 
取消事务，放弃执行事务块内的所有命令。
EXEC 
执行所有事务块内的命令。
MULTI 
标记一个事务块的开始。
UNWATCH 
取消 WATCH 命令对所有 key 的监视。
WATCH key [key ...] 
监视一个(或多个) key ，如果在事务执行之前这个(或这些) key 被其他命令所改动，那么事务将被打断。
```





https://redis.io

https://github.com/microsoftarchive/redis