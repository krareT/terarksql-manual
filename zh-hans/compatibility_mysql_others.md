

### 统计信息不一致

我们在 MyRocks 的 information_schema 数据库中添加了一个 ROCKSDB_TF_OPTIONS，以及一个用于 flush `__system__` column family 的线程，这导致了统计与预期不一致。

```
main.mysqld--help-notwin-profiling
main.mysqlshow
```

main.mysqld--help-notwin-profiling：
```
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

### 测试未产生任何输出

```
perfschema_stress.setup
```

原因待确认。

### 测试需要运行在 windows 平台上

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

sys_vars.shared_memory_base_name_basic
sys_vars.shared_memory_basic
sys_vars.named_pipe_basic

perfschema.socket_instances_func_win
perfschema.socket_summary_by_instance_func_win

auth_sec.secure_file_priv_warnings_win
```

### 需要嵌入式 MySQL 程序

```
main.mysql_embedded
main.server_uuid_embedded
main.events_embedded

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

### 需要 32 位版本程序和平台

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

### 需要系统关闭 NUMA 和需要完整的正则支持

```
sys_vars.innodb_numa_interleave_basic
sys_vars.legacy_user_name_pattern_basic
```

### 需要federated 插件

```
federated.federated_plugin
```

### Need bug25714 test program

```
federated.federated_bug_25714
```


