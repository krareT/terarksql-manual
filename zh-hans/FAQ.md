# 常见问题
## MySQL on TerarkDB 在什么场景下最适用?
MySQL on TerarkDB 在大多数场景下都很出色，尤其在随机读非常多，并且文本数据较多的场景下表现最为出众。

## MySQL on TearkDB 和原生的 MySQL 兼容性如何?
MySQL on TerarkDB 使用 [MyRocks](https://github.com/facebook/mysql-5.6/wiki/) 作为 TerarkDB 和 MySQL 的适配层.

MySQL on TerarkDB 支持所有 MyRocks 有的特性, 当然，MyRocks 对于部分 MySQL 特性还未支持（因此 MySQL on TerarkDB 也暂时不支持）

具体的功能限制可以参考：[MyRocks Limits](https://github.com/facebook/mysql-5.6/wiki/MyRocks-limitations)

## 如何主动执行 compact?
```
set global rocksdb_compact_cf = 'default';
```
其中 '`default`' 是 ColumnFamily 的名字，也可以 compact 其他 ColumnFamily，例如：
```
set global rocksdb_compact_cf = '__system__';
```
'`default`' 是普通数据所在的 ColumnFamily<br/>
'`__system__`' 是数据字典、元数据所在的 ComlumnFamily

## 如何通过 `load data infile` 快速加载巨大的数据文件?

由于 MySQL on TerarkDB 基于 MyRocks 开发, 而 MyRocks 主要针对较小的事务进行了优化，所以对于较大的数据集加载需要一些特殊操作：

MyRocks 通过一些环境变量来加速大数据集的加载，详情可以参考 [MyRocks Data Loading Details](https://github.com/facebook/mysql-5.6/wiki/data-loading).

在 `my.cnf` 中添加如下设置：
```
rocksdb_commit_in_the_middle=ON
rocksdb_bulk_load_size=2000
```

亦可使用下面的方式进行参数设置:
```
SET session sql_log_bin=0;              -- 临时禁用日志
SET rocksdb_commit_in_the_middle=ON;     -- 定期提交事务(默认是 1000 次插入，亦可通过 rocksdb_bulk_load_size 设置)
```

MyRocks 对这两个选项的说明如下：
- rocksdb-commit-in-the-middle : Commit rows implicitly every rocksdb-bulk-load-size, during bulk load/insert/update/deletes.
- rocksdb-bulk-load-size : Sets the number of keys to accumulate before committing them to the storage engine during bulk loading.

## 在写入性能较差的磁盘（如 HDD）中如何配置 WAL 以提高性能?

在 `my.cnf` 中添加如下设置：
```
rocksdb_flush_log_at_trx_commit=2
rocksdb_background_sync=ON
```
这样 MySQL-on-TerarkDB 会在后台**每秒同步一次 WAL 文件**，此时若发生 crash 将可能会丢失一秒的数据。

在 `my.cnf` 中添加如下设置时
```
rocksdb_flush_log_at_trx_commit=0
```
将会关闭 fsync，此时若发生 crash，将会导致**大量数据丢失**。

MyRocks 对这两个选项的说明如下：

- rocksdb-background-sync : Enables MyRocks to issue fsyncs for the WAL files every second. If WAL files are sync'ed on every commits, then enabling this option is redundant.
- rocksdb-flush-log-at-trx-commit : Sync'ing on transaction commit similar to innodb-flush-log-at-trx-commit.
  - 0 : never sync
  - 1 : always sync
  - 2 : sync based on a timer controlled via rocksdb-background-sync
