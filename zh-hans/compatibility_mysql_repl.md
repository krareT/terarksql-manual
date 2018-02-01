
### binlog suite 下的异常 case

```
binlog.binlog_gtid_mysqlbinlog_row_innodb
binlog.binlog_gtid_mysqlbinlog_row
binlog.binlog_gtid_mysqlbinlog_row_myisam
```

错误原因

```
-#010909  4:46:40 server id 1  end_log_pos # CRC32 #    Write_rows: table id # flags: STMT_END_F
+#010909  4:46:40 server id 1  end_log_pos #    Write_rows: table id # flags: STMT_END_F
```

如果在 default_mysqld.cnf 里增加 ```binlog_checksum=CRC32``` 则上述问题消失，但是会出现下述错误

```
-###   @3=-128 (128) /* TINYINT meta=0 nullable=1 is_null=0 */
-###   @4=0 /* TINYINT meta=0 nullable=1 is_null=0 */
-###   @5=0 /* TINYINT meta=0 nullable=1 is_null=0 */
+###   @3=-128 /* TINYINT meta=0 nullable=1 is_null=0 */
+###   @4=0 /* TINYINT UNSIGNED meta=0 nullable=1 is_null=0 */
+###   @5=0 /* TINYINT UNSIGNED meta=0 nullable=1 is_null=0 */
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

(DONE)

