# 主从

## 配置
保持主从数据库版本一致、数据一致，主数据库创建同步账号：
```sql
GRANT REPLICATION SLAVE,FILE ON *.* TO 'slave_test'@'从数据IP' IDENTIFIED BY '123456';
```
主数据配置：
```
[mysqld]
server-id=1
# 开启二进制日志
log-bin=mysql-bin
# 要同步的数据库
binlog-do-db=test_slave_db
# 要忽略的数据库
binlog-ignore-db=mysql
```

从数据配置：
```
[mysqld]
server-id=2
master-host=主数据库
# 同步账号
master-user=slave_test
master-password=123456
master-port=3306
master-connect-retry=60
# 要同步的数据库
replicate-do-db=test_slave_db
# 要忽略的数据库
replicate-ignore-db=mysql
```

登录主服务器：`show master status;`

登录从服务器：`start slave; show slave status\G`，如果`slave_io_running`和`slave_sql_running`都为yes，表明可以成功同步了  

## 原理

master中所有操作都以“事件”的方式写入二进制日志，slave中通过一个I/O线程与主服务器通信，监控master中的二进制日志文件变化，如果发现有变化，就会将变化复制到slave自己的中继日志中，slave的SQL线程会把相关事件执行到当前数据库。

`主数据库更改 -> binlog -> I/O线程 -> 中继日志 -> SQL线程 -> 从数据库`

MySQL的复制（replication）是一个异步的复制，从`Master`复制到`Slave`。三个线程完成，两个进程在Slave（SQL线程和IO线程），另外一个线程在Master（IO线程）上。

要实施复制，首先必须打开Master端的binarylog（bin-log）功能，否则无法实现。因为整个复制过程实际上就是Slave从Master端获取该日志然后再在自己身上完全顺序的执行日志中所记录的各种操作。复制的基本过程如下：

（1）Slave上面的IO进程连接上Master，并请求从指定日志文件的指定位置（或者从最开始的日志）之后的日志内容；

（2）Master接收到来自Slave的IO进程的请求后，通过负责复制的IO进程根据请求信息读取指定日志指定位置之后的日志信息，返回给Slave的IO进程。返回信息中除了日志所包含的信息之外，还包括本次返回的信息已经到Master端的bin-log文件的名称以及bin-log的位置；

（3）Slave的IO进程接收到信息后，将接收到的日志内容依次添加到Slave端的relay-log文件的最末端，并将读取到的Master端的bin-log的文件名和位置记录到master-info文件中，以便在下一次读取的时候能够清楚的告诉Master“我需要从某个bin-log的某个位置开始往后的日志内容，请发给我”；

（4）Slave的Sql进程检测到relay-log中新增加了内容后，会马上解析relay-log的内容成为在Master端真实执行时候的那些可执行的内容，并在自身执行。
   
https://www.cnblogs.com/zhenyuyaodidiao/p/4635458.html