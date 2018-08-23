
## 主从同步测试

### 测试环境

- CPU: Intel(R) Xeon(R) CPU E5-2630 v3 @ 2.40GHz x2 （共16核32线程）
- 内存: DDR4 1866 MHz（共192G）
- SSD: INTEL 730 IOPS 89000
- CentOS Linux release 7.3.1611 (Core)

### 测试程序及数据

使用的测试程序为经我们修改过的 [YCSB](https://github.com/Terark/YCSB/tree/rpl-test) （ rpl-test 分支），测试中使用的 workload 为 [FileWorkload](https://github.com/Terark/YCSB/blob/master/README-terark.md) 。

测试程序所使用的配置见附录 1 。

表结构及索引如下：

```
CREATE TABLE `lineitem` (
  `YCSB_KEY` 		varchar(30) COLLATE utf8_bin NOT NULL DEFAULT '',
  `L_SUPPKEY` 		int(11) NOT NULL,
  `L_LINENUMBER`	int(11) NOT NULL,
  `L_QUANTITY`		decimal(15,2) NOT NULL,
  `L_EXTENDEDPRICE`	decimal(15,2) NOT NULL,
  `L_DISCOUNT` 		decimal(15,2) NOT NULL,
  `L_TAX`		decimal(15,2) NOT NULL,
  `L_RETURNFLAG` 	char(1) COLLATE utf8_bin NOT NULL,
  `L_LINESTATUS` 	char(1) COLLATE utf8_bin NOT NULL,
  `L_SHIPDATE` 		date NOT NULL,
  `L_COMMITDATE` 	date NOT NULL,
  `L_RECEIPTDATE` 	date NOT NULL,
  `L_SHIPINSTRUCT` 	char(25) COLLATE utf8_bin NOT NULL,
  `L_SHIPMODE` 		char(10) COLLATE utf8_bin NOT NULL,
  `L_COMMENT` 		varchar(1024) COLLATE utf8_bin NOT NULL,
  PRIMARY KEY (`YCSB_KEY`)
) DEFAULT CHARSET=utf8 COLLATE=utf8_bin
```

数据样本见附录 2 ，每条数据平均大小为：0.6 KB 。

测试以插入为主，并有 **5%** 的几率立刻执行 **update** 操作将该条记录的 **L_COMMENT** 更新为随机生成的 36 个字节的 uuid（如：7f451daf-b512-439b-8996-c61df9d1017c），另有 **1%** 的几率执行 **delete** 操作将该条记录删除。


### 数据库配置

- 原生 MySQL : 使用 InnoDB 为存储引擎，下记为 innodb；
- 原生 MyRocks: 使用 RocksDB 为存储引擎，下记为 rocksdb；
- TerarkSQL：使用 TerarkDB 为存储引擎，下记为 terarkdb。

各个主、从数据库配置见附录 3 。

如下为测试结果，所有的测试均进行了一个小时，每隔 5 分钟纪录一次数据，记录的数据有：主库的插入速度（记为 OPS），从库同步情况（即 ```Seconds_Behind_Master```， 记为 SBM）。

### 主 innodb 从 innodb

|     | 5 min |10 min |15 min |20 min |25 min |30 min |35 min |40 min |45 min |50 min |55 min |60 min |  平均  | checksum |
|:---:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:---------:|
| OPS | 968.2 | 989.7 | 951.7 | 975.1 | 969.5 | 982.7 |1000.3 | 996.4 | 949.1 | 988.6 | 993.4 | 978.2 |  975  | 883418646 |
| SBM |   0   |   0   |   0   |   0   |   0   |   0   |   0   |   0   |   0   |   0   |   0   |   0   |  N/A  | 883418646 |

适当增大 ops，可以看到从库出现了延迟

|     | 5 min |10 min |15 min |20 min |25 min |30 min |35 min |40 min |45 min |50 min |55 min |60 min |  平均  | checksum |
|:---:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:---------:|
| OPS | 982.2 | 994.9 |1000.4 |1004.8 |1011.2 | 987.5 |1006.6 | 1002  | 998.1 | 989.5 |1005.7 | 955.4 | 998.9 | 271305144 |
| SBM |   2   |   4   |   6   |   8   |  15   |  22   |  27   |  33   |  40   |  48   |  56   |   61  |  N/A  | 271305144 |

### 主 innodb 从 terarkdb

|     | 5 min |10 min |15 min |20 min |25 min |30 min |35 min |40 min |45 min |50 min |55 min |60 min |  平均  | checksum |
|:---:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:---------:|
| OPS | 967.3 | 971.3 | 954.5 | 967.8 | 971.2 | 937.2 | 942.8 | 948.3 | 984.1 | 943.1 |  977  | 966.9 | 966.4 | 18798117  |
| SBM |   9   |  15   |  24   |  32   |  43   |  51   |  56   |  61   |  64   |  65   |  66   |   69  |  N/A  | 18798117  |

在 terarkdb options 配置中设置将 ```__system__``` 加入到 blackList（```TerarkZipTable_blackListColumnFamily=__system__```） 以减少 ```__system__``` column family 在后台频繁 compact 对从库同步的影响。

|     | 5 min |10 min |15 min |20 min |25 min |30 min |35 min |40 min |45 min |50 min |55 min |60 min |  平均  | checksum |
|:---:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:---------:|
| OPS |  989  | 978.3 | 970.5 | 972.9 | 997.2 | 990.8 | 991.5 | 973.1 | 980.6 | 945.2 | 979.6 | 982.9 | 973.8 |3284333877 |
| SBM |   3   |   4   |  10   |  10   |  18   |  25   |  31   |  39   |  48   |  43   |  50   |  55   |  N/A  |3284333877 |

适当降低 ops，主从便能正常同步。

|     | 5 min |10 min |15 min |20 min |25 min |30 min |35 min |40 min |45 min |50 min |55 min |60 min |  平均  | checksum |
|:---:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:---------:|
| OPS | 867.1 | 888.3 | 878.2 | 891.1 | 879.4 | 879.3 | 875.8 | 891.9 |  897  | 890.6 |  880  | 886.7 | 877.1 |1479834484 |
| SBM |   0   |   0   |   0   |   0   |   0   |   0   |   0   |   0   |   0   |   0   |   0   |   0   |  N/A  |1479834484 |

### 主 innodb 从 rocksdb

|     | 5 min |10 min |15 min |20 min |25 min |30 min |35 min |40 min |45 min |50 min |55 min |60 min |  平均  | checksum |
|:---:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:---------:|
| OPS |1005.2 |  996  |1017.9 | 980.7 | 964.1 |1003.9 |  983  | 996.9 | 990.2 | 965.6 | 996.6 | 939.3 | 986.6 |3842942686 |
| SBM |  11   |  26   |  34   |  45   |  57   |  68   |  81   |  91   |  100  |  114  |  123  |  135  |  N/A  |3842942686 |


### 主 terarkdb 从 innodb

|     | 5 min |10 min |15 min |20 min |25 min |30 min |35 min |40 min |45 min |50 min |55 min |60 min |  平均  | checksum |
|:---:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:---------:|
| OPS | 943.8 | 904.7 |  909  | 896.6 | 954.9 | 954.3 | 948.8 | 889.9 | 951.1 | 923.3 | 906.8 | 931.4 |  925  |1088205640 |
| SBM |   0   |   0   |   0   |   0   |   0   |   0   |   0   |   0   |   0   |   0   |   0   |   0   |  N/A  |1088205640 |

适当增大 ops，可以看到半小时后从库出现延迟

|     | 5 min |10 min |15 min |20 min |25 min |30 min |35 min |40 min |45 min |50 min |55 min |60 min |  平均  | checksum |
|:---:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:---------:|
| OPS | 982.5 | 976.8 | 985.6 | 976.9 | 961.6 | 973.7 | 975.4 | 984.8 | 971.9 | 993.1 | 999.7 | 991.7 | 982.8 |3662635349 |
| SBM |   0   |   0   |   0   |   0   |   0   |   3   |   5   |   4   |   4   |   7   |   9   |   10  |  N/A  |3662635349 |


### 主 terarkdb 从 terarkdb

|     | 5 min |10 min |15 min |20 min |25 min |30 min |35 min |40 min |45 min |50 min |55 min |60 min |  平均  | checksum |
|:---:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:---------:|
| OPS | 931.1 |  904  |  933  | 934.2 | 970.1 | 957.4 |  875  | 899.5 | 930.9 | 906.7 | 917.7 | 904.8 | 925.3 |4273250936 |
| SBM |   1   |   0   |   0   |   0   |   1   |   2   |   0   |   0   |   2   |   0   |   0   |   0   |  N/A  |4273250936 |


### 测试分析

从上述测试可以看出，主 innodb 从 terarkdb 时且 OPS 大于 ～950 后，会出现同步延迟。

我们当前正进行改进以解决延迟问题，改进包括有 使用 RBTree 替换原生的 SkipList，Rocksdb 更新，MyRocks 的更新等。

### 从库追同步

在 ops 约为 870 时，主从刚好能够正常同步，执行操作 10 分钟后，从库停止 1 分钟后再启动，在 13 分钟内可以和主库保持同步。

在 ops 约为 580 时，主从能够正常同步，执行操作 10 分钟后，从库停止 1 分钟后再启动，在 1 分 30 秒内可以和主库保持同步。

### 主从切换

主 innodb，从 terarkdb 执行操作（增删改）10 分钟，切换主从后执行 5 分钟，数据一致。然后切换回主 innodb，从 terarkdb，如此重复 3 次，主、从数据库的数据能保持一致。

主 terarkdb，从 terarkdb 构架下执行流程同上，主、从数据库的数据也能保持一致。

主从切换的操作步骤如下：

1. 停止 YCSB 测试程序。
2. 等待从库同步完毕并对主、从库数据进行 checksum 检查。
3. 从库执行 ```stop slave;``` 停止同步。
4. 从库执行 ```reset master;``` 设置为新的主库。
5. 原主库执行 ```CHANGE MASTER TO MASTER_HOST='127.0.0.1', MASTER_USER='rpl', MASTER_PORT=3337,  MASTER_PASSWORD='123456'; start slave;``` 将其设置为新的从库，指向新的主库并开始同步。
6. 启动 YCSB 测试程序。


## 稳定性

稳定性方面的测试，我们构建了一个测试用程序，使用多个线程随机向 MyTerark 里发送包含建表、删表、改表、插入、删除等操作。持续跑了10个小时，MyTerark 依然可以正常运行。以及，在此过程中，使用```kill -9```并重启，程序表现正常。程序如下：

```
for i = 1 to N:
	start_thread(executor)
	
func executor:
	while (true):
		op = rand()
		switch op:
		case CreateTable: 	CreateTable(); break;
		case DropTable: 	DropTable(); break;
		case AlterTable: 	AlterTable(); break;
		case Insert:		Insert(); break;
		case Delete:		Delete(); break;
		case Query:			Query(); break;

// terark_N is a random selected table between terark_[1, 100]
func CreateTable() 	// create table terark_N, with 9 indexes, from tpcl
func DropTable()		// drop table terark_N
func AlterTable()	// use table terark_N, drop 2 indexes, create 2 indexes
func Insert()			// use table terark_N, insert aound 10k rows from tpch
func Delete()			// use table terark_N, delete 10k rows
```


### 附录

#### 附录 1

测试时所使用的配置如下：

```
workload=com.yahoo.ycsb.workloads.FileWorkload
 
recordcount=40000000
operationcount=1500000
 
fieldnames=L_ORDERKEY,L_PARTKEY,L_SUPPKEY,L_LINENUMBER,L_QUANTITY,L_EXTENDEDPRICE,L_DISCOUNT,L_TAX,L_RETURNFLAG,L_LINESTATUS,L_SHIPDATE,L_COMMITDATE,L_RECEIPTDATE,L_SHIPINSTRUCT,L_SHIPMODE,L_COMMENT
usecustomkey=true
keyfield=0,1
fieldnum=16
 
datafile=/data/lineitem_512b.tbl
table=lineitem 
delimiter=|

threadcount=16
 
jdbc.driver=com.mysql.jdbc.Driver
db.url=jdbc:mysql://127.0.0.1:3336/ycsb
db.user=root
db.passwd=
mysql.upsert=true

maxexecutiontime=3720
```

#### 附录 2

数据样本如下：

```
1|16605260|822776|1|17|19795.31|0.04|0.02|N|O|1996-03-13|1996-02-12|1996-03-22|DELIVER IN PERSON|TRUCK|fluffily ironic pinto beans sleep daringly pending instructions. fluffily permanent foxes mold along the furiously express ideas. ironic pinto beans cajole fluffily unusual theodolites. carefully regular theodolites across the pending pinto beans haggle even, ironic deposits. quickly special packages above the furiously regular deposits wake carefully pending requests. carefully regular courts above the pending, even accounts haggle blithely even |
1|7202072|782073|2|36|35053.56|0.09|0.06|N|O|1996-04-12|1996-02-28|1996-04-20|TAKE BACK RETURN|MAIL|n excuses. fluffily final requests wake quickly against the carefully unusual courts. packages cajole. unusual ideas nag quickly about the slyly pending warhorses! carefully ironic escapades are carefully. requests integrate blithely. slyly regular patterns alongside of the blithely ironic packages sleep carefully blithely final ideas. blithely dogged accounts according to the frays would use fluffily across the blithely final dependencies. carefully blithe instructions according to the blithely bold packages are busily carefully final dependencies: slyly bold requests across the fluffily pending pinto beans wake furiously slyly regular ide|
1|6815877|395878|3|8|14340.24|0.10|0.02|N|O|1996-01-29|1996-03-05|1996-01-31|TAKE BACK RETURN|REG AIR|y final asymptotes are unusual, special warhorses. silent foxes sleep carefully regular requests! ironic orbits haggle furiously. bold instructions sleep ironically fluffily silent dependencies. slyly special accounts solve slyly special grouches. silent dependencies sleep furiously. regular, express deposits wake. carefully final theodolites haggle blithely. quickly even foxes across the fluffily silent hockey players nag slyly pending requests. slyly ironic requests at the bold accounts wake carefully regular packages. deposits snooze blithely special reque|
```

#### 附录 3

innodb（主）配置如下：

```
[mysqld]

character-set-server=utf8
collation-server=utf8_bin

back_log = 600
max_connections = 6000

binlog-format=ROW
secure_file_priv=""

server-id = 1

```

innodb（从）配置如下：

```
[mysqld]

character-set-server=utf8
collation-server=utf8_bin

back_log = 600
max_connections = 6000

binlog-format=ROW
secure_file_priv=""

server-id = 2
log_slave_updates = 1
read_only         = 1
replicate-ignore-db = mysql
```

rocksdb（从）配置如下：

```
[mysqld]

rocksdb
default-storage-engine=rocksdb
skip-innodb
default-tmp-storage-engine=MyISAM
character-set-server=utf8
collation-server=utf8_bin

binlog-format=ROW
secure_file_priv=""
rocksdb_commit_in_the_middle=ON
rocksdb_bulk_load_size=1000
rocksdb_strict_collation_check=OFF

server-id = 2
log_slave_updates = 1
read_only         = 1

replicate-ignore-db = mysql
```

terarkdb（主）配置如下：

```
[mysqld]

rocksdb
default-storage-engine=rocksdb
skip-innodb
default-tmp-storage-engine=MyISAM
character-set-server=utf8
collation-server=utf8_bin

back_log = 600
max_connections = 6000

binlog-format=ROW
secure_file_priv=""
rocksdb_commit_in_the_middle=ON
rocksdb_bulk_load_size=1000
rocksdb_strict_collation_check=OFF

server-id = 1
```

terarkdb（从）配置如下：

```
[mysqld]

rocksdb
default-storage-engine=rocksdb
skip-innodb
default-tmp-storage-engine=MyISAM
character-set-server=utf8
collation-server=utf8_bin

back_log = 600
max_connections = 6000

binlog-format=ROW
secure_file_priv=""
rocksdb_commit_in_the_middle=ON
rocksdb_bulk_load_size=1000
rocksdb_strict_collation_check=OFF

server-id = 2
log_slave_updates = 1
read_only         = 1

replicate-ignore-db = mysql
```

terarkdb options 配置如下：

```
TerarkZipTable_localTempDir=$PWD/terark-temp
TerarkZipTable_keyPrefixLen=4
TerarkZipTable_offsetArrayBlockUnits=128
TerarkZipTable_indexCacheRatio=0.001
TerarkZipTable_extendedConfigFile=$PWD/license
TerarkUseDivSufSort=1
```

## 更新

1. 添加主从切换的操作步骤。 2018.01.05
