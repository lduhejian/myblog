---
title: "GTID"
date: 2018-08-19T20:09:15+08:00
draft: true
---

GTID(global transaction identifier) 在主服务器 master 上创建，唯一的表示主服务器上的一个 transaction commit。GTID 不仅在创建他的 server 上唯一，他在整个主从复制拓扑上都唯一。

GTID 可以区分不同的事务，他知道哪些是在 master 上提交的，哪些是 replicated transaction（为了实现主从复制，将 master 的事务应用到 slave 上），哪些是 slave 上提交的。当 client transaction 提交到 master, 这个事务将会被写入二进制日志之后，一个新的 GTID 将会被分配。GTID 会是一个增长的数字，数字之间没有间隙。如果一个事务没有被写入到二进制日志中，GTID 将不会被分配。

replicated transaction 将会和在 master 上产生时候的 GTID 保持一致。The GTID is present before the replicated transaction begins to execute, and is persisted even if the replicated transaction is not written to the binary log on the slave, or is filtered out on the slave. The MySQL system table mysql.gtid_executed is used to preserve the assigned GTIDs of all the transactions applied on a MySQL server, except those that are stored in a currently active binary log file.

auto-skip 函数用来过滤 master 产生的事务，使得 slave 对于同一 GTID 的事务只会执行一次，来保证主从服务器数据的一致性。一旦给定 GTID 的事务已经在 slave 上执行过了, 之后再有相同 GTID 的事务到来, slave 会忽略掉他（不会报错，事务不会被执行）。

如果事务正在服务器上执行但是还没有提交或回滚，尝试开启一个拥有相同 GTID 的并行事务，那么这个并行事务将会被阻塞。服务器不会开始执行并行事务，也不会给 client 返回数据。一旦前一个事务提交或者回滚了，这个并行事务才开始被处理。如果前一个事务回滚了，并行事务就开始执行。如果前一个事务要提交了，并行事务将会被忽略。

GTID 是一对坐标值来组成的，中间以冒号分割：

```
GTID = source_id:transaction_id
```

source_id 标示 originating server。通常是服务器的 uuid。  
transaction_id 是该服务器上依据提交顺序确定的序列号，他是依次增加的。例如，第一个提交的事务 transaction_id = 1, 第十个提交的事务 transaction_id = 10。例如，一个服务器他的 uuid 是 3E11FA47-71CA-11E1-9E33-C80AA9429562，那么在服务器上提交的第 23 个事务的 GTID 将会是：

```
3E11FA47-71CA-11E1-9E33-C80AA9429562:23
```

在 SHOW_SLAVE_STATUS，SHOW BINLOG EVENTS，mysqlbinlog --base64-output=DECODE-ROWS  命令的输出和二进制日志中都可以看到这种格式的 GTID。

在一些命令的输出中我们往往会将一系列的 GTID 组合成一个语句来展示，例如  SHOW MASTER STATUS 和 SHOW SLAVE：

```
3E11FA47-71CA-11E1-9E33-C80AA9429562:1-5
```

上面的语句表示，uuid 是 3E11FA47-71CA-11E1-9E33-C80AA9429562 的服务器上的第一到第五个事务。

这种格式也被用作命令参数选项：START_SLAVE 下面的 SQL_BEFORE_GTIDS 和 SQL_AFTER_GTIDS。
