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

## MyRocks 对 mysql-test 的测试情况

MyRocks 的测试有以下 36 种：
```
main,auth_sec,connection_control,federated,funcs_2,json,multiengine,opt_trace,perfschema,stress,
xtrabackup,binlog,engines,funcs_1,large_tests,parts,perfschema_stress,rpl,rpl_recovery,sys_vars,
innodb_fts,innodb_zip,ndb_big,ndb_rpl,rpl_ndb,innodb,innodb_stress,jp,ndb,ndb_binlog,ndb_team,
rocksdb,rocksdb_rpl,rocksdb_sys_vars,rocksdb_hotbackup,rocksdb_stress
```

rocksdb 引擎相关的测试情况在[这里](compatibility_myrocks.md)，innodb 和 ndb 引擎相关的测试略去，剩余的测试情况如下：

| suite              |total|success| fail |
|:------------------:|:---:|:-----:|:----:|
| main               | 871 | 856 | **15**|
| sys_vars           | 725 | 724 | **1** |
| binlog             | 165 | 164 | **1** |
| federated          |   7 |   7 |   0   |
| rpl                | 921 | 915 | **6** |
| rpl_recovery       |  51 |  51 |   0   |
| perfschema         | 314 | 314 |   0   |
| funcs_1            | 103 | 103 |   0   |
| opt_trace          |  13 | 13  |   0   |
| parts              | 115 | 115 |   0   |
| auth_sec           |   7 |   7 |   0   |
| connection_control |   8 |   8 |   0   |
| json               |  25 |  25 |   0   |
| funcs_2            |   3 |   3 |   0   |
| multiengine        |   2 |   2 |   0   |
| stress             |   5 |   5 |   0   |
| xtrabackup         |   9 |   0 | **9** |
| engines/funcs      | 310 | 310 |   0   |
| engines/iuds       |  13 |  13 |   0   |
| engines/rr_trx     |  16 |   0 | **16**|
| large_tests        |   1 |   1 |   0   |
| perfschema_stress  | 314 | 314 |   0   |

### 1. main
```
Failing test(s): main.admission_control_multi_query main.ssl_8k_key main.ssl_crl main.openssl_1 main.plugin_auth_sha256_tls
main.ssl-session-reuse main.ssl main.ssl_compress main.mysql_shutdown_logging main.mysqld--help-notwin-profiling
main.innodb_mysql_lock main.mysqlshow main.commit_1innodb main.enable_multiple_engines main.admission_control_stress
```

#### 1.1 SSL 加密算法不同

涉及测试：
```
main.ssl_8k_key main.ssl_crl main.plugin_auth_sha256_tls main.ssl-session-reuse main.ssl main.ssl_compress
```

错误信息如下：
```
SHOW STATUS LIKE 'ssl_Cipher'；
Variable_name	Value
-Ssl_cipher	ECDHE-RSA-AES128-GCM-SHA256
+Ssl_cipher	ECDHE-RSA-AES256-GCM-SHA384
```

#### 1.2 access denied

涉及测试：
```
main.openssl_1 main.mysql_shutdown_logging
```

错误信息如下：
```
mysqltest: At line 24: query 'connect  con2,localhost,ssl_user2,,,,,SSL' failed: 1045: Access denied for user 'ssl_user2'@'localhost' (using password: NO)
```

#### 1.3 统计信息不一致

### sys_vars
```
Failing test(s): sys_vars.all_vars
```

### binlog
```
Failing test(s): binlog.binlog_row_binlog
```

### rpl
```
Failing test(s): rpl.rpl_ssl rpl.rpl_report rpl.rpl_stm_innodb
```

### xtrabackup
```
Failing test(s): xtrabackup.xb_with_xbstream xtrabackup.xb_drop_tables xtrabackup.xb_basic 
xtrabackup.xb_rechecksum xtrabackup.xb_rechecksum_compressed xtrabackup.xb_with_logs 
xtrabackup.xb_gtid xtrabackup.xb_partitioned_table xtrabackup.xb_compressed_table
```

### engines/rr_trx
```
Failing test(s): engines/rr_trx.rr_iud_rollback-multi-50 engines/rr_trx.rr_c_stats 
engines/rr_trx.rr_replace_7-8 engines/rr_trx.rr_id_900 engines/rr_trx.rr_id_3 
engines/rr_trx.rr_insert_select_2 engines/rr_trx.rr_c_count_not_zero engines/rr_trx.rr_i_40-44 
engines/rr_trx.rr_s_select-uncommitted engines/rr_trx.rr_sc_sum_total engines/rr_trx.rr_u_10-19_nolimit 
engines/rr_trx.rr_sc_select-limit-nolimit_4 engines/rr_trx.rr_sc_select-same_2 engines/rr_trx.rr_u_10-19 
engines/rr_trx.rr_u_4 engines/rr_trx.init_innodb
```
