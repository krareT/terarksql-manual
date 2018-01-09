
以下 case 均需要相关设置，但因为暂时没有找到合适设置方法从而导致被跳过，

```
main.bug46261                            w1 [ skipped ]  Example plugin requires the environment variable \$EXAMPLE_PLUGIN to be set (normally done by mtr)
main.plugin_load                         w3 [ skipped ]  Example plugin requires the environment variable \$EXAMPLE_PLUGIN to be set (normally done by mtr)
main.plugin_load_option                  w4 [ skipped ]  Example plugin requires the environment variable \$EXAMPLE_PLUGIN to be set (normally done by mtr)
main.plugin                              w1 [ skipped ]  Example plugin requires the environment variable \$EXAMPLE_PLUGIN to be set (normally done by mtr)
main.plugin_not_embedded                 w1 [ skipped ]  Example plugin requires the environment variable \$EXAMPLE_PLUGIN to be set (normally done by mtr)
main.multi_plugin_load                   w2 [ skipped ]  Need the plugin test_plugin_server
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
main.not_partition                       w1 [ skipped ]  Test requires: 'true'
main.system_mysql_db_fix40123            w3 [ skipped ]  Test need MYSQL_FIX_PRIVILEGE_TABLES
main.system_mysql_db_fix50030            w3 [ skipped ]  Test needs MYSQL_FIX_PRIVILEGE_TABLES
main.system_mysql_db_fix50117            w3 [ skipped ]  Test needs MYSQL_FIX_PRIVILEGE_TABLES
main.fix_priv_tables                     w3 [ skipped ]  Test need MYSQL_FIX_PRIVILEGE_TABLES
main.timezone3                           w3 [ skipped ]  Test requires: 'have_moscow_leap_timezone'
main.lowercase_mixed_tmpdir_innodb       w2 [ skipped ]  Test requires: 'lowercase2'
main.lowercase_table2                    w3 [ skipped ]  Test requires: 'lowercase2'
main.slow_log_legacy_user                w1 [ skipped ]  Test requires full regex (not supported in gcc 4.8 and prior)
main.dynamic_tracing                     w1 [ skipped ]  dtrace/stap tool requires additional privileges to run this test.
main.func_encrypt_nossl                  w4 [ skipped ]  Test requires: 'not_openssl'


binlog.binlog_spurious_ddl_errors 'mix'  w2 [ skipped ]  Example plugin requires the environment variable \$EXAMPLE_PLUGIN to be set (normally done by mtr)
binlog.binlog_spurious_ddl_errors 'stmt' w3 [ skipped ]  Example plugin requires the environment variable \$EXAMPLE_PLUGIN to be set (normally done by mtr)
binlog.binlog_spurious_ddl_errors 'row'  w1 [ skipped ]  Example plugin requires the environment variable \$EXAMPLE_PLUGIN to be set (normally done by mtr)

rpl.rpl_plugin_load w4 [ skipped ]  Example plugin requires the environment variable \$EXAMPLE_PLUGIN to be set (normally done by mtr)

rpl_recovery.rpl_crash_safe_idempotent_master_binlog_format w3 [ skipped ]  Test requires: 'have_slave_use_idempotent_for_recovery'
rpl_recovery.rpl_gtid_mts_stress_crash 'row-idempotent-recovery' w3 [ skipped ]  Test cannot run with idempotent recovery
rpl_recovery.rpl_gtid_stress_crash 'row-idempotent-recovery' w1 [ skipped ]  Test cannot run with idempotent recovery
rpl_recovery.rpl_gtid_crash_safe 'row-idempotent-recovery' w1 [ skipped ]  Test cannot run with idempotent recovery
rpl_recovery.rpl_gtid_crash_safe_idempotent 'row' w1 [ skipped ]  Test requires: 'have_slave_use_idempotent_for_recovery'

```

#### Test requires: 'lowercase2'

测试需要数据库设置 lowercase2，但未能找到正确的设置方法，待确认。

```
parts.partition_mgm_lc2_archive
parts.partition_mgm_lc2_memory
parts.partition_mgm_lc2_myisam
parts.partition_mgm_lc2_innodb
```

#### CAST() in partitioning function is currently not supported.

partitioning function 的 CAST 函数当前版本不支持。

```
parts.partition_value_myisam
parts.partition_value_innodb
```

#### Need ps-protocol

需要使用 ps-protocol 协议。

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

#### Test requires: ps-protocol enabled, other protocols disabled

需要使用 ps-protocol 协议，并禁用其他协议。

```
funcs_1.processlist_priv_ps
funcs_1.processlist_val_ps
```

#### Test makes sense only to run with MTS

测试需要使用 MTS 来运行，但是未能确定 MTS 为何。

```
rpl.rpl_stm_mix_mts_show_relaylog_events
rpl.rpl_parallel_worker_error
rpl.rpl_mts_relay_log_post_crash_recovery
rpl.rpl_mts_relay_log_recovery_on_error
rpl.rpl_row_mts_show_relaylog_events
rpl.rpl_mts_stop_slave
```

#### This test needs on slave side: InnoDB disabled, default engine: MyISAM

测试需要从库默认使用 MyISAM 引擎，并禁用 InnoDB 引擎，设置方法待查。

```
rpl.rpl_ddl
```

#### ndbcluster disabled

ndbcluster 未启用，测试跳过。
```
binlog.binlog_multi_engine
```

#### 需要系统关闭 NUMA 和需要完整的正则支持

```
sys_vars.innodb_numa_interleave_basic
sys_vars.legacy_user_name_pattern_basic
```



