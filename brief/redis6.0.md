
## Redis 6.0的新特性

Redis 6.0版本基于Redis 5.0支持了很多特性，主要是针对于性能的优化。主要包含下面方面：

- (1) 客户端缓存：将访问频繁的key缓存到客户端侧，降低客户端访问Redis服务端的次数，从而达到优化性能的效果，可以有效缓解热key的问题。
- (2) 多线程I/O：对于Redis而言，主要瓶颈在于CPU和网络IO，所以在Redis 6.0的版本当中，将读写Redis命令的部分优化成了多线程模式，加快了命令读取的速度。
- (3) ACL权限控制：在之前的版本中，Redis只有一个默认用户，没有权限管理的概念，在6.0当中，增加了权限管理。
- (4) 通道加密：在Redis 6.0版本中，增加了对于安全方面的考虑，支持了通道加密；但是在开启通道加密时候会带来性能损耗，经过作者测试大概能损耗40%+，开启需谨慎。

