
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

这里考虑脚本有问题，修改脚本

```
MYSQL_INNOBACKUPEX 在被使用时仍然为空，修改测试脚本 xb_run.sh，
增加： MYSQL_INNOBACKUPEX=xtrabackup
替换：$ibbackup_opt $backup_dir 为 "--backup="$backup_dir

```
脚本正常运行，但是报错在 ssl 相关，可参看"安全性异常"里的介绍。
