
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

1. 测试中 assertion 失败，具体原因待查。

```
rpl.rpl_sbm_previous_gtid_event
```

2. Result content mismatch

用户名长度限制由 32 位变为 80 位，测试更新了，但是测试预期结果未更新，导致测试结果与预期不一致。

```
rpl.rpl_current_user
```

错误信息如下：

```
-GRANT CREATE USER ON *.* TO '012345678901234567890123456789012'@'fakehost';
-ERROR HY000: String '012345678901234567890123456789012' is too long for user name (should be no longer than 32)
+GRANT CREATE USER ON *.* TO 'abcdefghij1234567890abcdefghij1234567890abcdefghij1234567890abcdefghij1234567890a'@'fakehost';
+ERROR HY000: String 'abcdefghij1234567890abcdefghij1234567890abcdefghij1234567890abcdefghij' is too long for user name (should be no longer than 80)
 # the host name is too long
 GRANT CREATE USER ON *.* TO 'fakename'@'0123456789012345678901234567890123456789012345678901234567890';
 ERROR HY000: String '0123456789012345678901234567890123456789012345678901234567890' is too long for host name (should be no longer than 60)
```

3. Test makes sense only to run with MTS

测试需要使用 Multi-Threaded Slave，待确认问题所在。

```
rpl.rpl_stm_mix_mts_show_relaylog_events
rpl.rpl_parallel_worker_error
rpl.rpl_mts_relay_log_post_crash_recovery
rpl.rpl_mts_relay_log_recovery_on_error
rpl.rpl_row_mts_show_relaylog_events
rpl.rpl_mts_stop_slave
```


