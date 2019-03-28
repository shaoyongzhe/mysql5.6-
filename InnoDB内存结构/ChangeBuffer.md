# 14.5.2 Change Buffer

change buffer 是一种特殊的数据结构，主要针对的是不在缓冲池中的二级索引页。 缓冲的更改（可能由INSERT，UPDATE或DELETE操作（DML）引起）并不会立即合并，而是该页面通过其它读操作加载到缓冲池中时合并。
图14.3 Change Buffer
![缓存池列表](../images/innodb-change-buffer.png)

与聚簇索引不同，二级索引通常不是唯一的，并且插入二级索引的顺序相对随机。 同样，删除和更新可能会影响不在索引树中相邻的二级索引页。 当受影响的页面被其他操作读入缓冲池时，合并缓存的更改，避免了从磁盘读取二级索引页到缓冲池所需的大量随机访问I/O.

在系统空闲时或在mysql正常关闭时，purge操作会定期将更新的索引页写入磁盘。 对于一系列的被更新的索引值，与每个值立即写入磁盘相比，purge操作写入磁盘块的效率更高。

当有许多受影响的行和许多要更新的辅助索引时，change buffer的merge操作可能需要几个小时。 在此期间，磁盘I/O会增加，这会导致磁盘相关查询显着减慢。 在提交事务之后，甚至在服务器关闭并重新启动之后，change buffer的merge也可能继续（有关更多信息，请参见第14.21.2节“强制InnoDB恢复”）。

在内存中，更改缓冲区占用缓冲池的一部分。 在磁盘上，change-buffer是系统表空间的一部分(数据库服务器关闭时，变更的索引缓存在系统表空间中)。

change buffer中缓存的数据类型由innodb_change_buffering变量控制。 有关更多信息，请参阅配置更改缓冲。 您还可以配置最大change buffer容量。 有关更多信息，请参阅配置change buffer最大容量。

如果二级索引的索引中包含一个降序的索引列或者主键包含了一个降序索引列，change buffer不支持这种类型的二级索引。 **为什么呢？**
有关更改缓冲区的常见问题解答，请参见第A.15节“MySQL 5.7 FAQ：InnoDB更改缓冲区”。

## 配置change buffer

 当对表执行INSERT，UPDATE和DELETE操作时，索引列的值（特别是辅助键的值）通常是无序的，需要大量I/O才能使二级索引保持最新。 当相关页面不在缓冲池中时，change buffer会缓存对二级索引条目的更改，从而避免了立即从磁盘读取页面进行的昂贵的I/O操作。 当页面加载到缓冲池中时，将合并缓冲的更改，稍后将更新的页面刷新到磁盘。InnoDB主线程在服务器几乎空闲时以及在慢速关闭期间合并缓冲的更改。

 因为它可以减少磁盘读取和写入，所以change buffer 功能对于I/O-bound的工作负载最有价值，例如具有大量DML操作的应用程序（如批量插入）。

 但是，change buffer占用缓冲池的一部分，从而减少了可用于缓存数据页的内存。如果woking set几乎都适用于缓冲池，或者您的表具有相对较少的二级索引，则禁用change buffer可能很有用。 如果working data set完全适合缓冲池，则change buffer不会产生额外开销，因为它仅适用于不在缓冲池中的页面。

 您可以使用innodb_change_buffering配置参数控制InnoDB执行change buffer的程度。 您可以为插入，删除操作（当索引记录最初标记为删除时）和清除操作（物理删除索引记录时）启用或禁用缓冲。 更新操作是插入和删除的组合。 默认的innodb_change_buffering值是all。

允许的innodb_change_buffering值包括：

* all

默认值：缓冲区插入，删除标记操作和清除。

* none
不要缓冲任何操作。

* inserts

缓冲插入操作。

* deletes

缓冲区删除标记操作。

* changes

缓冲插入和删除标记操作。

* purges

缓冲在后台发生的物理删除操作。

您可以在MySQL参数配置文件（my.cnf或my.ini）中设置innodb_change_buffering参数，或使用SET GLOBAL语句动态更改它，这需要足以设置全局系统变量的权限。 请参见第5.1.8.1节“系统变量权限”。 更改设置会影响新操作的缓冲; 现有缓冲条目的合并不受影响。

## 配置change buffer 最大size
 innodb_change_buffer_max_size变量表示允许将更改缓冲区的最大大小配置为缓冲池总大小的百分比。 默认情况下，innodb_change_buffer_max_size设置为25。最大设置为50。

考虑在具有大量插入，更新和删除活动的MySQL服务器上增加innodb_change_buffer_max_size，因为这会导致change buffer merge操作无法跟上产生新的change buffer条目速度，导致change buffer达到其最大大小限制。

 对于用于报告的静态数据的mysql服务器, 或者如果change buffer消耗了与缓冲池共享的太多内存空间，导致页面比预期更早地超出缓冲池, 考虑减少MySQL服务器上的innodb_change_buffer_max_size，。

使用具有代表性的工作负载测试不同的设置以确定最佳配置。innodb_change_buffer_max_size设置是动态的，允许在不重新启动服务器的情况下修改设置。

## 监控 Change Buffer

以下选项可用于更改change buffer的监视：

InnoDB Standard Monitor 输出了包括更改缓冲区状态信息。 要查看监视器数据，请使用SHOW ENGINE INNODB STATUS语句。
```
mysql> SHOW ENGINE INNODB STATUS\G
```

Change Buffer状态信息位于INSERT BUFFER和ADAPTIVE HASH INDEX标题下，并显示类似于以下内容：
```
-------------------------------------
INSERT BUFFER AND ADAPTIVE HASH INDEX
-------------------------------------
Ibuf: size 1, free list len 0, seg size 2, 0 merges
merged operations:
 insert 0, delete mark 0, delete 0
discarded operations:
 insert 0, delete mark 0, delete 0
Hash table size 4425293, used cells 32, node heap has 1 buffer(s)
13577.57 hash searches/s, 202.47 non-hash searches/s
```

有关更多信息，请参见第14.17.3节“InnoDB标准监视器和锁定监视器输出”。


NFORMATION_SCHEMA.INNODB_METRICS表提供了InnoDB Standard Monitor输出中的大多数数据点以及其他数据点。要查看Change Buffer指标及其各自的说明，请使用以下查询：

```
mysql> SELECT NAME, COMMENT FROM INFORMATION_SCHEMA.INNODB_METRICS WHERE NAME LIKE '%ibuf%'\G
```

有关INNODB_METRICS表使用信息，请参见第14.15.6节“InnoDB INFORMATION_SCHEMA度量表”。

INFORMATION_SCHEMA.INNODB_BUFFER_PAGE表提供有关缓冲池中每个页面的元数据，包括更改缓冲区索引和更改缓冲区位图页面。更改缓冲区页面由PAGE_TYPE标识。 IBUF_INDEX是更改缓冲区索引页面的页面类型，IBUF_BITMAP是更改缓冲区位图页面的页面类型。

> 警告 <br>
查询INNODB_BUFFER_PAGE表可能会带来显着的性能开销。为避免影响性能，请在测试实例上重现要调查的问题，并在测试实例上运行查询。

例如，您可以查询INNODB_BUFFER_PAGE表以确定IBUF_INDEX和IBUF_BITMAP页面的大致数量占总缓冲池页面的百分比。

```
mysql> SELECT (SELECT COUNT(*) FROM INFORMATION_SCHEMA.INNODB_BUFFER_PAGE
       WHERE PAGE_TYPE LIKE 'IBUF%') AS change_buffer_pages,
       (SELECT COUNT(*) FROM INFORMATION_SCHEMA.INNODB_BUFFER_PAGE) AS total_pages,
       (SELECT ((change_buffer_pages/total_pages)*100))
       AS change_buffer_page_percentage;
```

|change_buffer_pages |total_pages|change_buffer_page_percentage|
|-------------------:|:---------:|:---------------------------:|
|25                  |8192       |0.3052                       |

有关INNODB_BUFFER_PAGE表提供的其他数据的信息，请参见第24.32.1节“INFORMATION_SCHEMA INNODB_BUFFER_PAGE表”。有关相关用法信息，请参见第14.15.5节“InnoDB INFORMATION_SCHEMA缓冲池表”。

Performance Schema为高级性能监视提供Change Buffer互斥等待检测。要查看更改缓冲区检测，请使用以下查询：
```
mysql> SELECT * FROM performance_schema.setup_instruments
       WHERE NAME LIKE '%wait/synch/mutex/innodb/ibuf%';
```

|NAME|ENABLED|TIMED|
|:-------------------:|:---------:|:---------------------------:|
|wait/synch/mutex/innodb/ibuf_bitmap_mutex|YES       |YES      |
|:wait/synch/mutex/innodb/ibuf_mutex|YES|YES|
|:wait/synch/mutex/innodb/ibuf_pessimistic_insert_mutex |YES|YES|

有关监视InnoDB互斥等待的信息，请参见第14.16.2节“使用性能模式监视InnoDB Mutex等待”。