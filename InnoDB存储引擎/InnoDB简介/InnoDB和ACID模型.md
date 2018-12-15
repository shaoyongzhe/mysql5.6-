# InnoDB和ACID模型

ACID模型是一组数据库设计原则，强调可靠性方面，这些方面对业务数据和任务关键型应用程序非常重要。MySQL包括InnoDB存储引擎等组件，它们与ACID模型紧密结合，因此数据不会被破坏，并且结果不会因软件崩溃和硬件故障等特殊情况而失真。当您依赖符合ACID的功能时，您无需重新发明一致性检查和崩溃恢复机制。如果您有其他软件安全措施，超可靠硬件或可以容忍少量数据丢失或不一致的应用程序，您可以调整MySQL设置以牺牲一些ACID可靠性以获得更高的性能或吞吐量。

以下部分讨论MySQL的功能，特别是InnoDB存储引擎，如何与ACID模型的类别进行交互：
* A: 原子性。
* C: 一致性。
* I: 隔离性。
* D: 持久性。

## 原子性
ACID模型的原子性方面主要涉及InnoDB`事务`。相关的MySQL功能包括：

* 自动提交设置。

* COMMIT语句。

* ROLLBACK声明。

* 从INFORMATION_SCHEMA表中操作数据。

## 一致性

ACID模型的一致性方面主要涉及内部InnoDB处理以保护数据免于崩溃。相关的MySQL功能包括：

* InnoDB `doublewrite buffer`。

* InnoDB `crash recovery`。

## 隔离型性

ACID模型的隔离方面主要涉及InnoDB事务，特别是适用于每个事务的隔离级别。相关的MySQL功能包括：

* 自动提交设置。

* SET ISOLATION LEVEL语句。

* InnoDB锁定的低级细节。在性能调优期间，您可以通过INFORMATION_SCHEMA表查看这些详细信息。

## 持久性
ACID模型的持久性方面涉及MySQL软件功能以及与您的特定的硬件配置之间的交互。
取决于您的CPU，网络和存储设备的功能，持久性会产生多种可能性。因此，在提供具体的指导方针这一点上，持久性也是最复杂的。（这些指导方针可能采取购买“新硬件”的形式）。相关的MySQL功能包括：
* InnoDB `doublewrite buffer`，通过配置选项`innodb_doublewrite` 打开或者关闭。
* 配置选项 `innodb_flush_log_at_trx_commit`。
* 配置选项 `sync_binlog`。
* 配置选项 `innodb_file_per_table`。
* 写缓存到存储设备，例如磁盘驱动器，SSD或RAID阵列。
* 存储设备中的电池备份缓存。
* 运行MySQL的操作系统，最好支持fsync() 系统调用。
* 无间断电源（UPS）保护运行MySQL服务器和存储MySQL数据的所有计算机服务器和存储设备的电源。
* 您的备份策略，例如备份的频率和类型以及备份保留期。
* 对于分布式或托管数据应用程序，MySQL服务器的硬件所在的数据中心的特定特征，以及数据中心之间的网络连接。
