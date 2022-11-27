## 简介

压缩列表ziplist本质上就是一个字节数组，是Redis为了节约内存而设计的一种线性数据结构，可以包含多个元素，每个元素可以是一个字节数组或一个整数。
Redis的有序集合、散列和列表都直接或者间接使用了压缩列表。当有序集合或散列表的元素个数比较少，且元素都是短字符串时，Redis便使用压缩列表作为其底层数据存储结构。列表使用快速链表（quicklist）数据结构存储，而快速链表就是双向链表与压缩列表的组合。
ziplist 压缩列表是一个特殊编码的双端链表（内存上连续），为了尽可能节省内存而设计的。ziplist 可以存储字符串或者整数值，其中整数被编码保存为实际的整数，而不是字符数组。ziplist 支持 O(1) 的时间复杂度在列表的两端进行 push 和 pop 操作。然而因为这些操作都需要对整个 ziplist 进行内存重分配（因为是一块连续的内存），所以操作的实际复杂度和 ziplist 占用的内存大小有关。在 7.0 版本里，ziplist 已经全面被 listpack 替换了（主要是因为连锁更新较影响性能）


## 压缩列表的存储结构

Redis使用字节数组表示一个压缩列表，压缩列表结构如下所示：

```
<zlbytes> <zltail> <zllen> <entry> <entry> ... <entry> <zlend>
```

压缩列表各字段的含义如下：

- zlbytes：是一个 32 位无符号整数（4 bytes），记录整个 ziplist 占用的内存字节数，包含 4 个字节的 zlbytes 本身。
- zltail：是一个 32 位无符号整数（4 bytes），记录 ziplist 到尾节点的位置偏移量。通过这个偏移量我们可以直接定位到表尾节点，例如进行表尾的 pop 操作，不然得完整遍历 ziplist。
- zllen：是一个 16 位无符号整数（2 bytes），记录 ziplist 里的节点数量。由于它设计只用 2 个字节进行存储，2 字节实际最大可以表示为 2^16 - 1 即: 65535。当数字小于它时，则 zllen 的值就是实际的节点数量（O(1) 时间复杂度）, 也就是注释里的 2^16 - 2 的含义。否则当 zllen 值为 65535 时即 2^16-1，用它作为一个标识，表示需要完整遍历整个压缩列表 O(N) 时间复杂度才能计算出真实的节点数量。所以 ziplist 不适合存储过多元素（遍历计算节点数量开销很大，且我们假设它只用于元素数量较少的场景）。
- entryX：压缩列表存储的元素，可以是字节数组或者整数，长度不限。entry的编码结构将在后面详细介绍。
- zlend： 是一个 8 位无符号整数（1 byte），是一个特殊的标志位来标记压缩列表的结尾，0xFF(十进制表示为: 255)。其它正常节点不会有以这个字节开头的，在遍历 ziplist 的时候通过这个标记来判断是否遍历结束。

### 元素的存储结构

压缩列表元素的存储结构如下所示：
```
<prevlen> <encoding> <entry-data>
```
每一个 ziplist entry 压缩列表节点在实际的节点数据之前都会包含两部分元数据，也叫 entry header。
1. prevlen: 前置节点的字节长度，以支持我们从后往前遍历（通过指针偏移量定位前一个节点）
2. encoding: 当前节点 entry-data 节点数据部分的类型和编码，例如存储的是整数还是字符串，类型下还会细分多种编码。

有时候节点可以不用有 entry-data，可以在 encoding 部分直接存储节点数据。例如一些小整数，可以直接在 encoding 部分用几位来存储表示，对每一位都物尽其用。此时元素的存储结构为：

```
<prevlen> <encoding>
```
当前节点的前节点字节长度，prevlen 的编码方式如下（同时我们将存储 prevlen 所需的字节数为 prevlensize，即下面的 1 或者 5 字节）
- 如果前节点的字节长度 小于 254 字节，那么 prevlen 使用 1 个字节来保存它，一个 8 位无符号的整数。
 ```
 <prevlen from 0 to 253>    <encoding>    <entry>
 ```
 - `prevlen`：前一个entry长度，介于(0,253)
 - `encoding`:当前节点的实际数据类型以及长度
 - `entry`:当前节点的实际数据
- 如果前节点的字节长度 大于等于254 字节，那么 prevlen 使用 5 个字节来保存它：
 ```
 0xFE   <4 bytes unsigned little endian prevlen>    <encoding>     <entry>
 ```
 - `0xFE`: zlend 标识，值为 254（1 字节）
 - `<4 bytes unsigned little endian prevlen>`: 当前节点的实际长度（4 字节）
 - `<encoding>`: 当前节点的实际数据类型以及长度
 - `<entry>`: 当前节点的实际数据
 
压缩列表元素编码：

| encoding编码 | encoding 长度 |context类型 |
|---- |---- |---- |
| 00 pppppp  | 1 bytes | value最大长度为63字节，`pppppp`的长度为6  |
| 01 pppppp qqqqqqqq   | 2 bytes | value的最大长度16383字节  |
| 10000000 qqqqqqqq rrrrrrrr ssssssss tttttttt| 5 bytes | value的最大长度2^32-1。第一个字节的6个低位不被使用，并且被设置为零。 |
| 11000000 | 3 bytes | int 16 (2 bytes) |
| 11010000 | 5 bytes | int 32 (4 bytes) |
| 11100000 | 9 bytes | int 64 (8 bytes) |
| 11110000 | 4 bytes | 24位整数(3 bytes) |
| 11111110 | 2 bytes | 8位整数(1 byte) |
| 1111xxxx | 1 byte  | 0-12的无符号整数 |
| 11111111 | 1 byte  | ziplist结束 |

Redis 常见的encoding：

```c
#define ZIP_STR_MASK 0xc0
#define ZIP_INT_MASK 0x30
#define ZIP_STR_06B (0 << 6)
#define ZIP_STR_14B (1 << 6)
#define ZIP_STR_32B (2 << 6)
#define ZIP_INT_16B (0xc0 | 0<<4)
#define ZIP_INT_32B (0xc0 | 1<<4)
#define ZIP_INT_64B (0xc0 | 2<<4)
#define ZIP_INT_24B (0xc0 | 3<<4)
#define ZIP_INT_8B 0xfe
```

## 结构体




