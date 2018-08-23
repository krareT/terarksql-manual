## 在 HDD 上测试 TerarkSQL


### 1.背景
虽然作为关键服务的数据库，已经不怎么使用机械硬盘了，但是在某些特殊行业特殊领域，仍有一定的使用。

TerarkSQL 在机械硬盘上的优势会随着 IO 开销的大幅度增加而被一定程度降低

此次测试为测试 TerarkSQL 在机械硬盘上的性能表现。

测试分为三阶段：写、随机读、读写混合（随机读，随机写）

本测试的程序为修改版的 [YCSB](https://github.com/Terark/YCSB) 的 dev 分支。

测试平台如下：

```bash
CPU: Interl(R) Xeon(R) CPU E5-2630 v3 @ 2.4GHz x2 (16 核共 32 线程)
Memory: DDR4 16G @ 1866 MHz x12 (共 192 G)
HDD: ST6000DM001-1XY17Z 7200 rpm
OS: CentOS 7
```

测试使用的数据为 [wikipedia 数据](https://zh.wikipedia.org/wiki/Wikipedia:%E6%95%B0%E6%8D%AE%E5%BA%93%E4%B8%8B%E8%BD%BD)处理后的行文本文件，一共 38508221 条，每条约 2.8K，总大小 102G。

### 2.测试结果总结
测试报告较长，为了节省读者时间，我们把关键信息做了总结（每个线程对应一个客户端连接）：

|项目（默认 16 个线程）                   |  OPS  | 时间<br/>分钟 | 操作次数 | 写速度<br/>KB/s | 读速度<br/>KB/s|
|-----------------------------------------| -----:| -----------:| -------:| -------------:| ------------:|
| 写，关闭 wal<br/>tmpdir 与 dbdir 不同硬盘| 11373 | 56 | 38508221 | 31844 | 0 |
| 写，开启 wal，开启 fsync<br/>tmpdir 与 dbdir 同一硬盘| 89 | 47 | 250000 | 249 | 0 |
| 写，开启 wal，开启 fsync<br/>tmpdir 与 dbdir 不同硬盘| 89 | 46 | 250000 | 249 | 0 |
| 写，开启 wal，关闭 fsync<br/>tmpdir 与 dbdir 同一硬盘| 5407 | 118 | 38508221 | 15140 | 0 |
| 写，开启 wal，开启后台 fsync<br/>tmpdir 与 dbdir 同一硬盘 | 4686 | 137 | 38508221 | 13120 | 0|
| 读（16 线程）| 2435 | 26 | 3850822 | 0 | 6818 |
| 读（8  线程）| 2125 | 30 | 3850822 | 0 | 5950 |
| 读写混合，写占 4.8%<br/>开启 fsync | 433 | 38 | 1000000 读<br/>50339 写 | 59 | 1213 |
| 读写混合，写占 9.0%<br/>开启 fsync | 336 | 49 | 1000000 读<br/>100067 写 | 95 | 941 |
| 读写混合，写占 33%<br/>开启 fsync  | 128 | 34 | 250000 读<br/>124832 写 | 173 | 358 |
| 读写混合，写占 4.8%<br/>关闭 fsync | 1873 | 18 | 2000000 读<br/>99928 写 | 263 | 5244 |
| 读写混合，写占 9.0%<br/>关闭 fsync | 1778 | 14 | 1500000 读<br/>150529 写 | 498 | 4978 |
| 读写混合，写占 33%<br/>关闭 fsync  | 1362 | 18 | 1500000 读<br/>750647 写 | 1909 | 3813 |

其中 读/写磁盘速度 = 读/写 ops * 每条数据大小（约 2.8K）。

参数 ```write ahead log``` 一旦开启，则每次写会引入1次额外的写日志操作，也就意味着用户的每次写将对应2次系统写调用。对比关闭该参数时的1次系统调用，也即第一、四行速度差别的由来。

参数 fsync 开启，则每次写操作都会落盘，此时机械硬盘的性能极大的影响了数据库的写入性能。对比第三、四行，此时性能落差有2个数量级之大。

故在机械硬盘上使用 TerarkSQL 时，建议关闭 fsync， 即将
```rocksdb_flush_log_at_trx_commit``` 设为 2、 ```rocksdb_background_sync``` 设为 ON（具体选项说明见 1.2.1），这样能极大的提升性能（但因为持久化的间隔变为每秒一次，所以在断电等意外情况发生时最近一秒钟的数据可能会丢失）。

在默认开启 fsync 的写测试中，因写入速度太慢，仅插入了一小部分数据，此时并不会引发 compact，这时无论 TerarkDB 临时目录是否和 TerarkSQL 在同一个磁盘，对性能都不会有影响。

在随机读测试中，八个线程时，磁盘 IO 已经基本被占满，磁盘的性能成为瓶颈，这时即使使用 16 个线程性能也没有多大提升。读写混合测试时，若默认开启 fsync，写性能过低而导致整体 ops 不高，读写比例越高则读写混合 ops 越低。关闭 fsync 后，写性能大大提高，读写比例的提高也不会导致读写混合的 ops 迅速下降。

### 3.测试过程

#### 3.1.YCSB 配置文件

```bash
workload=com.yahoo.ycsb.workloads.FileWorkload

recordcount=38508221
operationcount=200000

fieldnames=cur_id,cur_namespace,cur_title,cur_text,cur_comment,cur_user,cur_user_text,cur_timestamp,cur_restrictions,cur_counter,cur_is_redirect,cur_minor_edit,cur_random,cur_touched,inverse_timestamp
usecustomkey=true
keyfield=0,1,2
fieldnum=15

datafile=/data/publicdata/wikipedia/wikipedia.txt.no.tab
keyfile=/data/publicdata/wikipedia/wikipedia_key_shuf.txt.no.tab
writerate=0
threadcount=16

jdbc.driver=com.mysql.jdbc.Driver
db.url=jdbc:mysql://127.0.0.1/ycsb
db.user=root
db.passwd=
mysql.upsert=true
```

TerarkSQL 配置文件如下：
```bash
[mysqld]
rocksdb
default-storage-engine=rocksdb
#rocksdb_validate_tables=2
skip-innodb
default-tmp-storage-engine=MyISAM
collation-server=latin1_bin

user = terark
bind-address = 0.0.0.0
port = 3306

back_log = 600
max_connections = 6000

basedir = /newssd1/temp/terarksql-4.8-bmi2-0
log_error        = /newssd1/temp/terarksql-4.8-bmi2-0/log/mysql_error.log
#general_log_file = /newssd1/temp/terarksql-4.8-bmi2-0/log/mysql.log
#general_log      = 1

datadir = /disk2/terarksql-data
#datadir = /newssd1/temp/terarksql-4.8-bmi2-0/data
#log-bin = /newssd1/temp/terarksql-4.8-bmi2-0/log/mysql-bin.log

binlog-format=ROW
secure_file_priv=""
```

其中 /disk2 即为测试所用的机械硬盘，```general_log``` 和 ```log-bin``` 都被关闭。其他选项均为默认值。

TerarkZipTable 配置如下：

```bash
TerarkZipTable_localTempDir=$PWD/terark-temp
TerarkZipTable_keyPrefixLen=4
TerarkZipTable_softZipWorkingMemLimit=2G
TerarkZipTable_hardZipWorkingMemLimit=4G
TerarkZipTable_offsetArrayBlockUnits=128
TerarkZipTable_indexCacheRatio=0.001
```

以上配置即为此次测试的默认配置，以下测试中未特殊说明的配置都为以上列出配置或者 MyRocks 数据库默认配置。

#### 3.2. 写入测试

所有的写测试都使用 16 个线程

##### 3.2.1. 关闭 write ahead log

为了使数据尽快的导入到磁盘中，首先将 ```write ahead log``` 关闭（```rocksdb-write-disable-wal``` 设为 ON），并将 TerarkDB 的临时目录放在另外的 ssd 盘中，且在导入数据时未限制内存，此时设置 ```TerarkZipTable_softZipWorkingMemLimit``` 为 32G、```TerarkZipTable_hardZipWorkingMemLimit``` 为 64G。

**测试时间为 56 分钟，共执行了 38,508,221 次写操作，平均 ops 为 11,373 。**

具体测试结果如下：
```bash
[OVERALL], RunTime(ms), 3385796.0
[OVERALL], Throughput(ops/sec), 11373.461661600404
[TOTAL_GCS_PS_Scavenge], Count, 365.0
[TOTAL_GC_TIME_PS_Scavenge], Time(ms), 25934.0
[TOTAL_GC_TIME_%_PS_Scavenge], Time(%), 0.7659646357902248
[TOTAL_GCS_PS_MarkSweep], Count, 7.0
[TOTAL_GC_TIME_PS_MarkSweep], Time(ms), 879.0
[TOTAL_GC_TIME_%_PS_MarkSweep], Time(%), 0.02596139873754946
[TOTAL_GCs], Count, 372.0
[TOTAL_GC_TIME], Time(ms), 26813.0
[TOTAL_GC_TIME_%], Time(%), 0.7919260345277743
[CLEANUP], Operations, 16.0
[CLEANUP], AverageLatency(us), 318.1875
[CLEANUP], MinLatency(us), 113.0
[CLEANUP], MaxLatency(us), 2591.0
[CLEANUP], 95thPercentileLatency(us), 472.0
[CLEANUP], 99thPercentileLatency(us), 2591.0
[INSERT], Operations, 3.8508221E7
[INSERT], AverageLatency(us), 1390.6873037837815
[INSERT], MinLatency(us), 107.0
[INSERT], MaxLatency(us), 3702783.0
[INSERT], 95thPercentileLatency(us), 4847.0
[INSERT], 99thPercentileLatency(us), 10295.0
[INSERT], Return=OK, 38508221
```

##### 3.2.2. 开启 write ahread log

wikipedia 数据经过压缩后大大小为 22G。为了测试 TerarkSQL 在机械硬盘上的实际读写性能，这里使用 cgroup 将其能使用内存限制为 16G，此时设置 ```TerarkZipTable_softZipWorkingMemLimit``` 为 2G、```TerarkZipTable_hardZipWorkingMemLimit``` 为 4G。

###### 3.2.2.1. 开启 fsync，TerarkDB 的临时目录和数据库在同一个机械硬盘中

数据库默认设置为开启 fsync，既 ```rocksdb_flush_log_at_trx_commit``` 默认设置为 1， 在每次事务提交都会同步（fsync）文件。可将 ```rocksdb_flush_log_at_trx_commit``` 设置为 2，即使用 ```rocksdb_background_sync``` 的计时器来同步文件。

```rocksdb_background_sync``` 默认设置为 OFF，设置为 ON 时会每秒同步（fsync）一次文件。

```rocksdb_flush_log_at_trx_commit```、```rocksdb_background_sync``` 说明如下：

- rocksdb-background-sync : Enables MyRocks to issue fsyncs for the WAL files every second. If WAL files are sync'ed on every commits, then enabling this option is redundant.
- rocksdb-flush-log-at-trx-commit : Sync'ing on transaction commit similar to innodb-flush-log-at-trx-commit.
  - 0 : never sync
  - 1 : always sync
  - 2 : sync based on a timer controlled via rocksdb-background-sync

具体的配置选项在[这里](https://github.com/facebook/mysql-5.6/wiki/New-MySQL-RocksDB-Server-Variables)。

因为机械硬盘的性能较差，开启 fsync 测试时只写入部分数据，压测时间均在 30 分钟以上。

**测试时间为 47 分钟，共执行了 250,000 次写操作，平均 ops 为 89。**

具体测试结果如下：
```bash
[OVERALL], RunTime(ms), 2811852.0
[OVERALL], Throughput(ops/sec), 88.9093736085683
[TOTAL_GCS_PS_Scavenge], Count, 12.0
[TOTAL_GC_TIME_PS_Scavenge], Time(ms), 1342.0
[TOTAL_GC_TIME_%_PS_Scavenge], Time(%), 0.047726551753079466
[TOTAL_GCS_PS_MarkSweep], Count, 1.0
[TOTAL_GC_TIME_PS_MarkSweep], Time(ms), 129.0
[TOTAL_GC_TIME_%_PS_MarkSweep], Time(%), 0.004587723678202125
[TOTAL_GCs], Count, 13.0
[TOTAL_GC_TIME], Time(ms), 1471.0
[TOTAL_GC_TIME_%], Time(%), 0.05231427543128159
[CLEANUP], Operations, 16.0
[CLEANUP], AverageLatency(us), 337.625
[CLEANUP], MinLatency(us), 114.0
[CLEANUP], MaxLatency(us), 2211.0
[CLEANUP], 95thPercentileLatency(us), 552.0
[CLEANUP], 99thPercentileLatency(us), 2211.0
[INSERT], Operations, 250000.0
[INSERT], AverageLatency(us), 179602.026688
[INSERT], MinLatency(us), 63648.0
[INSERT], MaxLatency(us), 2705407.0
[INSERT], 95thPercentileLatency(us), 300287.0
[INSERT], 99thPercentileLatency(us), 383999.0
[INSERT], Return=OK, 250000
```

测试过程中一段时间磁盘 IO 情况如下(每 30s 记录一次):

```bash
Device:         rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
sdb               0.00    23.47    0.00   35.80     0.00  1161.20    64.87     0.99   27.76    0.00   27.76  27.76  99.37
sdb               0.00    22.20    0.00   35.77     0.00   858.00    47.98     1.00   27.77    0.00   27.77  27.79  99.41
sdb               0.00    24.13    0.00   36.40     0.00  1299.07    71.38     0.99   27.35    0.00   27.35  27.30  99.38
sdb               0.00    23.87    0.00   36.00     0.00  1214.00    67.44     0.99   27.57    0.00   27.57  27.61  99.39
sdb               0.00    23.77    0.00   35.70     0.00  1216.80    68.17     0.99   27.67    0.00   27.67  27.82  99.32
sdb               0.00    23.77    0.00   35.70     0.00  1006.67    56.40     0.99   28.02    0.00   28.02  27.84  99.38
sdb               0.00    23.73    0.00   35.70     0.00   576.93    32.32     1.00   27.85    0.00   27.85  27.89  99.56
sdb               0.00    25.03    0.03   35.47     0.13   997.07    56.18     0.99   28.04   15.00   28.05  27.98  99.35
sdb               0.00    27.23    0.00   59.03     0.00 14754.53   499.87    16.25  275.34    0.00  275.34  16.84  99.40
sdb               0.00    23.80    0.00   36.03     0.00  1151.87    63.93     1.00   27.71    0.00   27.71  27.57  99.36
sdb               0.00    24.00    0.00   36.23     0.00  1218.53    67.26     0.99   27.47    0.00   27.47  27.42  99.37
sdb               0.00    23.70    0.00   35.73     0.00  1189.87    66.60     0.99   27.70    0.00   27.70  27.81  99.36
sdb               0.00    23.53    0.00   35.60     0.00  1234.67    69.36     0.99   28.01    0.00   28.01  27.90  99.34
sdb               0.00    23.70    0.00   35.70     0.00  1234.53    69.16     0.99   27.72    0.00   27.72  27.81  99.29
sdb               0.00    23.77    0.00   35.70     0.00  1225.60    68.66     0.99   27.88    0.00   27.88  27.82  99.32
sdb               0.00    23.63    0.00   35.60     0.00  1192.13    66.97     0.99   27.95    0.00   27.95  27.90  99.31
sdb               0.00    24.03    0.00   36.17     0.00  1187.47    65.67     0.99   27.45    0.00   27.45  27.46  99.32
sdb               0.00    23.67    0.00   35.60     0.00  1155.60    64.92     0.99   27.89    0.00   27.89  27.90  99.34
```

###### 3.2.2.2. 开启 fsync，TerarkDB 临时目录与数据库在不同的机械硬盘中

**测试时间为 46 分钟，共执行了 250,000 次写操作，平均 ops 为 89。**

具体测试结果如下：
```bash
[OVERALL], RunTime(ms), 2808644.0
[OVERALL], Throughput(ops/sec), 89.01092484487177
[TOTAL_GCS_PS_Scavenge], Count, 12.0
[TOTAL_GC_TIME_PS_Scavenge], Time(ms), 1196.0
[TOTAL_GC_TIME_%_PS_Scavenge], Time(%), 0.042582826445786655
[TOTAL_GCS_PS_MarkSweep], Count, 1.0
[TOTAL_GC_TIME_PS_MarkSweep], Time(ms), 126.0
[TOTAL_GC_TIME_%_PS_MarkSweep], Time(%), 0.004486150612181537
[TOTAL_GCs], Count, 13.0
[TOTAL_GC_TIME], Time(ms), 1322.0
[TOTAL_GC_TIME_%], Time(%), 0.047068977057968184
[CLEANUP], Operations, 16.0
[CLEANUP], AverageLatency(us), 542.0
[CLEANUP], MinLatency(us), 78.0
[CLEANUP], MaxLatency(us), 3717.0
[CLEANUP], 95thPercentileLatency(us), 1228.0
[CLEANUP], 99thPercentileLatency(us), 3717.0
[INSERT], Operations, 250000.0
[INSERT], AverageLatency(us), 179378.202944
[INSERT], MinLatency(us), 61280.0
[INSERT], MaxLatency(us), 2482175.0
[INSERT], 95thPercentileLatency(us), 300287.0
[INSERT], 99thPercentileLatency(us), 383487.0
[INSERT], Return=OK, 250000
```

测试过程中一段时间磁盘 IO 情况如下(每 30s 记录一次):

```bash
Device:         rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
sdb               0.00    20.67    0.00   54.60     0.00 12721.60   465.99    17.95  361.07    0.00  361.07  18.22  99.48
sda               0.00     0.00    6.93    0.00   887.47     0.00   256.00     0.01    1.16    1.16    0.00   0.92   0.64
sdb               0.00    23.80    0.00   36.27     0.00  1216.80    67.10     0.99   27.43    0.00   27.43  27.41  99.40
sda               0.00     0.00    6.63    0.00   849.07     0.00   256.00     0.00    0.60    0.60    0.00   0.60   0.40
sdb               0.00    23.93    0.00   35.90     0.00  1168.80    65.11     0.99   27.71    0.00   27.71  27.67  99.33
sda               0.00     0.00    6.53    0.00   836.27     0.00   256.00     0.00    0.52    0.52    0.00   0.52   0.34
sdb               0.00    23.63    0.00   35.57     0.00  1235.87    69.50     0.99   27.90    0.00   27.90  27.92  99.30
sda               0.00     0.00    6.83    0.00   874.67     0.00   256.00     0.00    0.42    0.42    0.00   0.42   0.29
sdb               0.00    23.63    0.00   35.83     0.00  1230.93    68.70     0.99   27.76    0.00   27.76  27.73  99.38
sda               0.00     0.00    6.33    0.00   810.67     0.00   256.00     0.00    0.54    0.54    0.00   0.54   0.34
sdb               0.00    23.90    0.00   35.90     0.00  1232.27    68.65     0.99   27.65    0.00   27.65  27.68  99.37
sda               0.00     0.00    7.13    0.00   913.07     0.00   256.00     0.01    0.71    0.71    0.00   0.70   0.50
sdb               0.00    24.13    0.00   36.33     0.00  1236.53    68.07     0.99   27.37    0.00   27.37  27.35  99.37
sda               0.00     0.00    6.63    0.00   849.07     0.00   256.00     0.00    0.69    0.69    0.00   0.69   0.46
sdb               0.00    24.00    0.00   36.13     0.00  1200.80    66.46     0.99   27.49    0.00   27.49  27.51  99.41
sda               0.00     0.00    7.07    0.00   904.53     0.00   256.00     0.01    0.81    0.81    0.00   0.81   0.57
sdb               0.00    23.60    0.00   35.53     0.00  1123.87    63.26     0.99   27.97    0.00   27.97  27.97  99.38
sda               0.00     0.00    7.37    0.00   942.93     0.00   256.00     0.00    0.66    0.66    0.00   0.61   0.45
sdb               0.00    23.93    0.00   35.90     0.00  1127.33    62.80     0.99   27.71    0.00   27.71  27.67  99.35
```

其中 sdb 为数据库目录所在机械硬盘，sda 为临时目录所在机械硬盘。因未触发 compact，sda 基本没有被使用。

###### 3.2.2.3. 关闭 fsync

通过将 rocksdb_flush_log_at_trx_commit 设为 2，可关闭在每次事务提交后进行同步。

**测试时间为 119 分钟，共执行了 38,508,221 次写操作，平均 ops 为 5407**

具体测试结果如下：

```bash
[OVERALL], RunTime(ms), 7121722.0
[OVERALL], Throughput(ops/sec), 5407.150265062298
[TOTAL_GCS_PS_Scavenge], Count, 2618.0
[TOTAL_GC_TIME_PS_Scavenge], Time(ms), 45253.0
[TOTAL_GC_TIME_%_PS_Scavenge], Time(%), 0.6354221633475724
[TOTAL_GCS_PS_MarkSweep], Count, 139.0
[TOTAL_GC_TIME_PS_MarkSweep], Time(ms), 4891.0
[TOTAL_GC_TIME_%_PS_MarkSweep], Time(%), 0.06867721037131189
[TOTAL_GCs], Count, 2757.0
[TOTAL_GC_TIME], Time(ms), 50144.0
[TOTAL_GC_TIME_%], Time(%), 0.7040993737188843
[CLEANUP], Operations, 16.0
[CLEANUP], AverageLatency(us), 389.875
[CLEANUP], MinLatency(us), 138.0
[CLEANUP], MaxLatency(us), 2101.0
[CLEANUP], 95thPercentileLatency(us), 678.0
[CLEANUP], 99thPercentileLatency(us), 2101.0
[INSERT], Operations, 3.8508221E7
[INSERT], AverageLatency(us), 2930.6821185533345
[INSERT], MinLatency(us), 113.0
[INSERT], MaxLatency(us), 3471359.0
[INSERT], 95thPercentileLatency(us), 11063.0
[INSERT], 99thPercentileLatency(us), 21903.0
[INSERT], Return=OK, 38508221
```

测试过程中一段时间磁盘 IO 情况如下(每 30s 记录一次):

```bash
sdb               0.00     1.47  280.57   48.30  1122.40 24067.73   153.19    36.74  111.86    7.04  720.74   3.04 100.00
sdb               0.00     2.13  953.73   24.00  3821.73 11684.67    31.72    14.39   14.72    1.94  522.66   1.02  99.99
sdb               0.00     1.80 1139.87   74.37  4559.73 37356.00    69.04    60.93   50.18    1.59  794.90   0.82 100.00
sdb               0.00     1.73 1381.20   31.13  5524.80 15420.80    29.66    17.85   12.64    1.27  516.87   0.71 100.00
sdb               0.00     2.90  985.77   87.40  3961.07 43914.80    89.22    62.72   58.45    1.90  696.23   0.93 100.00
sdb               0.00     2.30 1517.73   22.40  6072.27 10916.27    22.06    12.68    7.63    1.16  446.29   0.65 100.00
sdb               0.00     1.67  889.40   64.37  3565.73 32585.33    75.81    49.41   52.78    2.02  754.12   1.05 100.00
sdb               0.00     2.33  460.67   47.57  1847.20 23750.27   100.73    34.15   61.79    3.88  622.62   1.97  99.99
sdb               0.00     3.00  245.37  106.20   986.40 53655.60   310.85   104.50  305.05    7.77  991.89   2.84 100.00
sdb               0.57     3.73  745.70  105.30 10088.67 53253.20   148.86    62.33   73.24    2.44  574.62   1.17  99.83
sdb               0.00     2.40 1111.90   64.33  4449.33 32265.60    62.43    47.51   40.39    1.61  710.64   0.85 100.00
sdb               0.00     2.37 1597.17   34.90  7529.87 17304.67    30.43    20.81   12.75    1.13  544.48   0.61 100.00
sdb               0.00     2.33 1143.03   53.83  4576.00 26820.27    52.46    38.05   31.67    1.57  670.84   0.84  99.99
sdb               0.00     2.80 1257.73   72.23  6129.33 36083.87    63.48    48.58   36.63    1.51  648.18   0.75 100.00
sdb               0.00     2.73  972.00   67.87  4663.33 33957.73    74.28    54.29   52.21    1.87  773.27   0.96 100.00
sdb               0.00     2.47 1836.30   38.20  7619.07 18913.07    28.31    23.71   12.65    0.97  574.28   0.53 100.00
sdb               0.00     1.57  998.43   54.07  5170.27 27257.47    61.62    45.05   40.51    1.81  755.16   0.95 100.00
sdb               0.00     1.67 1476.43   47.03  7007.47 23528.53    40.09    28.62   20.37    1.25  620.58   0.66 100.00
sdb               0.00     2.10 1372.27   51.30  5517.87 25632.13    43.76    42.33   28.44    1.27  755.20   0.70 100.00
sdb               0.00     1.93 1489.60   43.40  7094.40 21705.33    37.57    30.97   21.41    1.24  713.72   0.65 100.00
```

###### 3.2.2.4. 开启后台 fsync

将 ```rocksdb_flush_log_at_trx_commit``` 设为 2，并将 ```rocksdb_background_sync``` 设为 ON，可设置启用后台 compact，每秒进行一次 fsync。

**测试时间为 137 分钟，共执行了 38,508,221 次写操作，平均 ops 为 4684**

具体测试结果如下：

```bash
[OVERALL], RunTime(ms), 8220987.0
[OVERALL], Throughput(ops/sec), 4684.135980266116
[TOTAL_GCS_PS_Scavenge], Count, 2503.0
[TOTAL_GC_TIME_PS_Scavenge], Time(ms), 46865.0
[TOTAL_GC_TIME_%_PS_Scavenge], Time(%), 0.5700653704962677
[TOTAL_GCS_PS_MarkSweep], Count, 112.0
[TOTAL_GC_TIME_PS_MarkSweep], Time(ms), 4198.0
[TOTAL_GC_TIME_%_PS_MarkSweep], Time(%), 0.051064428151996834
[TOTAL_GCs], Count, 2615.0
[TOTAL_GC_TIME], Time(ms), 51063.0
[TOTAL_GC_TIME_%], Time(%), 0.6211297986482645
[CLEANUP], Operations, 16.0
[CLEANUP], AverageLatency(us), 334.25
[CLEANUP], MinLatency(us), 95.0
[CLEANUP], MaxLatency(us), 2415.0
[CLEANUP], 95thPercentileLatency(us), 883.0
[CLEANUP], 99thPercentileLatency(us), 2415.0
[INSERT], Operations, 3.8508221E7
[INSERT], AverageLatency(us), 3398.3445242510684
[INSERT], MinLatency(us), 116.0
[INSERT], MaxLatency(us), 5853183.0
[INSERT], 95thPercentileLatency(us), 14175.0
[INSERT], 99thPercentileLatency(us), 30655.0
[INSERT], Return=OK, 38508221
```

测试过程中一段时间磁盘 IO 情况如下(每 30s 记录一次):

```bash
sdb               0.00     8.43 3626.80   62.90 16184.40 28903.73    24.44    20.20    5.47    1.02  262.16   0.27 100.00
sdb               0.00     5.67 1693.07   70.77  7322.40 34563.47    47.49    52.83   29.91    1.94  698.96   0.57 100.00
sdb               0.00    10.47 3754.73   63.60 15398.00 29305.87    23.42    18.68    4.69    0.96  224.61   0.26 100.00
sdb               0.00     5.73 1742.80   75.87  7782.00 37002.00    49.25    39.02   20.77    1.94  453.18   0.55 100.00
sdb               0.00     7.70 2917.83   62.83 14647.20 29838.00    29.85    21.53    7.93    1.51  306.30   0.34 100.00
sdb               5.63     5.23 2133.00   69.93 45279.33 34177.20    72.14    49.84   21.47    2.40  603.08   0.45  99.74
sdb               7.47     7.70 1803.57   77.97 50529.07 37014.67    93.06    37.55   21.21    2.81  446.75   0.53 100.00
sdb               2.97     9.67  466.67  157.77 11068.40 77459.87   283.55   145.18  232.56    7.89  897.12   1.60 100.00
sdb               7.60     6.67 2018.37   51.43 59671.87 23955.73    80.81    23.66   10.73    2.67  327.20   0.48 100.00
sdb               5.77     9.13  853.53  135.53 25705.20 66967.60   187.39   103.57  106.17    5.74  738.66   1.01 100.00
sdb               3.03     6.53 1866.47  101.33 32811.87 50506.27    84.68    77.26   37.11    1.73  688.65   0.51 100.00
sdb               3.30    11.17 2306.07   68.60 38998.67 32181.60    59.95    25.69   11.64    1.53  351.51   0.42 100.00
sdb               4.67     5.83 2412.67   65.67 46740.13 31327.47    63.00    24.84   10.95    1.24  367.93   0.40  99.82
sdb               6.00     9.07 1336.93   91.30 52858.40 44385.20   136.17    47.46   33.07    1.70  492.42   0.70 100.00
sdb               4.00    11.33  489.47  154.07 17336.67 75814.40   289.50   111.40  173.45    5.81  706.04   1.55  99.79
sdb               3.77    10.50  545.10  145.20 27644.80 72003.07   288.71    88.60  125.39    5.17  576.71   1.45  99.75
```

可见开启关闭 fsync 和使用后台 fsync 性能差别不大，故在后续的读写混合测试中，仅测试关闭 fsync 的情况。

#### 3.3. 随机读测试

随机读测试分别使用了 16 个和 8 个线程进行测试。

##### 3.3.1. 16 线程

**测试时间为 26 分钟，共进行了 3,850,822 次随机读操作，平均 ops 为 2,435。**

具体测试结果如下：

```bash
[OVERALL], RunTime(ms), 1581169.0
[OVERALL], Throughput(ops/sec), 2435.427206073481
[TOTAL_GCS_PS_Scavenge], Count, 202.0
[TOTAL_GC_TIME_PS_Scavenge], Time(ms), 5850.0
[TOTAL_GC_TIME_%_PS_Scavenge], Time(%), 0.3699794266141064
[TOTAL_GCS_PS_MarkSweep], Count, 0.0
[TOTAL_GC_TIME_PS_MarkSweep], Time(ms), 0.0
[TOTAL_GC_TIME_%_PS_MarkSweep], Time(%), 0.0
[TOTAL_GCs], Count, 202.0
[TOTAL_GC_TIME], Time(ms), 5850.0
[TOTAL_GC_TIME_%], Time(%), 0.3699794266141064
[READ], Operations, 3850822.0
[READ], AverageLatency(us), 6536.370804986572
[READ], MinLatency(us), 114.0
[READ], MaxLatency(us), 6488063.0
[READ], 95thPercentileLatency(us), 41055.0
[READ], 99thPercentileLatency(us), 115199.0
[READ], Return=OK, 3850822
[CLEANUP], Operations, 16.0
[CLEANUP], AverageLatency(us), 361.4375
[CLEANUP], MinLatency(us), 117.0
[CLEANUP], MaxLatency(us), 2371.0
[CLEANUP], 95thPercentileLatency(us), 582.0
[CLEANUP], 99thPercentileLatency(us), 2371.0
```

测试过程中一段时间磁盘 IO 情况如下(每 30s 记录一次):

```bash
Device:         rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
sdb               0.00     0.00  528.00    0.00  2112.00     0.00     8.00    15.15   29.82   29.82    0.00   1.89 100.00
sdb               0.00     0.00  527.00    0.00  2108.00     0.00     8.00    15.35   28.52   28.52    0.00   1.90 100.00
sdb               0.00     0.00  475.00    0.00  1900.00     0.00     8.00    15.40   31.31   31.31    0.00   2.11 100.00
sdb               0.00     0.00  518.00    0.00  2072.00     0.00     8.00    15.31   30.72   30.72    0.00   1.93 100.00
sdb               0.00     0.00  538.00    0.00  2152.00     0.00     8.00    15.35   28.77   28.77    0.00   1.86 100.00
sdb               0.00     0.00  494.00    0.00  1976.00     0.00     8.00    15.27   30.57   30.57    0.00   2.02 100.00
sdb               0.00     0.00  552.00    0.00  2208.00     0.00     8.00    15.31   27.78   27.78    0.00   1.81 100.00
sdb               0.00     0.00  532.00    0.00  2128.00     0.00     8.00    15.31   29.05   29.05    0.00   1.88 100.00
sdb               0.00     0.00  515.00    0.00  2060.00     0.00     8.00    15.36   30.16   30.16    0.00   1.94 100.00
sdb               0.00     0.00  523.00    0.00  2092.00     0.00     8.00    15.35   29.00   29.00    0.00   1.91 100.00
sdb               0.00     0.00  547.00    0.00  2188.00     0.00     8.00    15.05   27.44   27.44    0.00   1.83 100.00
sdb               0.00     0.00  511.00    0.00  2044.00     0.00     8.00    15.34   29.82   29.82    0.00   1.96 100.00
sdb               0.00     0.00  547.00    0.00  2188.00     0.00     8.00    15.30   28.22   28.22    0.00   1.83 100.00
sdb               0.00     0.00  580.00    0.00  2320.00     0.00     8.00    15.29   26.08   26.08    0.00   1.72 100.00
sdb               0.00     0.00  518.00    0.00  2072.00     0.00     8.00    15.26   29.21   29.21    0.00   1.93 100.00
sdb               0.00     0.00  536.00    0.00  2144.00     0.00     8.00    15.36   29.07   29.07    0.00   1.87 100.00
```

##### 3.3.2. 8 线程

**测试时间为 30 分钟，共进行了 3,850,822 次随机读操作，平均 ops 为 2,125。**

具体测试结果如下：

```bash
[OVERALL], RunTime(ms), 1812125.0
[OVERALL], Throughput(ops/sec), 2125.0311098848038
[TOTAL_GCS_PS_Scavenge], Count, 204.0
[TOTAL_GC_TIME_PS_Scavenge], Time(ms), 5437.0
[TOTAL_GC_TIME_%_PS_Scavenge], Time(%), 0.3000344898944609
[TOTAL_GCS_PS_MarkSweep], Count, 0.0
[TOTAL_GC_TIME_PS_MarkSweep], Time(ms), 0.0
[TOTAL_GC_TIME_%_PS_MarkSweep], Time(%), 0.0
[TOTAL_GCs], Count, 204.0
[TOTAL_GC_TIME], Time(ms), 5437.0
[TOTAL_GC_TIME_%], Time(%), 0.3000344898944609
[READ], Operations, 3850822.0
[READ], AverageLatency(us), 3749.5292638818414
[READ], MinLatency(us), 110.0
[READ], MaxLatency(us), 1473535.0
[READ], 95thPercentileLatency(us), 23519.0
[READ], 99thPercentileLatency(us), 62495.0
[READ], Return=OK, 3850822
[CLEANUP], Operations, 8.0
[CLEANUP], AverageLatency(us), 459.625
[CLEANUP], MinLatency(us), 107.0
[CLEANUP], MaxLatency(us), 2183.0
[CLEANUP], 95thPercentileLatency(us), 2183.0
[CLEANUP], 99thPercentileLatency(us), 2183.0
```

测试过程中一段时间磁盘 IO 情况如下(每 30s 记录一次):

```bash
Device:         rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
sdb               0.00     0.00  454.00    0.00  1816.00     0.00     8.00     7.19   15.91   15.91    0.00   2.20 100.00
sdb               0.00     0.00  422.00    0.00  1688.00     0.00     8.00     7.31   17.11   17.11    0.00   2.37 100.00
sdb               0.00     0.00  439.00    0.00  1756.00     0.00     8.00     7.31   16.59   16.59    0.00   2.28 100.00
sdb               0.00     0.00  422.00    0.00  1688.00     0.00     8.00     7.25   16.86   16.86    0.00   2.37 100.00
sdb               0.00     0.00  428.00    0.00  1712.00     0.00     8.00     7.35   17.36   17.36    0.00   2.34 100.00
sdb               0.00     0.00  435.00    0.00  1740.00     0.00     8.00     7.17   16.66   16.66    0.00   2.29  99.80
sdb               0.00     0.00  387.00    0.00  1548.00     0.00     8.00     7.38   18.82   18.82    0.00   2.58 100.00
sdb               0.00     0.00  407.00    0.00  1628.00     0.00     8.00     7.25   18.02   18.02    0.00   2.46 100.00
sdb               0.00     0.00  435.00    0.00  1740.00     0.00     8.00     7.31   17.06   17.06    0.00   2.30 100.00
sdb               0.00     0.00  451.00    0.00  1812.00     0.00     8.04     7.29   16.48   16.48    0.00   2.22 100.10
sdb               0.00     0.00  410.00    0.00  1640.00     0.00     8.00     7.20   17.18   17.18    0.00   2.42  99.30
sdb               0.00     0.00  433.00    0.00  1748.00     0.00     8.07     7.29   16.48   16.48    0.00   2.31 100.10
sdb               0.00     0.00  458.00    0.00  1832.00     0.00     8.00     7.36   16.55   16.55    0.00   2.18 100.00
sdb               0.00     0.00  400.00    0.00  1600.00     0.00     8.00     7.36   18.32   18.32    0.00   2.50 100.00
sdb               0.00     0.00  424.00    0.00  1696.00     0.00     8.00     7.22   17.13   17.13    0.00   2.36 100.00
sdb               0.00     0.00  450.00    0.00  1800.00     0.00     8.00     7.15   15.86   15.86    0.00   2.21  99.50
sdb               0.00     0.00  434.00    0.00  1768.00     0.00     8.15     7.36   16.97   16.97    0.00   2.30 100.00
```

#### 3.4. 读写混合测试

所有的读写混合测试都使用 16 个线程

##### 3.4.1. 开启 fsync

###### 3.4.1.1. 读写比例 0.05（写占比 4.8%）

**测试时间为 38 分钟，共执行了 1,000,000 次随机读操作、50,339 次插入操作，平均 ops 为 433。**

具体测试结果如下：

```bash
[OVERALL], RunTime(ms), 2308003.0
[OVERALL], Throughput(ops/sec), 433.27500007582313
[TOTAL_GCS_PS_Scavenge], Count, 192.0
[TOTAL_GC_TIME_PS_Scavenge], Time(ms), 3114.0
[TOTAL_GC_TIME_%_PS_Scavenge], Time(%), 0.13492183502361133
[TOTAL_GCS_PS_MarkSweep], Count, 0.0
[TOTAL_GC_TIME_PS_MarkSweep], Time(ms), 0.0
[TOTAL_GC_TIME_%_PS_MarkSweep], Time(%), 0.0
[TOTAL_GCs], Count, 192.0
[TOTAL_GC_TIME], Time(ms), 3114.0
[TOTAL_GC_TIME_%], Time(%), 0.13492183502361133
[READ], Operations, 1000000.0
[READ], AverageLatency(us), 21584.214466
[READ], MinLatency(us), 152.0
[READ], MaxLatency(us), 3686399.0
[READ], 95thPercentileLatency(us), 130623.0
[READ], 99thPercentileLatency(us), 218751.0
[READ], Return=OK, 1000000
[CLEANUP], Operations, 16.0
[CLEANUP], AverageLatency(us), 389.9375
[CLEANUP], MinLatency(us), 121.0
[CLEANUP], MaxLatency(us), 2145.0
[CLEANUP], 95thPercentileLatency(us), 1131.0
[CLEANUP], 99thPercentileLatency(us), 2145.0
[INSERT], Operations, 50339.0
[INSERT], AverageLatency(us), 298817.5065853513
[INSERT], MinLatency(us), 72064.0
[INSERT], MaxLatency(us), 959999.0
[INSERT], 95thPercentileLatency(us), 459263.0
[INSERT], 99thPercentileLatency(us), 525823.0
[INSERT], Return=OK, 50339
```

测试过程中一段时间磁盘 IO 情况如下(每 30s 记录一次):

```bash
Device:         rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
sdb               0.00     5.00   73.00    9.00   292.00    72.00     8.88     9.50  116.51  117.47  108.78  12.20 100.00
sdb               0.00     6.00   57.00    9.00   248.00   116.00    11.03     9.02  109.64  112.72   90.11  15.15 100.00
sdb               0.00    10.00   96.00   15.00   384.00   160.00     9.80     9.83  103.45  107.12   79.93   9.01 100.00
sdb               0.00     6.00   82.00    9.00   400.00    76.00    10.46    10.90  114.66  115.89  103.44  10.99 100.00
sdb               0.00     5.00   68.00    9.00   272.00    84.00     9.25    10.19  124.23  127.60   98.78  12.99 100.00
sdb               0.00     8.00   83.00   12.00   428.00   116.00    11.45     9.66  103.43  105.93   86.17  10.53 100.00
sdb               0.00     8.00   95.00   12.00   380.00   116.00     9.27    11.75  108.72  111.57   86.17   9.35 100.00
sdb               0.00     7.00  102.00   12.00   408.00   108.00     9.05    11.15  105.35  107.40   87.92   8.77 100.00
sdb               0.00     6.00   71.00    9.00   304.00    80.00     9.60    10.90  128.74  131.52  106.78  12.50 100.00
sdb               0.00     5.00   91.00    9.00   364.00    76.00     8.80    12.37  124.34  127.74   90.00  10.00 100.00
sdb               0.00     6.00   41.00    9.00   168.00    88.00    10.24     7.73  128.02  130.95  114.67  20.00 100.00
sdb               0.00     8.00   74.00   12.00   332.00   176.00    11.81     8.73  112.15  115.36   92.33  11.63 100.00
sdb               0.00     7.00   97.00   12.00   388.00   116.00     9.25    11.38  104.11  106.52   84.67   9.17 100.00
sdb               0.00     6.00  100.00   12.00   444.00   100.00     9.71    12.12  114.17  117.43   87.00   8.93 100.00
sdb               0.00     5.00   55.00    9.00   220.00    76.00     9.25     9.24  127.55  132.18   99.22  15.62 100.00
sdb               0.00     8.00   85.00   12.00   340.00   116.00     9.40     9.92  107.12  110.36   84.17  10.31 100.00
sdb               0.00     8.00   95.00   12.00   380.00   108.00     9.12    11.06  103.66  105.58   88.50   9.35 100.00
```

###### 3.4.1.2. 读写比例 0.1（写占比 9.0%）

**测试时间为 49 分钟，共进行了 1,000,000 次随机读操作、100,067 次写操作，平均 ops 为 336。**

具体测试结果如下：

```bash
[OVERALL], RunTime(ms), 2974802.0
[OVERALL], Throughput(ops/sec), 336.156826571987
[TOTAL_GCS_PS_Scavenge], Count, 205.0
[TOTAL_GC_TIME_PS_Scavenge], Time(ms), 3417.0
[TOTAL_GC_TIME_%_PS_Scavenge], Time(%), 0.11486478763964796
[TOTAL_GCS_PS_MarkSweep], Count, 0.0
[TOTAL_GC_TIME_PS_MarkSweep], Time(ms), 0.0
[TOTAL_GC_TIME_%_PS_MarkSweep], Time(%), 0.0
[TOTAL_GCs], Count, 205.0
[TOTAL_GC_TIME], Time(ms), 3417.0
[TOTAL_GC_TIME_%], Time(%), 0.11486478763964796
[READ], Operations, 1000000.0
[READ], AverageLatency(us), 18938.891436
[READ], MinLatency(us), 144.0
[READ], MaxLatency(us), 2904063.0
[READ], 95thPercentileLatency(us), 120959.0
[READ], 99thPercentileLatency(us), 202751.0
[READ], Return=OK, 1000000
[CLEANUP], Operations, 16.0
[CLEANUP], AverageLatency(us), 316.875
[CLEANUP], MinLatency(us), 107.0
[CLEANUP], MaxLatency(us), 2247.0
[CLEANUP], 95thPercentileLatency(us), 343.0
[CLEANUP], 99thPercentileLatency(us), 2247.0
[INSERT], Operations, 100067.0
[INSERT], AverageLatency(us), 282428.5252081106
[INSERT], MinLatency(us), 69888.0
[INSERT], MaxLatency(us), 732671.0
[INSERT], 95thPercentileLatency(us), 428031.0
[INSERT], 99thPercentileLatency(us), 475391.0
[INSERT], Return=OK, 100067
```

测试过程中一段时间磁盘 IO 情况如下(每 30s 记录一次):

```bash
Device:         rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
sdb               0.00    11.77   92.37   18.20   369.47   223.47    10.73     6.98   63.13   64.82   54.55   9.04 100.00
sdb               0.00    11.57   90.23   17.90   360.93   208.13    10.53     6.91   63.94   65.54   55.83   9.25  99.99
sdb               0.00    11.70   92.90   18.23   371.60   258.80    11.34     7.05   63.47   65.18   54.73   9.00 100.00
sdb               0.00    11.60   91.23   18.13   364.93   219.33    10.68     7.01   64.12   65.92   55.03   9.14  99.99
sdb               0.00    11.83   91.70   18.07   366.80   219.47    10.68     7.15   65.13   67.10   55.12   9.11  99.96
sdb               0.00    11.47   94.17   17.83   376.67   219.07    10.64     7.29   64.93   66.68   55.70   8.93 100.00
sdb               0.00    11.47   90.77   17.87   363.07   240.00    11.10     7.09   65.42   67.24   56.18   9.20  99.99
sdb               0.00    11.80   92.20   18.23   368.80   207.87    10.44     7.11   64.34   66.25   54.67   9.06 100.00
sdb               0.00    11.50   90.17   18.03   360.67   222.53    10.78     7.00   64.46   66.30   55.25   9.24 100.00
sdb               0.00    11.47   85.87   17.83   343.47   211.20    10.70     6.78   65.35   67.34   55.80   9.64  99.98
sdb               0.00    11.77   88.83   18.17   355.33   228.13    10.91     6.86   64.37   66.24   55.20   9.35 100.00
sdb               0.00    11.67   94.60   18.03   378.40   203.87    10.34     7.22   63.96   65.64   55.15   8.88 100.00
sdb               0.00    11.70   88.40   18.13   353.60   210.00    10.58     6.83   64.24   66.11   55.11   9.38  99.98
sdb               0.00    11.77   90.53   18.20   362.13   223.60    10.77     6.95   63.82   65.66   54.71   9.20  99.99
sdb               0.00    11.47   92.07   17.70   368.27   200.00    10.35     7.30   66.54   68.50   56.34   9.11 100.00
sdb               0.00    11.77   91.00   18.23   364.00   245.47    11.16     7.12   65.05   67.19   54.37   9.15  99.99
sdb               0.00    11.63   90.80   18.57   363.20   255.87    11.32     6.87   62.99   64.81   54.07   9.14 100.00
sdb               0.00    12.03   74.90   19.23   299.60   223.33    11.11     5.69   60.37   62.57   51.82  10.62  99.98
```

###### 3.4.1.3. 读写比例 0.5（写占比 33%）

**测试时间为 34 分钟，共执行了 250,000 次随机读操作、124,832 次插入操作，平均 ops 为 128。**

具体测试结果如下：

```bash
[OVERALL], RunTime(ms), 2016954.0
[OVERALL], Throughput(ops/sec), 123.94928193701988
[TOTAL_GCS_PS_Scavenge], Count, 26.0
[TOTAL_GC_TIME_PS_Scavenge], Time(ms), 728.0
[TOTAL_GC_TIME_%_PS_Scavenge], Time(%), 0.036094030900060185
[TOTAL_GCS_PS_MarkSweep], Count, 0.0
[TOTAL_GC_TIME_PS_MarkSweep], Time(ms), 0.0
[TOTAL_GC_TIME_%_PS_MarkSweep], Time(%), 0.0
[TOTAL_GCs], Count, 26.0
[TOTAL_GC_TIME], Time(ms), 728.0
[TOTAL_GC_TIME_%], Time(%), 0.036094030900060185
[READ], Operations, 250000.0
[READ], AverageLatency(us), 10596.699152
[READ], MinLatency(us), 186.0
[READ], MaxLatency(us), 4771839.0
[READ], 95thPercentileLatency(us), 67583.0
[READ], 99thPercentileLatency(us), 148351.0
[READ], Return=OK, 250000
[CLEANUP], Operations, 16.0
[CLEANUP], AverageLatency(us), 362.125
[CLEANUP], MinLatency(us), 134.0
[CLEANUP], MaxLatency(us), 2861.0
[CLEANUP], 95thPercentileLatency(us), 514.0
[CLEANUP], 99thPercentileLatency(us), 2861.0
[INSERT], Operations, 124832.0
[INSERT], AverageLatency(us), 235040.94539861573
[INSERT], MinLatency(us), 54144.0
[INSERT], MaxLatency(us), 799743.0
[INSERT], 95thPercentileLatency(us), 375807.0
[INSERT], 99thPercentileLatency(us), 479999.0
[INSERT], Return=OK, 124832
```

测试过程中一段时间磁盘 IO 情况如下(每 30s 记录一次):
```bash
Device:         rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
sdb               0.00    16.83   30.50   25.47   122.00   347.33    16.77     2.29   41.08   42.54   39.32  17.81  99.68
sdb               0.00    16.27   30.27   24.70   121.07   352.67    17.24     2.17   39.43   38.75   40.26  18.14  99.69
sdb               0.00    16.27   32.17   24.47   128.67   354.27    17.05     2.33   41.13   41.43   40.74  17.60  99.70
sdb               0.00    16.57   28.97   25.10   115.87   334.00    16.64     2.17   40.18   40.46   39.86  18.44  99.68
sdb               0.00    16.00   29.73   24.13   118.93   334.40    16.83     2.25   41.68   42.08   41.18  18.51  99.69
sdb               0.00    16.30   33.53   24.70   134.13   344.40    16.44     2.35   40.30   40.25   40.36  17.12  99.69
sdb               0.00    15.57   31.67   23.60   126.67   326.67    16.41     2.27   41.17   40.43   42.18  18.04  99.72
sdb               0.00    16.07   32.27   24.30   129.07   317.47    15.79     2.23   39.31   38.11   40.90  17.63  99.71
sdb               0.00    15.97   33.13   24.20   132.53   344.00    16.62     2.38   41.60   41.88   41.21  17.38  99.67
sdb               0.00    16.53   29.67   24.90   118.67   353.33    17.30     2.21   40.51   40.97   39.96  18.27  99.70
sdb               0.00    16.27   31.77   24.60   127.07   325.87    16.07     2.29   40.48   40.50   40.44  17.69  99.69
sdb               0.00    16.83   32.10   25.63   128.40   378.00    17.54     2.21   38.27   37.81   38.85  17.26  99.65
sdb               0.00    16.07   31.30   24.50   125.20   342.27    16.76     2.25   40.31   40.09   40.59  17.86  99.68
sdb               0.00    15.87   31.10   23.93   124.40   316.27    16.01     2.26   41.16   40.73   41.72  18.11  99.68
sdb               0.00    16.13   32.73   24.50   130.93   352.80    16.90     2.31   40.27   40.04   40.59  17.41  99.66
sdb               0.00    16.10   32.17   24.20   128.67   316.80    15.81     2.28   40.40   39.82   41.19  17.68  99.68
```

##### 3.4.2 关闭 fsync

###### 3.4.2.1 读写比例 0.05（写占比 4.8%）

**测试时间为 18 分钟，共执行了 2,000,000 次随机读操作、99,928 次插入操作，平均 ops 为 1,873。**

具体测试结果如下：
```bash
[OVERALL], RunTime(ms), 1067787.0
[OVERALL], Throughput(ops/sec), 1873.032730310446
[TOTAL_GCS_PS_Scavenge], Count, 485.0
[TOTAL_GC_TIME_PS_Scavenge], Time(ms), 6840.0
[TOTAL_GC_TIME_%_PS_Scavenge], Time(%), 0.6405771937661725
[TOTAL_GCS_PS_MarkSweep], Count, 0.0
[TOTAL_GC_TIME_PS_MarkSweep], Time(ms), 0.0
[TOTAL_GC_TIME_%_PS_MarkSweep], Time(%), 0.0
[TOTAL_GCs], Count, 485.0
[TOTAL_GC_TIME], Time(ms), 6840.0
[TOTAL_GC_TIME_%], Time(%), 0.6405771937661725
[READ], Operations, 2000000.0
[READ], AverageLatency(us), 8468.388276
[READ], MinLatency(us), 129.0
[READ], MaxLatency(us), 2467839.0
[READ], 95thPercentileLatency(us), 51743.0
[READ], 99thPercentileLatency(us), 134655.0
[READ], Return=OK, 2000000
[CLEANUP], Operations, 16.0
[CLEANUP], AverageLatency(us), 293.75
[CLEANUP], MinLatency(us), 116.0
[CLEANUP], MaxLatency(us), 1908.0
[CLEANUP], 95thPercentileLatency(us), 464.0
[CLEANUP], 99thPercentileLatency(us), 1908.0
[INSERT], Operations, 99928.0
[INSERT], AverageLatency(us), 451.8563865983508
[INSERT], MinLatency(us), 146.0
[INSERT], MaxLatency(us), 208895.0
[INSERT], 95thPercentileLatency(us), 816.0
[INSERT], 99thPercentileLatency(us), 2207.0
[INSERT], Return=OK, 99928
```

测试过程中一段时间磁盘 IO 情况如下(每 30s 记录一次):
```bash
sdb               0.00     0.30  504.80    1.00  2019.20   268.27     9.04    15.38   30.48   30.39   79.13   1.98 100.00
sdb               0.00     0.23  500.87    1.07  2003.47   303.73     9.19    15.40   30.65   30.54   85.16   1.99 100.00
sdb               0.00     0.30  496.77    1.00  1987.07   265.87     9.05    15.35   30.86   30.77   79.07   2.01  99.98
sdb               0.00     0.30  499.53    1.03  1998.13   285.60     9.12    15.41   30.79   30.67   88.42   2.00 100.00
sdb               0.00     0.30  497.83    0.93  1991.33   222.40     8.88    15.39   30.84   30.75   80.54   2.00 100.00
sdb               0.00     0.30  498.90    1.03  1995.60   282.80     9.11    15.41   30.83   30.72   82.58   2.00 100.00
sdb               0.00     0.30  493.70    1.00  1974.80   263.73     9.05    15.45   31.23   31.05  117.40   2.02 100.00
sdb               0.00     0.30  499.73    1.03  1998.93   273.20     9.07    15.42   30.69   30.56   95.71   2.00 100.00
sdb               0.00     0.30  502.27    1.00  2009.07   262.67     9.03    15.41   30.72   30.59   94.10   1.99 100.00
sdb               0.00     0.30  495.17    1.03  1980.67   268.40     9.07    15.42   31.05   30.93   90.32   2.02 100.00
sdb               0.00     0.30  495.23    1.03  1980.93   282.80     9.12    15.41   31.09   31.01   69.81   2.02 100.00
sdb               0.00     0.30  500.80    0.97  2003.20   245.07     8.96    15.40   30.69   30.57   91.52   1.99 100.00
sdb               0.00     0.33  502.97    1.17  2011.87   335.20     9.31    15.44   30.62   30.46  100.91   1.98 100.00
sdb               0.00     0.37  495.20    1.00  1980.80   235.07     8.93    15.39   31.03   30.97   64.53   2.02 100.00
sdb               0.00     0.30  503.30    1.17  2014.53   346.67     9.36    15.38   30.46   30.37   71.77   1.98 100.00
sdb               0.00     0.30  502.77    0.97  2011.07   242.67     8.95    15.40   30.61   30.49   93.76   1.99 100.00
sdb               0.00     0.30  499.13    0.97  1996.53   250.80     8.99    15.39   30.73   30.64   79.41   2.00 100.00
sdb               0.00     0.30  499.20    0.97  2000.93   263.87     9.06    15.38   30.77   30.66   86.76   2.00 100.00
sdb               0.00     0.30  493.97    1.03  1975.87   264.31     9.05    15.39   31.08   30.96   85.16   2.02  99.97
sdb               0.00     0.30  503.87    1.00  2015.47   277.16     9.08    15.39   30.51   30.42   75.37   1.98 100.03
sdb               0.00     0.30  494.70    0.97  1978.80   252.53     9.00    15.44   31.15   31.01  100.21   2.02 100.00
```

###### 3.4.2.2 读写比例 0.1 （写占比 9.0%）

**测试时间为 14 分钟，共执行了 1,500,000 次随机读操作、150,529 次插入操作，平均 ops 为 1,778。**

具体测试结果如下：
```bash
[OVERALL], RunTime(ms), 843703.0
[OVERALL], Throughput(ops/sec), 1777.8768121009407
[TOTAL_GCS_PS_Scavenge], Count, 418.0
[TOTAL_GC_TIME_PS_Scavenge], Time(ms), 5176.0
[TOTAL_GC_TIME_%_PS_Scavenge], Time(%), 0.6134860252956312
[TOTAL_GCS_PS_MarkSweep], Count, 0.0
[TOTAL_GC_TIME_PS_MarkSweep], Time(ms), 0.0
[TOTAL_GC_TIME_%_PS_MarkSweep], Time(%), 0.0
[TOTAL_GCs], Count, 418.0
[TOTAL_GC_TIME], Time(ms), 5176.0
[TOTAL_GC_TIME_%], Time(%), 0.6134860252956312
[READ], Operations, 1500000.0
[READ], AverageLatency(us), 8859.850204
[READ], MinLatency(us), 131.0
[READ], MaxLatency(us), 3600383.0
[READ], 95thPercentileLatency(us), 54015.0
[READ], 99thPercentileLatency(us), 136447.0
[READ], Return=OK, 1500000
[CLEANUP], Operations, 16.0
[CLEANUP], AverageLatency(us), 318.375
[CLEANUP], MinLatency(us), 124.0
[CLEANUP], MaxLatency(us), 2235.0
[CLEANUP], 95thPercentileLatency(us), 403.0
[CLEANUP], 99thPercentileLatency(us), 2235.0
[INSERT], Operations, 150529.0
[INSERT], AverageLatency(us), 444.2268200811804
[INSERT], MinLatency(us), 134.0
[INSERT], MaxLatency(us), 67007.0
[INSERT], 95thPercentileLatency(us), 810.0
[INSERT], 99thPercentileLatency(us), 2171.0
[INSERT], Return=OK, 150529
```

测试过程中一段时间磁盘 IO 情况如下(每 30s 记录一次):
```bash
sdb               0.00     0.23  499.77    1.47  1999.07   530.13    10.09    15.48   30.92   30.67  116.59   2.00 100.00
sdb               0.00     0.23  492.77    1.50  1971.07   530.80    10.12    15.56   31.49   31.09  165.07   2.02 100.00
sdb               0.00     0.23  492.70    1.47  1970.80   515.60    10.06    15.49   31.30   31.07  108.75   2.02 100.00
sdb               0.00     0.23  498.97    1.43  1995.87   484.40     9.91    15.51   31.03   30.71  140.81   2.00 100.00
sdb               0.00     0.23  504.60    1.40  2018.40   495.07     9.93    15.45   30.55   30.33  111.57   1.98 100.00
sdb               0.00     0.23  499.07    1.43  1996.27   485.20     9.92    15.43   30.81   30.65   89.26   2.00 100.00
sdb               0.00     0.27  497.23    1.47  1988.93   528.93    10.10    15.47   31.01   30.75  119.89   2.01 100.00
sdb               0.00     0.23  494.80    1.13  1979.20   351.73     9.40    15.47   31.20   30.92  150.38   2.02 100.00
sdb               0.00     0.23  497.53    2.00  1990.13   783.20    11.10    15.52   31.06   30.74  108.93   2.00 100.00
sdb               0.00     0.27  496.03    1.10  1984.13   330.27     9.31    15.45   31.11   30.93  109.00   2.01 100.00
sdb               0.00     0.23  490.20    1.27  1960.80   426.13     9.71    15.46   31.46   31.28  100.76   2.03 100.00
sdb               0.00     0.23  493.40    1.13  1973.60   356.67     9.42    15.47   31.28   31.05  133.35   2.02 100.00
sdb               0.00     0.23  495.20    2.40  1980.80   965.87    11.84    15.66   31.42   30.86  145.81   2.01 100.00
sdb               0.00     0.23  494.47    1.47  1977.87   520.53    10.08    15.57   31.45   31.05  164.91   2.02 100.00
sdb               0.00     0.23  491.80    1.57  1967.20   541.07    10.17    15.58   31.57   31.14  166.85   2.03 100.00
sdb               0.00     0.27  498.03    1.10  1992.13   345.87     9.37    15.41   30.87   30.73   93.27   2.00 100.00
sdb               0.00     0.23  498.83    1.27  1995.33   413.20     9.63    15.42   30.76   30.63   82.74   2.00 100.00
sdb               0.00     0.23  477.23    1.90  1908.93   716.80    10.96    13.25   27.78   27.59   75.95   2.09 100.00
```

###### 3.4.2.3 读写比例 0.5（写占比 33%）

**测试时间为 18 分钟，共执行了 1,500,000 次随机读操作、750,647 次插入操作，平均 ops 为 1,362。**

具体测试结果如下：

```bash
[OVERALL], RunTime(ms), 1101276.0
[OVERALL], Throughput(ops/sec), 1362.056378237608
[TOTAL_GCS_PS_Scavenge], Count, 511.0
[TOTAL_GC_TIME_PS_Scavenge], Time(ms), 5784.0
[TOTAL_GC_TIME_%_PS_Scavenge], Time(%), 0.5252089394484216
[TOTAL_GCS_PS_MarkSweep], Count, 0.0
[TOTAL_GC_TIME_PS_MarkSweep], Time(ms), 0.0
[TOTAL_GC_TIME_%_PS_MarkSweep], Time(%), 0.0
[TOTAL_GCs], Count, 511.0
[TOTAL_GC_TIME], Time(ms), 5784.0
[TOTAL_GC_TIME_%], Time(%), 0.5252089394484216
[READ], Operations, 1500000.0
[READ], AverageLatency(us), 11381.387611333334
[READ], MinLatency(us), 142.0
[READ], MaxLatency(us), 1751039.0
[READ], 95thPercentileLatency(us), 67007.0
[READ], 99thPercentileLatency(us), 150783.0
[READ], Return=OK, 1500000
[CLEANUP], Operations, 16.0
[CLEANUP], AverageLatency(us), 378.5
[CLEANUP], MinLatency(us), 130.0
[CLEANUP], MaxLatency(us), 2361.0
[CLEANUP], 95thPercentileLatency(us), 561.0
[CLEANUP], 99thPercentileLatency(us), 2361.0
[INSERT], Operations, 750647.0
[INSERT], AverageLatency(us), 443.9395747934782
[INSERT], MinLatency(us), 130.0
[INSERT], MaxLatency(us), 498431.0
[INSERT], 95thPercentileLatency(us), 805.0
[INSERT], 99thPercentileLatency(us), 2165.0
[INSERT], Return=OK, 750647
```


测试过程中一段时间磁盘 IO 情况如下(每 30s 记录一次):
```bash
sdb               0.00     0.27  477.27    6.20  1909.07  2891.47    19.86    17.25   35.66   31.94  321.53   2.07 100.00
sdb               0.00     0.30  477.83    4.27  1911.33  1920.40    15.90    16.06   33.34   31.95  188.96   2.07 100.00
sdb               0.00     0.30  482.90    3.50  1931.60  1547.07    14.30    16.29   33.47   31.60  292.15   2.06 100.00
sdb               0.00     0.27  481.50    3.50  1926.00  1542.00    14.30    15.96   32.92   31.79  188.67   2.06 100.00
sdb               0.00     0.27  480.67    3.53  1922.67  1560.80    14.39    16.02   33.02   31.67  216.26   2.07 100.00
sdb               0.00     0.27  480.53    6.07  1922.13  2817.60    19.48    16.84   34.69   31.84  260.69   2.06 100.00
sdb               0.00     0.30  471.63    4.33  1886.53  1968.67    16.20    15.99   33.56   32.22  179.94   2.10 100.00
sdb               0.00     0.30  483.00    3.43  1932.00  1506.13    14.14    15.77   32.42   31.44  170.80   2.06 100.00
sdb               0.00     0.23  488.63    3.20  1954.53  1377.20    13.55    15.91   32.38   31.09  228.16   2.03 100.00
sdb               0.00     0.27  481.20    5.33  1924.80  2455.07    18.00    17.03   34.10   31.59  260.69   2.06 100.00
sdb               0.00     0.30  474.83    4.77  1899.33  2199.60    17.09    16.18   34.64   32.10  288.45   2.09 100.00
sdb               0.00     0.30  471.47    3.57  1885.87  1586.53    14.62    15.88   33.46   32.35  179.60   2.11 100.00
sdb               0.00     0.23  477.50    2.87  1910.00  1230.13    13.07    16.23   33.77   31.94  338.92   2.08 100.00
sdb               0.00     0.23  476.10    4.50  1904.40  2068.67    16.53    16.95   34.49   32.08  289.21   2.08 100.00
sdb               0.00     0.30  481.17    4.67  1924.67  2152.53    16.78    16.44   34.58   31.70  331.23   2.06 100.00
sdb               0.00     1.73  450.60   19.47  1802.53  9481.60    48.01    22.57   46.09   32.67  356.84   2.13 100.00
```
