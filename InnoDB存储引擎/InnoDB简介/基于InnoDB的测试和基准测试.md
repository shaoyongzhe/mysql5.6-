# 14.1.4 基于InnoDB的测试和基准测试

如果InnoDB不是您的默认存储引擎，您可以通过在命令行上使用`--default-storage-engine = InnoDB`重新启动服务器或使用MySQL服务器选项文件的[mysqld]部分中定义`default-storage-engine = innodb` 来确定数据库服务器或应用程序是否可以正常使用InnoDB。

由于更改默认存储引擎仅会在创建新表时影响新表，因此请运行所有应用程序安装和设置步骤，以确认所有内容都已正确安装。然后检查所有应用程序功能，以确保所有数据加载，编辑和查询功能都能正常工作。如果表依赖于特定于另一个存储引擎的功能，您将收到错误;将ENGINE = other_engine_name子句添加到CREATE TABLE语句以避免错误。
如果您没有对存储引擎作出深思熟虑的决定，并且想要预览某些表使用InnoDB创建后的工作方式，
可以通过在每张表上使用命令`ALERT TABLE table_name ENGINE = InnoDB`。或者，在不干扰原始表的情况下，运行测试查询和其它语句，请复制：
```
CREATE TABLE InnoDB_Table (...) ENGINE=InnoDB AS SELECT * FROM other_engine_table;
```

要在实际工作负载下评估完整应用程序的性能，请安装最新的MySQL服务器并运行基准测试。

测试整个应用程序生命周期，从安装，大量使用和服务器重启。通过在数据库比较繁忙时，终止服务器进程来模拟电源故障，并通过重新启动服务器来验证数据是否已经成功恢复。
测试任何复制配置，尤其是在主服务器和从服务器上使用不同的MySQL版本和选项时。
