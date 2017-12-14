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
| main               | 878 | 867 | **11**|  45 |
| sys_vars           | 727 | 727 |   0   |  29 |
| binlog             | 182 | 179 | **3** |   5 |
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

其中 fail 的测试为当前版本 MyRocks 未能通过的测试，skipped 的测试为需要特殊的条件或者设置才能运行。

下面详细说明每个 suite 中的失败或者被跳过的原因

### 1. main

#### 1.1 失败的测试
```
Failing test(s):
    main.ssl_8k_key
    main.ssl_crl
    main.openssl_1
    main.plugin_auth_sha256_tls
    main.ssl-session-reuse
    main.ssl
    main.ssl_compress
    main.mysql_shutdown_logging
    main.mysqld--help-notwin-profiling
    main.mysqlshow
    main.ssl_ca
```
共 11 个，按失败原因分类可分为一下几类

#### 1.1.1 SSL 加密算法与预期不同

测试预期使用的 SSL 加密算法与当前进行测试时使用的加密算法不一致导致测试结果与预期不一致。

涉及测试：
```
main.ssl_8k_key
main.ssl_crl
main.plugin_auth_sha256_tls
main.ssl-session-reuse
main.ssl
main.ssl_compress
main.ssl_ca
```
共 7 个。

错误信息类似如下：
```
SHOW STATUS LIKE 'ssl_Cipher'；
Variable_name	Value
-Ssl_cipher	ECDHE-RSA-AES128-GCM-SHA256
+Ssl_cipher	ECDHE-RSA-AES256-GCM-SHA384
```

#### 1.1.2 access denied

在测试过程中，测试程序不能连接到服务器，具体原因待查证。

涉及测试：
```
main.openssl_1
main.mysql_shutdown_logging
```
共 2 个。

错误信息如下：
```
mysqltest: At line 24: query 'connect  con2,localhost,ssl_user2,,,,,SSL' failed: 1045: Access denied for user 'ssl_user2'@'localhost' (using password: NO)
```

#### 1.1.3 统计信息不一致

我们在 MyRocks 的 information_schema 数据库中添加了一个 ROCKSDB_TF_OPTIONS，以及添加了一个用于 flush `__system__` column family 的背景线程，故统计系统与预期不一致。

涉及测试：
```
main.mysqld--help-notwin-profiling
main.mysqlshow
```
共 2 个。

错误信息如下：

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

#### 1.2 跳过的测试

##### 1.2.1 need windows

测试需要运行在 Windows 平台上

涉及测试：
```
main.named_pipe
main.shm
main.secure_file_priv_win
main.partition_windows
main.perror-win
main.variables-win
main.windows
main.mysqld--help-win
main.symlink_windows
main.mysqlbinlog_raw_mode_win
```
共 10 个

##### 1.2.2 is_embedded

需要嵌入式版 MySQL 程序

涉及测试：
```
main.mysql_embedded
main.server_uuid_embedded
main.events_embedded
```
共 3 个

##### 1.2.3 其他

需要特殊的设置或者特殊编译程序，但是不能合适的设置或者没有找到合适设置方法

```
main.bug46261                            w1 [ skipped ]  Example plugin requires the environment variable \$EXAMPLE_PLUGIN to be set (normally done by mtr)
main.plugin_load                         w3 [ skipped ]  Example plugin requires the environment variable \$EXAMPLE_PLUGIN to be set (normally done by mtr)
main.plugin_load_option                  w4 [ skipped ]  Example plugin requires the environment variable \$EXAMPLE_PLUGIN to be set (normally done by mtr)
main.plugin                              w1 [ skipped ]  Example plugin requires the environment variable \$EXAMPLE_PLUGIN to be set (normally done by mtr)
main.plugin_not_embedded                 w1 [ skipped ]  Example plugin requires the environment variable \$EXAMPLE_PLUGIN to be set (normally done by mtr)
main.multi_plugin_load                   w2 [ skipped ]  Need the plugin test_plugin_server
main.udf                                 w2 [ skipped ]  UDF requires the environment variable \$UDF_EXAMPLE_LIB to be set (normally done by mtr)
main.udf_skip_grants                     w1 [ skipped ]  UDF requires the environment variable \$UDF_EXAMPLE_LIB to be set (normally done by mtr)
main.partition_open_files_limit          w4 [ skipped ]  Need open_files_limit to be lower than 1025
main.archive_plugin                      w1 [ skipped ]  archive plugin not available
main.blackhole_plugin                    w1 [ skipped ]  blackhole plugin not available;
main.ssl-sha512                          w4 [ skipped ]  Test requires: 'not_openssl'
main.query_cache_ps_ps_prot              w1 [ skipped ]  Test requires: ps-protocol enabled, other protocols disabled
main.grant_lowercase_fs                  w1 [ skipped ]  Test requires: 'case_insensitive_file_system'
main.lowercase_fs_on                     w1 [ skipped ]  Test requires: 'case_insensitive_file_system'
main.innodb_recovery_with_upper_case_names w3 [ skipped ]  Test requires: 'case_insensitive_file_system'
main.lowercase_table4                    w2 [ skipped ]  Test requires: 'case_insensitive_file_system'
main.jemalloc                            w1 [ skipped ]  Test requires jemalloc
main.mysqld--help-notwin                 w1 [ skipped ]  Test requires: 'have_noprofiling'
main.ctype_cp932_binlog_row              w2 [ skipped ]  Test requires: 'have_binlog_format_row'
main.not_partition                       w1 [ skipped ]  Test requires: 'true'
main.system_mysql_db_fix40123            w3 [ skipped ]  Test need MYSQL_FIX_PRIVILEGE_TABLES
main.system_mysql_db_fix50030            w3 [ skipped ]  Test needs MYSQL_FIX_PRIVILEGE_TABLES
main.system_mysql_db_fix50117            w3 [ skipped ]  Test needs MYSQL_FIX_PRIVILEGE_TABLES
main.fix_priv_tables                     w3 [ skipped ]  Test need MYSQL_FIX_PRIVILEGE_TABLES
main.timezone3                           w3 [ skipped ]  Test requires: 'have_moscow_leap_timezone'
main.lowercase_mixed_tmpdir_innodb       w2 [ skipped ]  Test requires: 'lowercase2'
main.lowercase_table2                    w3 [ skipped ]  Test requires: 'lowercase2'
main.slow_log_legacy_user                w1 [ skipped ]  Test requires full regex (not supported in gcc 4.8 and prior)
main.sp_trans_log                        w3 [ skipped ]  Test requires: 'have_binlog_format_row'
main.dynamic_tracing                     w1 [ skipped ]  dtrace/stap tool requires additional privileges to run this test.
main.func_encrypt_nossl                  w4 [ skipped ]  Test requires: 'not_openssl'
```

共 32 个。

### 2. sys_vars

#### 2.1 跳过的测试

##### 2.1.1 Need windows

需要运行在 Windows 平台上。

涉及测试如下：
```
sys_vars.shared_memory_base_name_basic
sys_vars.shared_memory_basic
sys_vars.named_pipe_basic
```
共 3 个。

##### 2.1.2 Need a 32 bit machine/binary

需要 32 位版本程序和平台。

涉及测试如下：
```
sys_vars.log_warnings_basic_32
sys_vars.binlog_cache_size_basic_32
sys_vars.binlog_stmt_cache_size_basic_32
sys_vars.max_connect_errors_basic_32
sys_vars.bulk_insert_buffer_size_basic_32
sys_vars.max_seeks_for_key_basic_32
sys_vars.max_tmp_tables_basic_32
sys_vars.max_write_lock_count_basic_32
sys_vars.min_examined_row_limit_basic_32
sys_vars.slave_transaction_retries_basic_32
sys_vars.multi_range_count_basic_32
sys_vars.myisam_max_sort_file_size_basic_32
sys_vars.delayed_insert_limit_basic_32
sys_vars.myisam_repair_threads_basic_32
sys_vars.delayed_queue_size_basic_32
sys_vars.sort_buffer_size_basic_32
sys_vars.myisam_sort_buffer_size_basic_32
sys_vars.net_retry_count_basic_32
sys_vars.query_alloc_block_size_basic_32
sys_vars.query_cache_limit_basic_32
sys_vars.query_cache_min_res_unit_basic_32
sys_vars.range_alloc_block_size_basic_32
sys_vars.join_buffer_size_basic_32
sys_vars.key_cache_age_threshold_basic_32
```
共 24 个。

##### 2.1.3 其他

需要系统关闭 NUMA 和需要完整的正则支持

涉及测试：
```
sys_vars.innodb_numa_interleave_basic
sys_vars.legacy_user_name_pattern_basic
```
共 2 个。

### 3. binlog

#### 3.1 失败的测试
```
Failing test(s): 
    binlog.binlog_gtid_mysqlbinlog_row_innodb
    binlog.binlog_gtid_mysqlbinlog_row
    binlog.binlog_gtid_mysqlbinlog_row_myisam
```

按其失败的原因可以分为以下几类

##### 3.1.1 测试中测试程序失去连接

在测试中，测试程序失去连接，不能连接到数据库。失去连接的原因待确定。

涉及测试：
```
binlog.binlog_gtid_mysqlbinlog_row_innodb
binlog.binlog_gtid_mysqlbinlog_row_myisam
```
共 2 个。

##### 3.1.2 开启 GTID MODE 后 binlog 格式不一致

开启 GTID MODE 后 binlog 格式不一致，具体原因待确认。

涉及测试：
```
binlog.binlog_gtid_mysqlbinlog_row
```
共 1 个。

#### 3.2 被跳过的测试

##### 3.2.1 ndbcluster disabled

ndbcluster 未启用，导致测试被跳过。

涉及测试：
```
binlog.binlog_multi_engine
```
共 1 个。

##### 3.2.2 其他

需要特殊的设置或者特殊编译程序，但是不能合适的设置或者没有找到合适设置方法

```
binlog.binlog_spurious_ddl_errors 'mix'  w2 [ skipped ]  Example plugin requires the environment variable \$EXAMPLE_PLUGIN to be set (normally done by mtr)
binlog.binlog_spurious_ddl_errors 'stmt' w3 [ skipped ]  Example plugin requires the environment variable \$EXAMPLE_PLUGIN to be set (normally done by mtr)
binlog.binlog_unsafe 'stmt'              w4 [ skipped ]  UDF requires the environment variable \$UDF_EXAMPLE_LIB to be set (normally done by mtr)
binlog.binlog_spurious_ddl_errors 'row'  w1 [ skipped ]  Example plugin requires the environment variable \$EXAMPLE_PLUGIN to be set (normally done by mtr)
```
共 4 个。

### 4. federated

#### 4.1 被跳过的测试

##### 4.1.1 federated plugin not available

federated 插件不可用，导致测试被跳过。

涉及测试：
```
federated.federated_plugin
```
共 1 个。

##### 4.1.2 Need bug25714 test program

需要 bug25714 测试程序。

涉及测试：
```
federated.federated_bug_25714
```
共 1 个。

### 5. rpl

#### 5.1 失败的测试

```
rpl.rpl_sbm_previous_gtid_event
rpl.rpl_current_user
rpl.rpl_ssl
```
共 3 个。

##### 5.1.1 SSL 加密算法与预期不同

测试预期使用的 SSL 加密算法与当前进行测试时使用的加密算法不一致导致测试结果与预期不一致。

涉及测试：
```
rpl.rpl_ssl
```
共 1 个

错误信息如下：

rpl.rpl_ssl：
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

##### 5.1.2 Test assertion failed

测试中 assertion 失败，具体原因待查。

涉及测试：
```
rpl.rpl_sbm_previous_gtid_event
```
共 1 个。

##### 5.1.3 Result content mismatch

用户名长度限制由 32 位变为 80 位，测试更新了，但是测试预期结果为更新，导致测试结果与预期不一致。

涉及测试：
```
rpl.rpl_current_user
```
共 1 个。

错误信息如下：

rpl.rpl_current_user：
```
-GRANT CREATE USER ON *.* TO '012345678901234567890123456789012'@'fakehost';
-ERROR HY000: String '012345678901234567890123456789012' is too long for user name (should be no longer than 32)
+GRANT CREATE USER ON *.* TO 'abcdefghij1234567890abcdefghij1234567890abcdefghij1234567890abcdefghij1234567890a'@'fakehost';
+ERROR HY000: String 'abcdefghij1234567890abcdefghij1234567890abcdefghij1234567890abcdefghij' is too long for user name (should be no longer than 80)
 # the host name is too long
 GRANT CREATE USER ON *.* TO 'fakename'@'0123456789012345678901234567890123456789012345678901234567890';
 ERROR HY000: String '0123456789012345678901234567890123456789012345678901234567890' is too long for host name (should be no longer than 60)
```

#### 5.2 跳过的测试

##### 5.2.1 Test makes sense only to run with MTS

测试需要使用 MTS 来运行，但是未能确定 MTS 为何。

涉及测试：
```
rpl.rpl_stm_mix_mts_show_relaylog_events
rpl.rpl_parallel_worker_error
rpl.rpl_mts_relay_log_post_crash_recovery
rpl.rpl_mts_relay_log_recovery_on_error
rpl.rpl_row_mts_show_relaylog_events
rpl.rpl_mts_stop_slave
```

共 6 个。

##### 5.2.2 This test needs on slave side: InnoDB disabled, default engine: MyISAM

测试需要从库默认使用 MyISAM 引擎，并禁用 InnoDB 引擎，设置方法待查。

涉及测试：
```
rpl.rpl_ddl
```
共 1 个。

##### 5.2.5 Test requires: 'have_binlog_format_row'

测试需要 binlog-format 为 row，但是设置为 row 后又显示不支持 row 格式，疑为测试 bug。

涉及测试：
```
rpl.rpl_row_loaddata_concurrent
```
共 1 个。

##### 5.2.4 其他

需要特殊的设置或者特殊编译程序，但是不能合适的设置或者没有找到合适设置方法

```
rpl.rpl_plugin_load               w4 [ skipped ]  Example plugin requires the environment variable \$EXAMPLE_PLUGIN to be set (normally done by mtr)
rpl.rpl_udf                        w3 [ skipped ]  UDF requires the environment variable \$UDF_EXAMPLE_LIB to be set (normally done by mtr)
rpl.rpl_mixed_implicit_commit_binlog 'mix' w3 [ skipped ]  UDF requires the environment variable \$UDF_EXAMPLE_LIB to be set (normally done by mtr)
rpl.rpl_row_implicit_commit_binlog 'row' w3 [ skipped ]  UDF requires the environment variable \$UDF_EXAMPLE_LIB to be set (normally done by mtr)
rpl.rpl_stm_implicit_commit_binlog 'stmt' w3 [ skipped ]  UDF requires the environment variable \$UDF_EXAMPLE_LIB to be set (normally done by mtr)
```
共 5 个。

### 6. rpl_recovery

#### 6.1 跳过的测试

##### 6.1.1 其他

需要特殊的设置或者特殊编译程序，但是不能合适的设置或者没有找到合适设置方法

```
rpl_recovery.rpl_crash_safe_idempotent_master_binlog_format w3 [ skipped ]  Test requires: 'have_slave_use_idempotent_for_recovery'
rpl_recovery.rpl_gtid_mts_stress_crash 'row-idempotent-recovery' w3 [ skipped ]  Test cannot run with idempotent recovery
rpl_recovery.rpl_gtid_stress_crash 'row-idempotent-recovery' w1 [ skipped ]  Test cannot run with idempotent recovery
rpl_recovery.rpl_gtid_crash_safe 'row-idempotent-recovery' w1 [ skipped ]  Test cannot run with idempotent recovery
rpl_recovery.rpl_gtid_crash_safe_idempotent 'row' w1 [ skipped ]  Test requires: 'have_slave_use_idempotent_for_recovery'
```
共 5 个。

### 7. perfschema

#### 7.1 跳过的测试

##### 7.1.1 Need windows

测试需要程序运行在 Windows 平台上。

涉及测试：
```
perfschema.socket_instances_func_win
perfschema.socket_summary_by_instance_func_win
```
共 2 个。

##### 7.1.2 Need open_files_limit to be at least 5000

需要 open_files_limit 设置为至少 5000，但是未能正确确认，待处理。

涉及测试：
```
perfschema.sizing_default
```

### 8. funcs_1

#### 8.1 跳过的测试

##### 8.1.1 Test requires: embedded server

测试需要嵌入式版程序。

涉及测试：
```
funcs_1.is_triggers_embedded
funcs_1.is_views_embedded
funcs_1.is_schemata_embedded
funcs_1.is_statistics_mysql_embedded
funcs_1.is_columns_is_embedded
funcs_1.is_table_constraints_mysql_embedded
funcs_1.is_columns_myisam_embedded
funcs_1.is_tables_embedded
funcs_1.is_columns_mysql_embedded
funcs_1.is_tables_myisam_embedded
funcs_1.is_tables_mysql_embedded
funcs_1.is_key_column_usage_embedded
funcs_1.is_routines_embedded
```
共 13 个。

##### 8.1.2 Test requires: ps-protocol enabled, other protocols disabled

需要使用 ps-protocol 协议，并禁用其他协议。

涉及测试：
```
funcs_1.processlist_priv_ps
funcs_1.processlist_val_ps
```
共 2 个。

### 9. opt_trace

#### 9.1 跳过的测试

##### 9.1.1 Need ps-protocol

需要使用 ps-protocol 协议。

涉及测试：
```
opt_trace.bugs_ps_prot_none
opt_trace.bugs_ps_prot_all
opt_trace.security_ps_prot
opt_trace.range_ps_prot
opt_trace.subquery_ps_prot
opt_trace.general_ps_prot_all
opt_trace.general_ps_prot_none
opt_trace.general2_ps_prot
```

共 8 个。

### 10. parts

#### 10.1 跳过的测试

##### 10.1.1 Test requires: 'lowercase2'

测试需要数据库设置 lowercase2，但未能找到正确的设置方法，待确认。

涉及测试：
```
parts.partition_mgm_lc2_archive
parts.partition_mgm_lc2_memory
parts.partition_mgm_lc2_myisam
parts.partition_mgm_lc2_innodb
```
共 4 个。

##### 10.1.2 Need open_files_limit >= 16450 (see ulimit -n)

测试需要 open_files_limit 设置为大于等于 16450，但未能找到正确的设置方法，待确认。

涉及测试：
```
parts.partition_max_parts_hash_myisam
parts.partition_max_parts_inv_myisam
parts.partition_max_parts_key_myisam
parts.partition_max_parts_list_myisam
parts.partition_max_parts_range_myisam
parts.partition_max_sub_parts_key_list_myisam
parts.partition_max_sub_parts_key_range_myisam
parts.partition_max_sub_parts_list_myisam
parts.partition_max_sub_parts_range_myisam
```
共 9 个。

##### 10.1.3 CAST() in partitioning function is currently not supported.

partitioning function 的 CAST 函数当前版本不支持。

涉及测试：
```
parts.partition_value_myisam
parts.partition_value_innodb
```

共 2 个。

### 11. auth_sec

#### 11.1 失败的测试

##### 11.1.1 SSL 库版本不同

测试预期的 SSL 版本与当前数据库使用的 SSL 版本不同。

涉及测试：
```
auth_sec.cert_verify
```
共 1 个。

错误信息如下：

auth_sec.cert_verify：
```
 #T1: Host name (/CN=localhost/) as OU name in the server certificate, server certificate verification should fail.
 #T2: Host name (localhost) as common name in the server certificate, server certificate verification should pass.
 Variable_name	Value
-Ssl_version	TLS_VERSION
+Ssl_version	TLS_VERSION.2
 # restart server using restart
```

#### 11.2 跳过的测试

##### 11.2.1 Need windows

测试需要程序运行在 Windows 平台上。

涉及测试：
```
auth_sec.secure_file_priv_warnings_win
```

##### 11.2.2 Need YaSSL support

需要 YaSSL 支持。

涉及测试：
```
auth_sec.server_withoutssl_client_withssl
auth_sec.server_withoutssl_client_withoutssl
```
共 2 个。

### 12. xtrabackup
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

找不到可执行文件 `MYSQL_INNOBACKUPEX`，待解决。

### 13. engines/rr_trx
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

测试未正确的初始化，测试程序有错，原版 MySQL 5.6.36 也全部失败。

### 14. large_tests

#### 12.1 跳过的测试

##### 12.1.1 Need open_files_limit to be greater than 6100

需要 open_files_limit 设置为大于等于 6100，但未能找到正确设置方法，待确认。
6100
涉及测试：
```
large_tests.lock_tables_big
```

### 15. perfschema_stress

#### 15.1 失败的测试

##### 15.1.1 performance_schema 数据库中相关表结构有差异

因测试为及时更新，当前版本的 performance_schema 数据库中表 events_waits_current、events_waits_history 的结构与测试预期不一致。

涉及测试：

```
perfschema_stress.read
```
共 1 个。

错误信息如下：

perfschema_stress.read：
```
 NAME	ENABLED	TIMED
 SELECT * FROM performance_schema.events_waits_current
 WHERE (TIMER_END - TIMER_START != TIMER_WAIT);
-THREAD_ID	EVENT_ID	EVENT_NAME	SOURCE	TIMER_START	TIMER_END	TIMER_WAIT	SPINS	OBJECT_SCHEMA	OBJECT_NAME	OBJECT_TYPE	OBJECT_INSTANCE_BEGIN	NESTING_EVENT_ID	OPERATION	NUMBER_OF_BYTESFLAGS
+THREAD_ID	EVENT_ID	END_EVENT_ID	EVENT_NAME	SOURCE	TIMER_START	TIMER_END	TIMER_WAIT	SPINS	OBJECT_SCHEMA	OBJECT_NAME	INDEX_NAME	OBJECT_TYPE	OBJECT_INSTANCE_BEGIN	NESTING_EVENT_ID	NESTING_EVENT_TYPE	OPERATION	NUMBER_OF_BYTES	FLAGS
 SELECT * FROM performance_schema.events_waits_history
 WHERE SPINS != NULL;
-THREAD_ID	EVENT_ID	EVENT_NAME	SOURCE	TIMER_START	TIMER_END	TIMER_WAIT	SPINS	OBJECT_SCHEMA	OBJECT_NAME	OBJECT_TYPE	OBJECT_INSTANCE_BEGIN	NESTING_EVENT_ID	OPERATION	NUMBER_OF_BYTESFLAGS
+THREAD_ID	EVENT_ID	END_EVENT_ID	EVENT_NAME	SOURCE	TIMER_START	TIMER_END	TIMER_WAIT	SPINS	OBJECT_SCHEMA	OBJECT_NAME	INDEX_NAME	OBJECT_TYPE	OBJECT_INSTANCE_BEGIN	NESTING_EVENT_ID	NESTING_EVENT_TYPE	OPERATION	NUMBER_OF_BYTES	FLAGS
 SELECT * FROM performance_schema.threads p,
 performance_schema.events_waits_current e
 WHERE p.THREAD_ID = e.THREAD_ID
 AND TIMER_START = 0
 ORDER BY e.EVENT_ID;
-THREAD_ID	NAME	TYPE	PROCESSLIST_ID	PROCESSLIST_USER	PROCESSLIST_HOST	PROCESSLIST_DB	PROCESSLIST_COMMAND	PROCESSLIST_TIME	PROCESSLIST_STATE	PROCESSLIST_INFO	PARENT_THREAD_ID	ROLE	INSTRUMENTED	THREAD_ID	EVENT_ID	EVENT_NAME	SOURCE	TIMER_START	TIMER_END	TIMER_WAIT	SPINS	OBJECT_SCHEMA	OBJECT_NAME	OBJECT_TYPE	OBJECT_INSTANCE_BEGIN	NESTING_EVENT_ID	OPERATION	NUMBER_OF_BYTES	FLAGS
+THREAD_ID	NAME	TYPE	PROCESSLIST_ID	PROCESSLIST_USER	PROCESSLIST_HOST	PROCESSLIST_DB	PROCESSLIST_COMMAND	PROCESSLIST_TIME	PROCESSLIST_STATE	PROCESSLIST_INFO	PARENT_THREAD_ID	ROLE	INSTRUMENTED	THREAD_ID	EVENT_ID	END_EVENT_ID	EVENT_NAME	SOURCE	TIMER_START	TIMER_END	TIMER_WAIT	SPINS	OBJECT_SCHEMA	OBJECT_NAME	INDEX_NAME	OBJECT_TYPE	OBJECT_INSTANCE_BEGIN	NESTING_EVENT_ID	NESTING_EVENT_TYPE	OPERATION	NUMBER_OF_BYTES	FLAGS
 SELECT * FROM performance_schema.events_waits_current
 WHERE THREAD_ID IN (SELECT THREAD_ID
 FROM performance_schema.threads
@@ -20,7 +20,7 @@
 AND TIMER_END = 0
 AND TIMER_WAIT != NULL
 ORDER BY EVENT_ID;
-THREAD_ID	EVENT_ID	EVENT_NAME	SOURCE	TIMER_START	TIMER_END	TIMER_WAIT	SPINS	OBJECT_SCHEMA	OBJECT_NAME	OBJECT_TYPE	OBJECT_INSTANCE_BEGIN	NESTING_EVENT_ID	OPERATION	NUMBER_OF_BYTESFLAGS
+THREAD_ID	EVENT_ID	END_EVENT_ID	EVENT_NAME	SOURCE	TIMER_START	TIMER_END	TIMER_WAIT	SPINS	OBJECT_SCHEMA	OBJECT_NAME	INDEX_NAME	OBJECT_TYPE	OBJECT_INSTANCE_BEGIN	NESTING_EVENT_ID	NESTING_EVENT_TYPE	OPERATION	NUMBER_OF_BYTES	FLAGS
 SELECT SUM(COUNT_READ) AS sum_count_read,
 SUM(COUNT_WRITE) AS sum_count_write,
 SUM(SUM_NUMBER_OF_BYTES_READ) AS sum_num_bytes_read,
```

##### 15.1.2 测试未产生任何输出

测试未产生任何输出，原因待确认。

涉及测试：
```
perfschema_stress.setup
```
共 1 个。
