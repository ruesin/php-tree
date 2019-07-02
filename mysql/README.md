# MySQL

## 逻辑架构和存储引擎

- 连接层：处理客户端连接、授权认证等
- 服务器层：查询语句的解析、优化、缓存以及内置函数的实现、存储过程等。
- 存储引擎：数据的存储和提取。MySQL中服务器层不管理事务，事务是由存储引擎实现的。

## 日志
MySQL 的日志有很多种，如二进制日志、错误日志、查询日志、慢查询日志、事务日志等。

InnoDB 存储引擎提供两种事务日志：
- redo log(重做日志)：用于保证事务持久性
- undo log(回滚日志)：事务原子性和隔离性实现的基础

- [索引](/mysql/index.md)
- [事务](/mysql/transaction.md)
- [主从](/mysql/replication.md)
- [分区](/mysql/partition.md)
- [引擎](/mysql/engine.md)
- [具体问题](/mysql/question.md)