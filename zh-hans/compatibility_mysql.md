# 对比原版 MySQL 的兼容性

## 总结

MySQL on TerarkDB 使用了 MyRocks 适配层，除了部分功能不同于 MyRocks 之外，我们和 MySQL 的兼容性和 MyRocks 对 MySQL 的兼容性一致。

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
