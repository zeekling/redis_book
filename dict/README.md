# 简介
字典是redis数据库最基本的结构，其本质结构是hash，用来存储键值对的一种数据结构，Redis本身就是K-V数据库，
整个数据库就是用字典存储的，对于Redis数据库的增、删、改、查操作，实际上都是对字典结构的操作。

对于字典来讲，计算key的hash比较重要，下面是Redis内核里面使用的计算的hash方法: [siphash](./siphash.md) 

```c
static uint8_t dict_hash_function_seed[16];
uint64_t dictGenHashFunction(const void *key, size_t len) {
  return siphash(key,len,dict_hash_function_seed);
}
```



# 基本操作



# 字典遍历



