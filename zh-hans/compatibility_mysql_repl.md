
### binlog suite 下的异常 case

```
binlog.binlog_gtid_mysqlbinlog_row_innodb
binlog.binlog_gtid_mysqlbinlog_row
binlog.binlog_gtid_mysqlbinlog_row_myisam
```

错误原因

```
binlog.binlog_gtid_mysqlbinlog_row 'mix' [ skipped ]  Doesn't support --binlog-format='mixed'
binlog.binlog_gtid_mysqlbinlog_row 'stmt' [ skipped ]  Doesn't support --binlog-format='statement'
binlog.binlog_gtid_mysqlbinlog_row 'row' w1 [ skipped ]  Test requires GTID_MODE=ON.
```


### rpl suite 下的异常 case

```
rpl.rpl_sbm_previous_gtid_event
rpl.rpl_current_user
rpl.rpl_stm_mix_mts_show_relaylog_events
rpl.rpl_parallel_worker_error
rpl.rpl_mts_relay_log_post_crash_recovery
rpl.rpl_mts_relay_log_recovery_on_error
rpl.rpl_row_mts_show_relaylog_events
rpl.rpl_mts_stop_slave
rpl.rpl_row_loaddata_concurrent
```

错误原因

```
rpl.rpl_sbm_previous_gtid_event 'mix'    [ skipped ]  Doesn't support --binlog-format='mixed'
rpl.rpl_sbm_previous_gtid_event 'row'    [ skipped ]  Doesn't support --binlog-format='row'
rpl.rpl_sbm_previous_gtid_event 'stmt'   w2 [ skipped ]  Test requires GTID_MODE=ON.
```

[UPDATED]

