14.1.5 不适用InnoDB

Oracle建议将InnoDB作为典型数据库应用程序的首选存储引擎，从单用户wiki和本地系统上运行的博客，到推动性能极限的高端应用程序。在MySQL 5.6中，InnoDB是新表的默认存储引擎。

如果您不想使用InnoDB表：
* 使用--innodb = OFF或--skip-innodb选项启动服务器以禁用InnoDB存储引擎。
> 注意<br>
从MySQL 5.6.21开始，--skip-innodb选项仍然有效但不推荐使用，并在使用时返回警告。它将在未来的MySQL版本中删除。这也适用于其同义词（ -  innodb = OFF， -  disable-innodb等）。
* 由于默认存储引擎是InnoDB，因此，除非您还通过使用--default-storage-engine和--default-tmp-storage-engine将permanent表和TEMPORARY表的默认存储引擎设置为其它的存储引擎。
* 要在查询与InnoDB相关的information_schema表时防止服务器宕掉，还要禁用与这些表关联的插件。在MySQL配置文件的[mysqld]部分中指定：

```
loose-innodb-trx=0
loose-innodb-locks=0
loose-innodb-lock-waits=0
loose-innodb-cmp=0
loose-innodb-cmp-per-index=0
loose-innodb-cmp-per-index-reset=0
loose-innodb-cmp-reset=0
loose-innodb-cmpmem=0
loose-innodb-cmpmem-reset=0
loose-innodb-buffer-page=0
loose-innodb-buffer-page-lru=0
loose-innodb-buffer-pool-stats=0
loose-innodb-metrics=0
loose-innodb-ft-default-stopword=0
loose-innodb-ft-inserted=0
loose-innodb-ft-deleted=0
loose-innodb-ft-being-deleted=0
loose-innodb-ft-config=0
loose-innodb-ft-index-cache=0
loose-innodb-ft-index-table=0
loose-innodb-sys-tables=0
loose-innodb-sys-tablestats=0
loose-innodb-sys-indexes=0
loose-innodb-sys-columns=0
loose-innodb-sys-fields=0
loose-innodb-sys-foreign=0
loose-innodb-sys-foreign-cols=0
```