# 对比原版 MySQL 的兼容性

## 总结

TerarkSQL 使用了 MyRocks 适配层，除了部分功能不同于 MyRocks 之外，我们和 MySQL 的兼容性和 MyRocks 对 MySQL 的兼容性一致。

由于兼容性测试非常复杂，我们会逐步补充相关的兼容性总结并更新：

- 安全性
- 备份
- 无效的测试用例
- 配置
- 主从同步
- 其他

## MyRocks 对 MySQL 的兼容性
目前，MyRocks 官方提供了以下说明可供参考，我们目前还正在进行充分的兼容性测试：


MyRocks currently lacks a large number of features compared to InnoDB:

* [Online DDL](https://github.com/facebook/mysql-5.6/issues/47) is not supported yet, but fast alter table add and drop indexes are supported.
* [EXCHANGE PARTITION does not work in MyRocks yet](https://github.com/facebook/mysql-5.6/issues/264)
* Lack of [SAVEPOINT support](https://github.com/facebook/mysql-5.6/issues/52)
* Transportable Tablespace, Foreign Key, Spatial Index, and Fulltext Index are not supported
* Gap Lock support - see [MyRocks Row Locking](https://github.com/facebook/mysql-5.6/wiki/Row-Locking) - ROW based binary logging must be used. Statement based binary logging may cause data inconsistency between master and slave because MyRocks does not support Next-Key Locking.
* "*_bin" (e.g. latin1_bin) or binary collation should be used on CHAR/VARCHAR indexed columns. By default, MyRocks prevents creating indexes with non-binary collations (including latin1). You can optionally use it by setting rocksdb_strict_collation_exceptions='t1' (table names with regex format), but non_binary covering indexes other than latin1 (excluding german1) still require a primary key lookup to return the CHAR/VARCHAR column.
* Either ORDER BY DESC or ASC is slow. This is because of "Prefix Key Encoding" feature in RocksDB. See http://www.slideshare.net/matsunobu/myrocks-deep-dive/58 for details. By default, ascending scan is faster but descending scan is slower. By configuring "reverse column family”, descending scan is faster but ascending scan is slower. Note that InnoDB also [imposes a cost](http://mysqlserverteam.com/mysql-8-0-labs-descending-indexes-in-mysql/) when the index is scanned in the opposite order.

## MyRocks 对 mysql-test 的测试情况

MySQL 随源码发布了一个测试框架 [mysql-test-run](https://dev.mysql.com/doc/dev/mysql-server/latest/PAGE_MYSQL_TEST_RUN.html)，并附带了大量的测试。MyRocks 继承了这个框架和所有的测试，并针对自身特点做了修改以及添加了对 RocksDB 引擎的测试。MyRocks 的测试有以下 36 种 suite：
```
main,auth_sec,connection_control,federated,funcs_2,json,multiengine,opt_trace,perfschema,stress,
xtrabackup,binlog,engines,funcs_1,large_tests,parts,perfschema_stress,rpl,rpl_recovery,sys_vars,
innodb_fts,innodb_zip,ndb_big,ndb_rpl,rpl_ndb,innodb,innodb_stress,jp,ndb,ndb_binlog,ndb_team,
rocksdb,rocksdb_rpl,rocksdb_sys_vars,rocksdb_hotbackup,rocksdb_stress
```

rocksdb 引擎相关的测试情况在[这里](compatibility_myrocks.md)，innodb 和 ndb 引擎相关的测试与此次兼容性无关，故略去，剩余的测试情况如下：

| suite              |total|success| fail | skipped |
|:------------------:|:---:|:-----:|:----:|:-------:|
| main               | 883 | 872 | **11**|  40 |
| sys_vars           | 727 | 727 |   0   |  29 |
| binlog             | 183 | 180 | **3** |   4 |
| federated          |   7 |   7 |   0   |   2 |
| rpl                | 972 | 969 | **3** |   9 |
| rpl_recovery       |  51 |  51 |   0   |   5 |
| perfschema         | 315 | 315 |   0   |   2 |
| funcs_1            | 104 | 104 |   0   |  15 |
| opt_trace          |  13 | 13  |   0   |   8 |
| parts              | 124 | 124 |   0   |   6 |
| auth_sec           |  15 |  14 | **1** |   3 |
| connection_control |   8 |   8 |   0   |   0 |
| json               |  25 |  25 |   0   |   0 |
| funcs_2            |   3 |   3 |   0   |   0 |
| multiengine        |   2 |   2 |   0   |   0 |
| stress             |   5 |   5 |   0   |   0 |
| xtrabackup         |   9 |   0 | **9** |   0 |
| engines/funcs      | 311 | 311 |   0   |   0 |
| engines/iuds       |  13 |  13 |   0   |   0 |
| engines/rr_trx     |  16 |   0 | **16**|   0 |
| large_tests        |   3 |   3 |   0   |   0 |
| perfschema_stress  |   4 |   2 | **2** |   0 |

其中 fail 的测试为当前版本 MyRocks 未能通过的测试，skipped 的测试为需要特殊的条件或者设置才能运行。

关于 case 失败或者被跳过的原因，请参看子页面。




## 更新

1. udf_example 插件正确编译并加入到插件库，所有被跳过的 udf 插件相关测试不再被跳过，并且都正确通过测试。 2017.12.20
2. main.openssl_1 测试能正确进行。SSL 加密算法不一致，将测试中的加密算法修改为与平台一致，就能通过测试。 2017.12.22
