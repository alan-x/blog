### 0x001 修改配置
---
- 复制两份配置文件，分别命名为`redis_6378.conf`、`redis_6377.conf`。他们将在`6378`、`6377`两个端口启动
- 分别修改两个配置：
    ```
    # redis_6378.conf
    ...
    port 6378
    slaveof 127.0.0.1 6379
    ...
    ```
    ```
    # redis_6377.conf
    ...
    port 6377
    slaveof 127.0.0.1 6378
    ...
    ```
**说明：**这里使用监听6379的节点作为主节点，`6378`作为`6379`的子节点，`6377`作为`6378`的子节点。
### 0x002 同步
---
- 启动`6379`主节点
    ```
    $ redis-server
    68931:C 29 May 10:53:54.234 # oO0OoO0OoO0Oo Redis is starting                                 
    
    # 省略部分过程
          _.-``    `.  `_.  ''-._           Redis 4.0.8 (00000000/0) 64 bit
      .-`` .-```.  ```\/    _.,_ ''-._                                   
     (    '      ,       .-`  | `,    )     Running in standalone mode
     |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
     |    `-._   `._    /     _.-'    |     PID: 68931
    # 省略部分过程
    68931:M 29 May 10:53:54.237 * Ready to accept connections
    ```
- 启动`6378`子节点
    ```shell
    $ redis-server ~/Desktop/redis_6378.conf 
    # 省略部分内容
                    _._                                                  
               _.-``__ ''-._                                             
          _.-``    `.  `_.  ''-._           Redis 4.0.8 (00000000/0) 64 bit
      .-`` .-```.  ```\/    _.,_ ''-._                                   
     (    '      ,       .-`  | `,    )     Running in standalone mode
     |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6378
     |    `-._   `._    /     _.-'    |     PID: 69033
    
    # 省略部分内容
    # 这里说明启动成功了
    69033:S 29 May 10:56:24.581 * Ready to accept connections
    # 开始连接主节点
    69033:S 29 May 10:56:24.581 * Connecting to MASTER 127.0.0.1:6379
    # 开始同步
    69033:S 29 May 10:56:24.581 * MASTER <-> SLAVE sync started
    69033:S 29 May 10:56:24.581 * Non blocking connect for SYNC fired the event.
    69033:S 29 May 10:56:24.581 * Master replied to PING, replication can continue...
    # 尝试增量同步
    69033:S 29 May 10:56:24.581 * Trying a partial resynchronization (request 87953c7dc97cf1192a243fe5d7708c11acbd0e0f:1).
    # 全量同步
    69033:S 29 May 10:56:24.582 * Full resync from master: 98a56521a0a958295e499853621029d19ef6b357:0
    69033:S 29 May 10:56:24.582 * Discarding previously cached master state.
    69033:S 29 May 10:56:24.684 * MASTER <-> SLAVE sync: receiving 826 bytes from master
    69033:S 29 May 10:56:24.685 * MASTER <-> SLAVE sync: Flushing old data
    69033:S 29 May 10:56:24.685 * MASTER <-> SLAVE sync: Loading DB in memory
    69033:S 29 May 10:56:24.685 * MASTER <-> SLAVE sync: Finished with success
    ```
- 此时的`6379`主节点
    ```shell
    # 省略部分内容
    68931:M 29 May 10:53:54.237 * Ready to accept connections
    # 上面是之前的状态，下面是`6378`连接进来以后的状态
    68931:M 29 May 10:56:24.581 * Slave 127.0.0.1:6378 asks for synchronization
    68931:M 29 May 10:56:24.581 * Partial resynchronization not accepted: Replication ID mismatch (Slave asked for '87953c7dc97cf1192a243fe5d7708c11acbd0e0f', my replication IDs are 'dbf6c826156fa5140d891996afcf8cc4f21b61fd' and '0000000000000000000000000000000000000000')
    68931:M 29 May 10:56:24.581 * Starting BGSAVE for SYNC with target: disk
    68931:M 29 May 10:56:24.582 * Background saving started by pid 69034
    69034:C 29 May 10:56:24.584 * DB saved on disk
    68931:M 29 May 10:56:24.684 * Background saving terminated with success
    # 同步成功
    68931:M 29 May 10:56:24.684 * Synchronization with slave 127.0.0.1:6378 succeeded
    ```
- 启动`6377`子节点
    ```
    redis-server ~/Desktop/redis_6377.conf 
    # 省略部分内容
                    _._                                                  
               _.-``__ ''-._                                             
          _.-``    `.  `_.  ''-._           Redis 4.0.8 (00000000/0) 64 bit
      .-`` .-```.  ```\/    _.,_ ''-._                                   
     (    '      ,       .-`  | `,    )     Running in standalone mode
     |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6377
     |    `-._   `._    /     _.-'    |     PID: 69207
    
    # 启动完成
    69207:S 29 May 11:02:22.569 * Ready to accept connections
    # 连接上一级节点
    69207:S 29 May 11:02:22.569 * Connecting to MASTER 127.0.0.1:6378
    69207:S 29 May 11:02:22.570 * MASTER <-> SLAVE sync started
    69207:S 29 May 11:02:22.570 * Non blocking connect for SYNC fired the event.
    69207:S 29 May 11:02:22.570 * Master replied to PING, replication can continue...
    69207:S 29 May 11:02:22.570 * Trying a partial resynchronization (request 98a56521a0a958295e499853621029d19ef6b357:1).
    69207:S 29 May 11:02:22.570 * Successful partial resynchronization with master.
    69207:S 29 May 11:02:22.570 * MASTER <-> SLAVE sync: Master accepted a Partial Resynchronization.
    ```
### 0x003数据写入
---
- 使用`redis-cli`向`6379`主节点写入数据
    ```
    $ redis-cli -h 127.0.0.1 -p 6379
    127.0.0.1:6379> set name redis
    OK
    127.0.0.1:6379> get name
    "redis"
    ```
- 使用`redis-cli`连接`6378`子节点并读取在主节点写入的数据
    ```
    $ redis-cli -h 127.0.0.1 -p 6378
    127.0.0.1:6378> get name
    "redis"
    ```
- 使用`redis-cli`连接`637 7`子节点并读去在主节点写入的数据
    ```
    $ redis-cli -h 127.0.0.1 -p 6377
    127.0.0.1:6377> get name
    "redis"
    ```
- 使用`redis-cli`连接`6378`子节点并尝试写入数据
    ```
    127.0.0.1:6378> set name redis-cli
    (error) READONLY You can't write against a read only slave.
    ```
**说明：**子节点可以读取到主节点写入的数据，说明同步成功了，子节点无法写入数据，因为子节点是默认只读的，修改配置可以做到子节点可读写
### 0x004 断开连接
---
- 关闭`6378子节点`
    ```
    $ redis-cli -h 127.0.0.1 -p 6378
    127.0.0.1:6378> shutdown
    not connected> 
    ```
- 此时的`6378`
    ```
    # 上面是之前的状态，下面是`6378`关机时候的状态
    9033:S 29 May 11:09:54.587 # User requested shutdown...
    69033:S 29 May 11:09:54.587 * Saving the final RDB snapshot before exiting.
    69033:S 29 May 11:09:54.588 * DB saved on disk
    69033:S 29 May 11:09:54.588 * Removing the pid file.
    69033:S 29 May 11:09:54.589 # Redis is now ready to exit, bye bye...
    ```
    - 此时的`6379`
    ```
    # 上面是之前的状态，下面是`6378`关机之后的状态
    68931:M 29 May 11:09:54.589 # Connection with slave 127.0.0.1:6378 lost.
    ```
- 此时的`6377`
    ```
    ...
    # 6377 会一直尝试重新连接`6378`
    69219:S 29 May 11:12:07.132 * Connecting to MASTER 127.0.0.1:6378
    69219:S 29 May 11:12:07.132 * MASTER <-> SLAVE sync started
    69219:S 29 May 11:12:07.132 # Error condition on socket for SYNC: Connection refused
    69219:S 29 May 11:12:07.132 * Connecting to MASTER 127.0.0.1:6378
    69219:S 29 May 11:12:07.132 * MASTER <-> SLAVE sync started
    69219:S 29 May 11:12:07.132 # Error condition on socket for SYNC: Connection refused
    ...
    ```
### 0x005 子节点可读
---
- 修改配置
    ```
    # redis_6378.conf
    slave-read-only no
    ```
- 重新启动`6378`，并向`6378`写入数据
    ```
    $ redis-cli -h 127.0.0.1 -p 6378
    # 此时配置子节点不是只读的，所以可以写入
    127.0.0.1:6378> set name redis-cli
    OK
    127.0.0.1:6378> get name 
    "redis-cli"
    ```

- 连接`6377`并读取`6378`写入的`key`
    ```
    $ redis-cli -h 127.0.0.1 -p 6377
    127.0.0.1:6377> get name
    "redis"
    ```
- 连接`6379`并读取`6378`写入的`key`
    ```
    $ redis-cli -h 127.0.0.1 -p 6379
    127.0.0.1:6379> GET name
    "redis"
    ```
**说明：**虽然此时的`6378`子节点是可读的，但是并不会传播到其他子节点或者主节点，并且在下次同步的时候，将会被删除。
