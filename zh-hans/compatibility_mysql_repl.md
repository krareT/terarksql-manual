
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

### rpl.rpl_current_user

修改测试脚本后可以通过，修改方案

```
1. include/default_mysqld.cnf 后面增加参数

gtid-mode=on
log-slave-updates=on
enforce-gtid-consistency=on
binlog_checksum=CRC32

2. suite/rpl/r/rpl_current_user.result 如下修改

-- GRANT CREATE USER ON *.* TO '012345678901234567890123456789012'@'fakehost';
-- ERROR HY000: String '012345678901234567890123456789012' is too long for user name (should be no longer than 32)
++ GRANT CREATE USER ON *.* TO 'abcdefghij1234567890abcdefghij1234567890abcdefghij1234567890abcdefghij1234567890a'@'fakehost';
++ ERROR HY000: String 'abcdefghij1234567890abcdefghij1234567890abcdefghij1234567890abcdefghij' is too long for user name (should be no longer than 80)

```

(UPDATED)

