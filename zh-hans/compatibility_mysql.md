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
| main               | 871 | 861 | **10**|
| sys_vars           | 727 | 727 |   0   |
| binlog             | 165 | 165 |   0   |
| federated          |   7 |   7 |   0   |
| rpl                | 921 | 918 | **3** |
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
| perfschema_stress  |   4 |   2 | **2** |

### 1. main
```
Failing test(s):
    main.admission_control_multi_query
    main.ssl_8k_key
    main.ssl_crl
    main.openssl_1
    main.plugin_auth_sha256_tls
    main.ssl-session-reuse
    main.ssl
    main.ssl_compress
    main.mysql_shutdown_logging
    main.mysqld--help-notwin-profiling
    main.innodb_mysql_lock
    main.mysqlshow
    main.commit_1innodb
    main.admission_control_stress
```

#### 1.1 SSL 加密算法与预期不同

涉及测试：
```
main.ssl_8k_key
main.ssl_crl
main.plugin_auth_sha256_tls
main.ssl-session-reuse
main.ssl
main.ssl_compress
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
main.openssl_1
main.mysql_shutdown_logging
```

错误信息如下：
```
mysqltest: At line 24: query 'connect  con2,localhost,ssl_user2,,,,,SSL' failed: 1045: Access denied for user 'ssl_user2'@'localhost' (using password: NO)
```

#### 1.3 统计信息不一致

涉及测试：
```
main.mysqld--help-notwin-profiling
main.mysqlshow
```

我们在 MyRocks 的 information_schema 数据库中添加了一个 ROCKSDB_TF_OPTIONS，以及添加了一个用于 flush `__system__` column family 的背景线程，故统计系统与预期不一致。

main.mysqld--help-notwin-profiling：
```
--- /newssd1/temp/mysql-on-terarkdb-4.8-bmi2-0/mysql-test/r/mysqld--help-notwin-profiling.result	2017-11-01 06:42:45.000000000 +0300
+++ /newssd1/temp/mysql-on-terarkdb-4.8-bmi2-0/mysql-test/var/log/mysqld--help-notwin-profiling.reject	2017-12-07 15:19:19.211744312 +0300
@@ -1298,6 +1298,9 @@
  --rocksdb-strict-collation-exceptions=name 
  List of tables (using regex) that are excluded from the
  case sensitive collation enforcement
+ --rocksdb-system-cf-background-flush-interval=# 
+ interval(seconds) of background flush column family
+ '__system__' for RocksDB . 0 to disable
  --rocksdb-table-cache-numshardbits=# 
  DBOptions::table_cache_numshardbits for RocksDB
  --rocksdb-table-stats-sampling-pct=# 
@@ -1305,6 +1308,10 @@
  statistics about table properties. Specify either 0 to
  sample everything or percentage [1..100]. By default 10%
  of entries are sampled.
+ --rocksdb-tf-options[=name] 
+ Enable or disable ROCKSDB_TF_OPTIONS plugin. Possible
+ values are ON, OFF, FORCE (don't start if the plugin
+ fails to load).
  --rocksdb-tmpdir[=name] 
  Directory for temporary files during DDL operations.
  --rocksdb-trace-sst-api 
@@ -2048,8 +2055,10 @@
 rocksdb-store-row-debug-checksums FALSE
 rocksdb-strict-collation-check TRUE
 rocksdb-strict-collation-exceptions (No default value)
+rocksdb-system-cf-background-flush-interval 120
 rocksdb-table-cache-numshardbits 6
 rocksdb-table-stats-sampling-pct 10
+rocksdb-tf-options ON
 rocksdb-tmpdir (No default value)
 rocksdb-trace-sst-api FALSE
 rocksdb-trx ON
```

main.mysqlshow：
```
--- /newssd1/temp/mysql-on-terarkdb-4.8-bmi2-0/mysql-test/r/mysqlshow.result	2017-11-01 06:42:45.000000000 +0300
+++ /newssd1/temp/mysql-on-terarkdb-4.8-bmi2-0/mysql-test/var/log/mysqlshow.reject	2017-12-07 15:14:16.721734704 +0300
@@ -138,6 +138,7 @@
 | ROCKSDB_LOCKS                         |
 | ROCKSDB_PERF_CONTEXT                  |
 | ROCKSDB_PERF_CONTEXT_GLOBAL           |
+| ROCKSDB_TF_OPTIONS                    |
 | ROCKSDB_TRX                           |
 | ROUTINES                              |
 | SCHEMATA                              |
@@ -220,6 +221,7 @@
 | ROCKSDB_LOCKS                         |
 | ROCKSDB_PERF_CONTEXT                  |
 | ROCKSDB_PERF_CONTEXT_GLOBAL           |
+| ROCKSDB_TF_OPTIONS                    |
 | ROCKSDB_TRX                           |
 | ROUTINES                              |
 | SCHEMATA                              |
```

### 2. rpl
```
Failing test(s):
    rpl.rpl_ssl
```
2.1 SSL 加密算法与预期不同

涉及测试：
```
rpl.rpl_ssl
```

错误信息如下：
```
--- /newssd1/temp/mysql-on-terarkdb-4.8-bmi2-0/mysql-test/suite/rpl/r/rpl_ssl.result	2017-06-08 06:26:45.000000000 +0300
+++ /newssd1/temp/mysql-on-terarkdb-4.8-bmi2-0/mysql-test/var/3/log/rpl_ssl.reject	2017-12-07 15:42:42.519286165 +0300
@@ -28,7 +28,7 @@
 Master_SSL_CA_File = 'MYSQL_TEST_DIR/std_data/cacert.pem'
 Master_SSL_Cert = 'MYSQL_TEST_DIR/std_data/client-cert.pem'
 Master_SSL_Key = 'MYSQL_TEST_DIR/std_data/client-key.pem'
-Master_SSL_Actual_Cipher = 'ECDHE-RSA-AES128-GCM-SHA256'
+Master_SSL_Actual_Cipher = 'ECDHE-RSA-AES256-GCM-SHA384'
 Master_SSL_Subject = '/C=SE/ST=Uppsala/O=MySQL AB/CN=localhost'
 Master_SSL_Issuer = '/C=SE/ST=Uppsala/L=Uppsala/O=MySQL AB'
 include/check_slave_is_running.inc
@@ -44,7 +44,7 @@
 Master_SSL_CA_File = 'MYSQL_TEST_DIR/std_data/cacert.pem'
 Master_SSL_Cert = 'MYSQL_TEST_DIR/std_data/client-cert.pem'
 Master_SSL_Key = 'MYSQL_TEST_DIR/std_data/client-key.pem'
-Master_SSL_Actual_Cipher = 'ECDHE-RSA-AES128-GCM-SHA256'
+Master_SSL_Actual_Cipher = 'ECDHE-RSA-AES256-GCM-SHA384'
 Master_SSL_Subject = '/C=SE/ST=Uppsala/O=MySQL AB/CN=localhost'
 Master_SSL_Issuer = '/C=SE/ST=Uppsala/L=Uppsala/O=MySQL AB'
 include/check_slave_is_running.inc
```


### 3. xtrabackup
```
Failing test(s):
    xtrabackup.xb_with_xbstream
    xtrabackup.xb_drop_tables
    xtrabackup.xb_basic
    xtrabackup.xb_rechecksum
    xtrabackup.xb_rechecksum_compressed
    xtrabackup.xb_with_logs
    xtrabackup.xb_gtid
    xtrabackup.xb_partitioned_table
    xtrabackup.xb_compressed_table
```

找不到可执行文件 `MYSQL_INNOBACKUPEX`

### 4. engines/rr_trx
```
Failing test(s):
    engines/rr_trx.rr_iud_rollback-multi-50
    engines/rr_trx.rr_c_stats
    engines/rr_trx.rr_replace_7-8
    engines/rr_trx.rr_id_900
    engines/rr_trx.rr_id_3
    engines/rr_trx.rr_insert_select_2
    engines/rr_trx.rr_c_count_not_zero
    engines/rr_trx.rr_i_40-44
    engines/rr_trx.rr_s_select-uncommitted
    engines/rr_trx.rr_sc_sum_total
    engines/rr_trx.rr_u_10-19_nolimit
    engines/rr_trx.rr_sc_select-limit-nolimit_4
    engines/rr_trx.rr_sc_select-same_2
    engines/rr_trx.rr_u_10-19
    engines/rr_trx.rr_u_4
    engines/rr_trx.init_innodb
```
