
以下是备份相关的异常 case，

```
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

异常原因

```suite/xtrabackup/include/xb_run.sh: line 21: --defaults-file=/newssd1/temp/mysql-on-terarkdb-4.8-bmi2-0/mysql-test/var/my.cnf: No such file or directory```
