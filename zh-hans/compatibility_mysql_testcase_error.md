
以下列出的 case 是测试程序本身即有问题，

### engines/rr_trx suite 下的异常 case

```
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
测试未正确的初始化，原版 MySQL 5.6.36 也全部失败。


### perfschema_stress suite 下的异常 case

因测试未及时更新，当前版本的 performance_schema 数据库中表 events_waits_current、events_waits_history 的字段个数与测试预期不一致。

```
perfschema_stress.read
```

### rpl suite 下的异常 case

```
rpl.rpl_sbm_previous_gtid_event
rpl.rpl_stm_mix_mts_show_relaylog_events
rpl.rpl_parallel_worker_error
rpl.rpl_mts_relay_log_post_crash_recovery
rpl.rpl_mts_relay_log_recovery_on_error
rpl.rpl_row_mts_show_relaylog_events
rpl.rpl_mts_stop_slave
rpl.rpl_row_loaddata_concurrent
```

错误原因，binlog-format 不支持 'row'

```
rpl.rpl_sbm_previous_gtid_event 'mix'    [ skipped ]  Doesn't support --binlog-format='mixed'
rpl.rpl_sbm_previous_gtid_event 'row'    [ skipped ]  Doesn't support --binlog-format='row'
rpl.rpl_sbm_previous_gtid_event 'stmt'   w2 [ skipped ]  Test requires GTID_MODE=ON.
```

#### Test requires: 'lowercase2'

测试程序设置的```lower_case_table_name2=2```未能起作用。

```
parts.partition_mgm_lc2_archive
parts.partition_mgm_lc2_memory
parts.partition_mgm_lc2_myisam
parts.partition_mgm_lc2_innodb
```

（DONE）
