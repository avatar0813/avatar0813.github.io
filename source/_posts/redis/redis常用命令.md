---
title: redis的常用命令
date: 2023-06-25 11:55:37
tags: [Redis]
index_img: /img/redis.svg
---
## redis的常用命令

了解redis的常用命令之前先了解一下redis：redis是一个运行在内存中，存储内容为键值对形式的一个高效的非关系性数据库，redis存储的值类型有很多：`string`，`list`， `set`，`sorted set`，`hash`， `geospatial`，`bitmap` 等。

### string类型的相关操作

- `SET` stores a string value 存储(已存在则修改)一个值.
- `SETNX` stores a string value only if the key doesn't already exist. Useful for implementing locks 存储一个值（只有当这个key不存在的时候）.
- `GET` retrieves a string value 检索一个值.
- `MGET` retrieves multiple string values in a single operation 一次检索多个key.
- `INCRBY` atomically increments (and decrements when passing a negative number) counters stored at a given key 根据一个数字自增一个key的value, `INCR` 以`1`为单位自增.

### list类型的相关操作

- `LRANGE` 查看list  lrange list1 0 -1
- `LPUSH` adds a new element to the head of a list; RPUSH adds to the tail 头部添加一个元素， RPUSH 是在尾部添加一个元素.
- `LPOP` removes and returns an element from the head of a list; RPOP does the same but from the tails of a list 头部删除一个元素， RPOP 是在尾部添加一个元素.
- `LLEN` returns the length of a list 返回list的长度.
- `LMOVE` atomically moves elements from one list to another 将list1 的一个元素移动的list2中去 `LMOVE list1 list2 LEFT LEFT`.
- `LTRIM` reduces a list to the specified range of elements 限制list的长度 `LTRIM list1 0 1`.

### set类型的相关操作

- `SADD` adds a new member to a set 添加一个成员到set中.
- `SREM` removes the specified member from the set 删除特定的成员.
- `SISMEMBER` tests a string for set membership 查询set是否包含一个成员.
- `SINTER` returns the set of members that two or more sets have in common (i.e., the intersection) 返回一个set与2个或多个set的交集.

```bash
> sadd set1 1 2 3 4
(integer) 4
> sadd set2 1 2 3 5
(integer) 4

> SINTER set1 set2
1) "1"
2) "2"
3) "3"
```

- `SCARD` returns the size (a.k.a. cardinality) of a set 返回set的大小.
