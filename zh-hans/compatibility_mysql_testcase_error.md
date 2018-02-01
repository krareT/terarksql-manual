
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

（DONE）
