
### binlog suite 下的异常 case

```
binlog.binlog_gtid_mysqlbinlog_row_innodb
binlog.binlog_gtid_mysqlbinlog_row
binlog.binlog_gtid_mysqlbinlog_row_myisam
```

共 3 个，按其失败的原因可以分为以下几类:

1. 测试中测试程序失去连接

在测试中，测试程序失去连接，不能连接到数据库。失去连接的原因待确定。

```
binlog.binlog_gtid_mysqlbinlog_row_innodb
binlog.binlog_gtid_mysqlbinlog_row_myisam
```


2. 开启 GTID MODE 后 binlog 格式不一致

开启 GTID MODE 后 binlog 格式不一致，具体原因待确认。

```
binlog.binlog_gtid_mysqlbinlog_row
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

