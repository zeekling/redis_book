
可以通过官方网站：https://redis.io/download/#redis-downloads 下载想要版本的Redis，或者从GitHub上面下载想要版本的Redis。本文以GitHub方式为例(操作系统以debian为例)：

```bash
sudo apt install libssl-dev
git clone git@github.com:redis/redis.git
cd redis
git checkout 7.0
make BUILD_TLS=yes -j8
make install 
```

安装好之后可以看到可执行文件在/usr/local/bin文件夹下面:

```bash
redis-benchmark redis-check-aof redis-check-rdb redis-cli redis-sentinel redis-server 
```

- redis-benchmark：官方提供的性能测试工具。
- redis-check-aof：可以用来检查AOF文件是否损坏。
- redis-check-rdb：可以用来检查RDB文件是否损坏。
- redis-cli:官方提供的客户端，用来连接Redis服务，支持集群模式和单实例模式；使用-p指定端口，-h指定ip，-c使用集群模式连接，--tls使用通道加密模式连接。
- redis-sentinel:哨兵模式的启动程序。
- redis-server:Redis服务启动程序，可以直接运行redis-server启动Redis服务，如下

```bash 

57532:C 27 Nov 2022 18:44:03.287 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
57532:C 27 Nov 2022 18:44:03.287 # Redis version=255.255.255, bits=64, commit=f36eb5a1, modified=0, pid=57532, just started
57532:C 27 Nov 2022 18:44:03.287 # Warning: no config file specified, using the default config. In order to specify a config file use ./src/redis-server /path/to/redis.conf
57532:M 27 Nov 2022 18:44:03.288 * Increased maximum number of open files to 10032 (it was originally set to 1024).
57532:M 27 Nov 2022 18:44:03.288 * monotonic clock: POSIX clock_gettime
                _._                                                  
           _.-``__ ''-._                                             
      _.-``    `.  `_.  ''-._           Redis 255.255.255 (f36eb5a1/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._                                  
 (    '      ,       .-`  | `,    )     Running in standalone mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
 |    `-._   `._    /     _.-'    |     PID: 57532
  `-._    `-._  `-./  _.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |           https://redis.io       
  `-._    `-._`-.__.-'_.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |                                  
  `-._    `-._`-.__.-'_.-'    _.-'                                   
      `-._    `-.__.-'    _.-'                                       
          `-._        _.-'                                           
              `-.__.-'                                               

57532:M 27 Nov 2022 18:44:03.290 # Server initialized
57532:M 27 Nov 2022 18:44:03.290 # WARNING Memory overcommit must be enabled! Without it, a background save or replication may fail under low memory condition. Being disabled, it can can also cause failures without low memory condition, see https://github.com/jemalloc/jemalloc/issues/1328. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
57532:M 27 Nov 2022 18:44:03.302 * Loading RDB produced by version 255.255.255
57532:M 27 Nov 2022 18:44:03.302 * RDB age 6040 seconds
57532:M 27 Nov 2022 18:44:03.302 * RDB memory usage when created 0.97 Mb
57532:M 27 Nov 2022 18:44:03.302 * Done loading RDB, keys loaded: 3, keys expired: 0.
57532:M 27 Nov 2022 18:44:03.302 * DB loaded from disk: 0.012 seconds
57532:M 27 Nov 2022 18:44:03.302 * Ready to accept connections tcp 
```


