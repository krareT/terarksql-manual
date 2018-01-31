# 对比原版 MyRocks 的兼容性

## 总结

MyRocks 针对自身特点提供了一系列的测试,这些测试共有五类：

- rocksdb
- rocksdb_hotbackup
- rocksdb_rpl
- rocksdb_stress
- rocksdb_sys_vars

针对这几类测试，以下是 MySQL on TerarkDB 的运行结果如下，其中部分失败的原因，在详细说明中做了阐述：

| suite             | total | success | fail |
| ----------------- |:-----:|:-----:|:-----:|
| rocksdb           |  242  |  214  |   28  |
| rocksdb_stress    |   2   |   2   |    0  |
| rocksdb_sys_vars  |  110  |  108  |    2  |
| rocksdb_hotbackup |   6   |   4   |    2  |
| rocksdb_rpl       |   15  |   15  |    0  |

## 详细说明

根据目前测试，针对 MyRocks 的功能测试，只有以下少数 MyRocks 官方原版可以通过但 Terark 不能通过的测试。

### 1. rocksdb
#### 1.1 rocksdb.show_engine

相关语句：`SELECT * FROM INFORMATION_SCHEMA.ROCKSDB_CF_OPTIONS;` MySQL on TerarkDB 在 INFORMATION_SCHEMA 数据库的表 ROCKSDB_CF_OPTIONS 中添加了 TABLE_FACTORY_NAME 相关的记录，用于显示所使用的 `table factory`，故与 MyRocks 预期结果不一致。

添加内容如下：
```
select * from information_schema.ROCKSDB_CF_OPTIONS where OPTION_TYPE='TABLE_FACTORY_NAME';
+------------+--------------------+----------------+
| CF_NAME    | OPTION_TYPE        | VALUE          |
+------------+--------------------+----------------+
| __system__ | TABLE_FACTORY_NAME | TerarkZipTable |
| default    | TABLE_FACTORY_NAME | TerarkZipTable |
+------------+--------------------+----------------+
```

MyRocks 官方在最新的 release [prod201704](https://github.com/facebook/mysql-5.6/releases/tag/prod201704) 中在表 ```INFORMATION_SCHEMA.ROCKSDB_CF_OPTIONS``` 添加了若干变量，但测试程序未及时更新，导致与测试程序预期结果不符合。

添加内容类似如下：
```
SELECT * FROM INFORMATION_SCHEMA.ROCKSDB_CF_OPTIONS;
 CF_NAME        OPTION_TYPE     VALUE
+default        COMPARATOR      #
+default        MERGE_OPERATOR  #
+default        COMPACTION_FILTER       #
+default        COMPACTION_FILTER_FACTORY       #
+default        WRITE_BUFFER_SIZE       #
+default        MAX_WRITE_BUFFER_NUMBER #
+default        MIN_WRITE_BUFFER_NUMBER_TO_MERGE        #
+default        NUM_LEVELS      #
+default        LEVEL0_FILE_NUM_COMPACTION_TRIGGER      #
+default        LEVEL0_SLOWDOWN_WRITES_TRIGGER  #
+default        LEVEL0_STOP_WRITES_TRIGGER      #
+default        MAX_MEM_COMPACTION_LEVEL        #
+default        TARGET_FILE_SIZE_BASE   #
...
```

#### 1.2 rocksdb.mysqldump2

相关语句：
```
select case when variable_value - @a > 100 then 'true' else 'false' end 
       from information_schema.global_status 
       where variable_name='rocksdb_block_cache_add';
```

预期结果：
```
case when variable_value - @a > 100 then 'true' else 'false' end
true
```

测试结果：
```
case when variable_value - @a > 100 then 'true' else 'false' end`
false
```

MySQL on TerarkDB 不使用 block cache，故相关统计数据与预期不符，不影响功能。

#### 1.3 rocksdb.cardinality

相关语句：`show index in t1;`

预期结果：
```
Table	Non_unique	Key_name	Seq_in_index	Column_name	Collation	Cardinality	Sub_part	Packed	Null	Index_type	Comment	Index_comment
t1	0	PRIMARY	1	id	A	100000	NULL	NULL		LSMTREE		
t1	1	t1_1	1	id	A	100000	NULL	NULL		LSMTREE		
t1	1	t1_1	2	i1	A	100000	NULL	NULL	YES	LSMTREE		
t1	1	t1_2	1	i1	A	100000	NULL	NULL	YES	LSMTREE		
t1	1	t1_2	2	i2	A	100000	NULL	NULL	YES	LSMTREE		
t1	1	t1_3	1	i2	A	11111	NULL	NULL	YES	LSMTREE		
t1	1	t1_3	2	i1	A	100000	NULL	NULL	YES	LSMTREE		
t1	1	t1_4	1	c1	A	100000	NULL	NULL	YES	LSMTREE		
t1	1	t1_4	2	c2	A	100000	NULL	NULL	YES	LSMTREE		
t1	1	t1_5	1	c2	A	11111	NULL	NULL	YES	LSMTREE		
t1	1	t1_5	2	c1	A	100000	NULL	NULL	YES	LSMTREE
```

测试结果：
```
Table	Non_unique	Key_name	Seq_in_index	Column_name	Collation	Cardinality	Sub_part	Packed	Null	Index_type	Comment	Index_comment
t1	0	PRIMARY	1	id	A	100595	NULL	NULL		LSMTREE		
t1	1	t1_1	1	id	A	100595	NULL	NULL		LSMTREE		
t1	1	t1_1	2	i1	A	100595	NULL	NULL	YES	LSMTREE		
t1	1	t1_2	1	i1	A	100595	NULL	NULL	YES	LSMTREE		
t1	1	t1_2	2	i2	A	100595	NULL	NULL	YES	LSMTREE		
t1	1	t1_3	1	i2	A	11177	NULL	NULL	YES	LSMTREE		
t1	1	t1_3	2	i1	A	100595	NULL	NULL	YES	LSMTREE		
t1	1	t1_4	1	c1	A	100595	NULL	NULL	YES	LSMTREE		
t1	1	t1_4	2	c2	A	100595	NULL	NULL	YES	LSMTREE		
t1	1	t1_5	1	c2	A	11177	NULL	NULL	YES	LSMTREE		
t1	1	t1_5	2	c1	A	100595	NULL	NULL	YES	LSMTREE
```
相关语句：`SELECT table_name, table_rows FROM information_schema.tables WHERE table_schema = DATABASE();`

预期结果：
```
table_name	table_rows
t1	100000
```

测试结果：
```
table_name	table_rows
t1	100403
```

`rocksdb.cardinality` 测试的输出如下：
```
show index in t1;
 Table	Non_unique	Key_name	Seq_in_index	Column_name	Collation	Cardinality	Sub_part	Packed	Null	Index_type	Comment	Index_comment
-t1	0	PRIMARY	1	id	A	100000	NULL	NULL		LSMTREE		
-t1	1	t1_1	1	id	A	100000	NULL	NULL		LSMTREE		
-t1	1	t1_1	2	i1	A	100000	NULL	NULL	YES	LSMTREE		
-t1	1	t1_2	1	i1	A	100000	NULL	NULL	YES	LSMTREE		
-t1	1	t1_2	2	i2	A	100000	NULL	NULL	YES	LSMTREE		
-t1	1	t1_3	1	i2	A	11111	NULL	NULL	YES	LSMTREE		
-t1	1	t1_3	2	i1	A	100000	NULL	NULL	YES	LSMTREE		
-t1	1	t1_4	1	c1	A	100000	NULL	NULL	YES	LSMTREE		
-t1	1	t1_4	2	c2	A	100000	NULL	NULL	YES	LSMTREE		
-t1	1	t1_5	1	c2	A	11111	NULL	NULL	YES	LSMTREE		
-t1	1	t1_5	2	c1	A	100000	NULL	NULL	YES	LSMTREE		
+t1	0	PRIMARY	1	id	A	100403	NULL	NULL		LSMTREE		
+t1	1	t1_1	1	id	A	100403	NULL	NULL		LSMTREE		
+t1	1	t1_1	2	i1	A	100403	NULL	NULL	YES	LSMTREE		
+t1	1	t1_2	1	i1	A	100403	NULL	NULL	YES	LSMTREE		
+t1	1	t1_2	2	i2	A	100403	NULL	NULL	YES	LSMTREE		
+t1	1	t1_3	1	i2	A	11155	NULL	NULL	YES	LSMTREE		
+t1	1	t1_3	2	i1	A	100403	NULL	NULL	YES	LSMTREE		
+t1	1	t1_4	1	c1	A	100403	NULL	NULL	YES	LSMTREE		
+t1	1	t1_4	2	c2	A	100403	NULL	NULL	YES	LSMTREE		
+t1	1	t1_5	1	c2	A	11155	NULL	NULL	YES	LSMTREE		
+t1	1	t1_5	2	c1	A	100403	NULL	NULL	YES	LSMTREE		
 SELECT table_name, table_rows FROM information_schema.tables WHERE table_schema = DATABASE();
 table_name	table_rows
-t1	100000
+t1	100403
```

Cardinality 为估计值，且需要在数据写入 SST 后才能计算得出，对功能不影响。

#### 1.4 rocksdb.optimize_table

错误日志：
```
At line 67: command "perl suite/rocksdb/optimize_table_check_sst.pl $MYSQL_TMP_DIR/sst_size.dat" failed

Output from before failure:
checking sst file reduction on optimize table from 0 to 1..
sst file reduction was not enough. 7140->7140 (minimum 1000kb)
```

测试中先创建 6 张表，然后依次插入 `10,000` 条数据，再依次删除 `9,900` 条，最后依次 compact 每张表，比较对每张表进行 compact 前后的空间压缩是否大于 1,000 kb。MySQL on TerarkDb 使用 universal compaction，MyRocks 未对其优化，故 MySQL on TerarkDB 不使用 `optimize table tablename;` 触发主动 compact，且 MySQL on TerarkDB 不能单独对一张表进行 compact。故与测试预期行为不一致，但是这不会对功能有任何影响。

#### 1.5 rocksdb.rocksdb_cf_per_partition

相关语句：`EXPLAIN PARTITIONS SELECT  * FROM t2 WHERE col3 = 0x4 AND col2 = 0x34567;`
预期结果：
```
id	select_type	table	partitions	type	possible_keys	key	key_len	ref	rows	Extra
1	SIMPLE	t2	custom_p2	ref	col3	col3	258	const	1	Using where
```
测试结果：
```
id	select_type	table	partitions	type	possible_keys	key	key_len	ref	rows	Extra
1	SIMPLE	t2	custom_p2	ref	col3	col3	258	const	2	Using where
```
其中 rows 为估计值，因使用的算法不同估算值也略有不同，对功能不会造成影响。

#### 1.6 rocksdb.add_index_inplace

相关语句：`select INDEX_LENGTH from information_schema.tables where table_schema=database() and table_name='t1';`

预期结果：
```
INDEX_LENGTH
1300
```

测试结果：
```
INDEX_LENGTH
1504
```

MySQL on TerarkDB 使用的索引算法与 MyRocks 不同，故 index_length 不同。

#### 1.7 rocksdb.drop_table2, rocksdb.drop_table3， rocksdb.truncate_table3

这一块背景比较复杂，MyRocks 在 drop table 时，会先后做两件事情：
1. 把**只包含该 table(极其 index)** 的 sst 直接删除
2. 执行 CompactRange，彻底删除该 table 所有相关的数据。

对于 Level Compaction，这个策略很高效，但是，MySQL on TerarkDB 默认使用的是 universal compaction，事情就比较麻烦了————

在 universal compaction 中，所有 Level 之间没有重叠的 seqnum（相当于数据插入的时间），从而，要保持这个不变式，就不能按 Key Range 去 Compact 不同 Level 中的数据，所以，在 CompactRange 的实现中，如果是 universal compaction，就忽略 Key Range，总是 compact 所有数据。

于是，在 drop table 时，第一步不受影响，仍然有效，但是第二步非常要命，哪怕我们删除的 table 只占所有数据的万分之一，它也会去 Compact
 所有数据。所以，我们对 MyRocks 做了一个修改，在 drop table 时，如果是 universal compaction，就跳过第二步的 CompactRange。

同时，我们对第一步增加了一个优化：每次 drop table 时，把新被 drop 的 table(及其 index) 加入被删除列表，如果某个 SST 只包含被删除列表中的数据，就把这个 SST 删除。———因为跳过了第二步，drop table 时会遗留数据，drop 多个 table，遗留的数据会增加，这些数据只要在 Key Range 上连续起来，就很有可能完全覆盖了某个(或多个) SST。

到此为止，这个用例跑不通就可以理解了，当然，如果我们在 drop table 之后，主动对所有 cf 进行 compact，测试就可以通过了。

#### 1.8 rocksdb.rocksdb

MySQL on TerarkDB 增加了一个后台线程用于定时 compact ```__system__``` column family，故统计信息中会多出这个线程的信息，这与预期结果不一致。错误信息如下：

```
--- /newssd1/temp/mysql-on-terarkdb-4.8-bmi2-0/mysql-test/suite/rocksdb/r/rocksdb.result        2018-01-02 14:11:45.000000000 +0300
+++ /oldssd2/tempvar/1/log/rocksdb.reject       2018-01-10 16:09:18.590916677 +0300
@@ -956,6 +956,7 @@
 rocksdb_store_row_debug_checksums      OFF
 rocksdb_strict_collation_check OFF
 rocksdb_strict_collation_exceptions
+rocksdb_system_cf_background_flush_interval    120
 rocksdb_table_cache_numshardbits       6
 rocksdb_table_stats_sampling_pct       10
 rocksdb_tmpdir
```

#### 1.9 rocksdb.compact_deletes

原因同 1.7

#### 1.10 rocksdb.singledelete

相关语句：
```
select case when variable_value-@s = 0 then 'true' else 'false' end 
       from information_schema.global_status 
       where variable_name='rocksdb_number_sst_entry_singledelete';
```

预期结果：
```
case when variable_value-@s = 0 then 'true' else 'false' end
true
```

测试结果：
```
case when variable_value-@s = 0 then 'true' else 'false' end
false
```
singledelete 相关统计数据与 compact 次数相关，而 MySQL on TerarkDB 的 write_buffer_size 默认设置（1G）与该测试中 MyRocks 不一致，write_buffer_size 不同时触发的 compact 次数不同，从而导致相关统计数据不一致，手动设置为与测试中一致（64K）即可通过。

#### 1.11 rocksdb.bulk_load_errors

错误日志：
```
mysqltest: At line 27: query 'SET rocksdb_bulk_load=0' succeeded - should have failed with errno 2013...
```

MyRocks 对 universal compaction 支持不完善，在该模式下使用 MyRocks bulk load 模式时会发生死锁，MySQL on TerarkDB 不支持 MyRocks bulk load 模式，在使用中不能开启（set rocksdb_bulk_load=1）该模式。

#### 1.12 rocksdb.compression_zstd

MySQL on TerarkDB 不使用 zstd 压缩算法。

#### 1.13 rocksdb.bulk_load_rev_data， rocksdb.bulk_load， rocksdb.bulk_load_rev_cf， rocksdb.bulk_load_rev_cf_and_data

相关语句：```SHOW TABLE STATUS WHERE name LIKE 't%';```

预期结果：
```
SHOW TABLE STATUS WHERE name LIKE 't%';
 Name   Engine  Version Row_format      Rows    Avg_row_length  Data_length     Max_data_length Index_length    Data_free       Auto_increment  Create_time     Update_time     Check_time      Collation       Checksum        Create_options  Comment
t1     ROCKSDB 10      Fixed   5000000 #       #       #       #       0       NULL    NULL    NULL    NULL    latin1_bin      NULL
 t2     ROCKSDB 10      Fixed   5000000 #       #       #       #       0       NULL    NULL    NULL    NULL    latin1_bin      NULL
 t3     ROCKSDB 10      Fixed   5000000 #       #       #       #       0       NULL    NULL    NULL    NULL    latin1_bin      NULL    partitioned
```

测试结果：
```
SHOW TABLE STATUS WHERE name LIKE 't%';
 Name   Engine  Version Row_format      Rows    Avg_row_length  Data_length     Max_data_length Index_length    Data_free       Auto_increment  Create_time     Update_time     Check_time      Collation       Checksum        Create_options  Comment
t1     ROCKSDB 10      Fixed   4910123 #       #       #       #       0       NULL    NULL    NULL    NULL    latin1_bin      NULL
 t2     ROCKSDB 10      Fixed   5000000 #       #       #       #       0       NULL    NULL    NULL    NULL    latin1_bin      NULL
 t3     ROCKSDB 10      Fixed   5000000 #       #       #       #       0       NULL    NULL    NULL    NULL    latin1_bin      NULL    partitioned
```

测试输出如下：
```
--- /newssd1/temp/mysql-on-terarkdb-4.8-bmi2-0/mysql-test/suite/rocksdb/r/bulk_load_rev_cf_and_data.result      2018-01-02 14:11:45.000000000 +0300
+++ /oldssd2/tempvar/3/log/bulk_load_rev_cf_and_data.reject     2018-01-23 12:13:48.943023180 +0300
@@ -36,7 +36,7 @@
 set rocksdb_bulk_load=0;
 SHOW TABLE STATUS WHERE name LIKE 't%';
 Name   Engine  Version Row_format      Rows    Avg_row_length  Data_length     Max_data_length Index_length    Data_free       Auto_increment  Create_time     Update_time     Check_time      Collation       Checksum        Create_options  Comment
-t1     ROCKSDB 10      Fixed   5000000 #       #       #       #       0       NULL    NULL    NULL    NULL    latin1_bin      NULL
+t1     ROCKSDB 10      Fixed   4910123 #       #       #       #       0       NULL    NULL    NULL    NULL    latin1_bin      NULL
 t2     ROCKSDB 10      Fixed   5000000 #       #       #       #       0       NULL    NULL    NULL    NULL    latin1_bin      NULL
 t3     ROCKSDB 10      Fixed   5000000 #       #       #       #       0       NULL    NULL    NULL    NULL    latin1_bin      NULL    partitioned
```

rows 为估计值，同 1.5。

#### 1.14 rocksdb.native_procedure

错误信息如下：
```
mysqltest: In included file "./include/native_procedure.inc": 
included from ./include/native_procedure.inc at line 38:
At line 37: Can't initialize replace from 'replace_result $NP_EXAMPLE_LIB NP_EXAMPLE_LIB'
```

官方原版 MyRocks 也不能通过，且 MySQL on TerarkDB 错误信息与原版 MyRocks 一致。

#### 1.15 rocksdb.rocksdb_cf_options

MySQL on TerarkDB 为针对自身特点，使用了不同于原版 MyRocks 的默认 cf options，故与测试预期结果不符合。其错误信息如下：

```
--- /newssd1/temp/mysql-on-terarkdb-4.8-bmi2-0/mysql-test/suite/rocksdb/r/rocksdb_cf_options.result     2017-06-08 06:26:45.000000000 +0300
+++ /oldssd2/tempvar/1/log/rocksdb_cf_options.reject    2018-01-10 16:12:53.374043657 +0300
@@ -17,21 +17,21 @@
 'MAX_BYTES_FOR_LEVEL_MULTIPLIER')
 order by cf_name, option_type;
 cf_name        option_type     value
-cf1    MAX_BYTES_FOR_LEVEL_MULTIPLIER  10.000000
-cf1    TARGET_FILE_SIZE_BASE   1048576
-cf1    WRITE_BUFFER_SIZE       12582912
-cf2    MAX_BYTES_FOR_LEVEL_MULTIPLIER  10.000000
-cf2    TARGET_FILE_SIZE_BASE   1048576
-cf2    WRITE_BUFFER_SIZE       12582912
-default        MAX_BYTES_FOR_LEVEL_MULTIPLIER  10.000000
-default        TARGET_FILE_SIZE_BASE   1048576
-default        WRITE_BUFFER_SIZE       12582912
-z      MAX_BYTES_FOR_LEVEL_MULTIPLIER  10.000000
-z      TARGET_FILE_SIZE_BASE   1048576
-z      WRITE_BUFFER_SIZE       12582912
-__system__     MAX_BYTES_FOR_LEVEL_MULTIPLIER  10.000000
-__system__     TARGET_FILE_SIZE_BASE   1048576
-__system__     WRITE_BUFFER_SIZE       12582912
+cf1    MAX_BYTES_FOR_LEVEL_MULTIPLIER  2.000000
+cf1    TARGET_FILE_SIZE_BASE   2147483648
+cf1    WRITE_BUFFER_SIZE       1073741824
+cf2    MAX_BYTES_FOR_LEVEL_MULTIPLIER  2.000000
+cf2    TARGET_FILE_SIZE_BASE   2147483648
+cf2    WRITE_BUFFER_SIZE       1073741824
+default        MAX_BYTES_FOR_LEVEL_MULTIPLIER  2.000000
+default        TARGET_FILE_SIZE_BASE   2147483648
+default        WRITE_BUFFER_SIZE       1073741824
+z      MAX_BYTES_FOR_LEVEL_MULTIPLIER  2.000000
+z      TARGET_FILE_SIZE_BASE   2147483648
+z      WRITE_BUFFER_SIZE       1073741824
+__system__     MAX_BYTES_FOR_LEVEL_MULTIPLIER  2.000000
+__system__     TARGET_FILE_SIZE_BASE   2147483648
+__system__     WRITE_BUFFER_SIZE       1073741824
```

#### 1.16 rocksdb.records_in_range

相关语句：```explain extended select a, b from t1 where a < 750;```

错误信息类似如下：

```
--- /newssd1/temp/mysql-on-terarkdb-4.8-bmi2-0/mysql-test/suite/rocksdb/r/records_in_range.result       2017-06-08 06:26:45.000000000 +0300
+++ /oldssd2/tempvar/6/log/records_in_range.reject      2018-01-23 11:39:56.283711337 +0300
@@ -56,7 +56,7 @@
 Note   1003    /* select#1 */ select `test`.`t1`.`a` AS `a` from `test`.`t1` where (`test`.`t1`.`a` < 750)
 explain extended select a, b from t1 where a < 750;
 id     select_type     table   type    possible_keys   key     key_len ref     rows    filtered        Extra
-1      SIMPLE  t1      ALL     ka      NULL    NULL    NULL    20000   75.00   Using where
+1      SIMPLE  t1      ALL     ka      NULL    NULL    NULL    19694   76.17   Using where
 Warnings:
 Note   1003    /* select#1 */ select `test`.`t1`.`a` AS `a`,`test`.`t1`.`b` AS `b` from `test`.`t1` where (`test`.`t1`.`a` < 750)
 explain extended select a from t1 where a = 700;
```

其中有 rows 为预估值，同 1.5。

#### 1.17 rocksdb.statistics

相关语句：```SELECT table_name, table_rows 
                  FROM information_schema.tables 
                  WHERE table_schema = DATABASE() and table_name <> 't1';```

错误信息如下：

```
--- /newssd1/temp/mysql-on-terarkdb-4.8-bmi2-0/mysql-test/suite/rocksdb/r/statistics.result     2017-06-08 06:26:45.000000000 +0300
+++ /oldssd2/tempvar/4/log/statistics.reject    2018-01-23 11:42:12.405805520 +0300
@@ -21,8 +21,8 @@
 ) engine=rocksdb;
 SELECT table_name, table_rows FROM information_schema.tables WHERE table_schema = DATABASE() and table_name <> 't1';
 table_name     table_rows
-t2     1000
-t3     1000
+t2     4999
+t3     4999
 SELECT CASE WHEN table_rows < 100000 then 'true' else 'false' end from information_schema.tables where table_name = 't1';
 CASE WHEN table_rows < 100000 then 'true' else 'false' end
 true
```

tables_rows 估计值，同 1.5。

#### 1.17 rocksdb.issue255

相关语句：```SHOW TABLE STATUS LIKE 't1';```

错误信息类似如下：

```
--- /newssd1/temp/mysql-on-terarkdb-4.8-bmi2-0/mysql-test/suite/rocksdb/r/issue255.result       2018-01-02 14:11:45.000000000 +0300
+++ /oldssd2/tempvar/5/log/issue255.reject      2018-01-23 11:50:13.060075980 +0300
@@ -2,7 +2,7 @@
 INSERT INTO t1 VALUES (5);
 SHOW TABLE STATUS LIKE 't1';
 Name   Engine  Version Row_format      Rows    Avg_row_length  Data_length     Max_data_length Index_length    Data_free       Auto_increment  Create_time     Update_time     Check_time      Collation       Checksum        Create_options  Comment
-t1     ROCKSDB #       Fixed   1       #       #       #       #       #       6       NULL    NULL    NULL    latin1_swedish_ci       NULL
+t1     ROCKSDB #       Fixed   0       #       #       #       #       #       6       NULL    NULL    NULL    latin1_swedish_ci       NULL
```

rows 为估计值，同 1.5。

#### 1.18 rocksdb.max_open_files

相关语句：```SELECT FLOOR(@@global.open_files_limit / 2) = @@global.rocksdb_max_open_files;```

预期结果：
```
FLOOR(@@global.open_files_limit / 2) = @@global.rocksdb_max_open_files
1
```

测试结果：
```
FLOOR(@@global.open_files_limit / 2) = @@global.rocksdb_max_open_files
0
```
为了避免 sst 被多次打开，MySQL on Terarkdb 需要将 rocksdb_max_open_files 设置为 -1，故与预期不一致。

#### 1.19 rocksdb.check_ignore_unknown_options

```
Corruption: Bad table magic number: expected 9863518390377041911, found 1234605616436508552 in ./.rocksdb/000126.sst
```

### 2. rocksdb_sys_vars
#### 2.1 rocksdb_sys_vars.all_vars

相关语句：
```
select variable_name as `There should be *no* variables listed below:` 
       from t2 left join t1 
       on variable_name=test_name 
       where test_name is null 
       ORDER BY variable_name;
```

预期结果：
```
There should be *no* variables listed below:
```

测试结果：
```
There should be *no* variables listed below:
ROCKSDB_SYSTEM_CF_BACKGROUND_FLUSH_INTERVAL
ROCKSDB_SYSTEM_CF_BACKGROUND_FLUSH_INTERVAL
```

因 MySQL on TerarkDB 有更多的背景线程用于 flush `__system__` cloumn famliy 以及时删除 WAL log 文件，故输出结果与预期不一致。

#### 2.2 rocksdb_sys_vars.rocksdb_flush_memtable_on_analyze_basic

相关语句：```SHOW TABLE STATUS LIKE 't1';```

预期结果：
```
Name	Engine	Version	Row_format	Rows	Avg_row_length	Data_length	Max_data_length	Index_length	Data_free	Auto_increment	Create_time	Update_time	Check_time	Collation	Checksum	Create_options	Comment
t1	ROCKSDB	10	Fixed	#	#	24	0	0	0	4	NULL	NULL	NULL	latin1_swedish_ci	NULL
```

测试结果：
```
Name	Engine	Version	Row_format	Rows	Avg_row_length	Data_length	Max_data_length	Index_length	Data_free	Auto_increment	Create_time	Update_time	Check_time	Collation	Checksum	Create_options	Comment
t1	ROCKSDB	10	Fixed	#	#	400	0	0	0	4	NULL	NULL	NULL	latin1_swedish_ci	NULL
```

其中 Rows 为估计值，MySQL on TerarkDB 压缩算法与 MyRocks 不同，故结果不一致，不影响功能。

### 3. rocksdb_hostbackup

#### 3.1 rocksdb_hotbackup.slocket、rocksdb_hotbackup.gtid、rocksdb_hotbackup.xbstream

错误信息：
```
mysqltest: Could not open connection 'default' after 500 attempts: 2002 Can't connect to local MySQL server through socket '/newssd1/temp/mysql-on-terarkdb-4.8-bmi2-0/mysql-test/var/tmp/1/mysqld.2.sock' (2)
```

测试中第二个实例不能启动，导致测试超时，测试程序有误，原版 MyRocks 也不能通过。

### Bloom Filter 相关的测试
TerarkDB 不需要 Bloom Filter，但 RocksDB 原版需要 Bloom Filter，这些测试不影响 TerarkDB 的功能。

1. rocksdb.bloomfilter
2. rocksdb.bloomfilter2
3. rocksdb.bloomfilter3
3. rocksdb.prefix_extractor_override



