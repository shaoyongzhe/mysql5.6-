# 14.1.3 检查InnoDB可用性
要确定您的服务器是否支持InnoDB：
* 使用SHOW ENGINES语句查看可用的MySQL存储引擎。在InnoDB那一行中查找DEFAULT。
```
mysql> SHOW ENGINES;
```

或者，查询INFORMATION_SCHEMA.ENGINES表。
```
mysql> SELECT * FROM INFORMATION_SCHEMA.ENGINES;
```

（现在InnoDB是默认的MySQL存储引擎，只有非常特殊的环境可能不支持它。）

* 使用'SHOW VARIABLES'语句确认InnoDB是否可用。
```
mysql> SHOW VARIABLES LIKE 'have_innodb';
```

* 如果没有InnoDB，说明你的mysqld使用的是没有InnoDB支持编译的二进制文件，你需要一个使用InnoDB支持的二进制文件。

* 如果InnoDB存在但已禁用，请返回启动选项和配置文件，并删除任何--skip-innodb选项。