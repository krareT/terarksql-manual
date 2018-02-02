

本测试使用的数据是由 tpch 程序产生的 lineitem 数据，共 2M(200万) 条，总字节数 1.2G，平均每条约 600 字节，单条数据格式如下：

```
1|16605260|822776|1|17|19795.31|0.04|0.02|N|O|1996-03-13|1996-02-12|1996-03-22|DELIVER IN PERSON|TRUCK|fluffily ironic pinto beans sleep daringly pending instructions. fluffily permanent foxes mold along the furiously express ideas. ironic pinto beans cajole fluffily unusual theodolites. carefully regular theodolites across the pending pinto beans haggle even, ironic deposits. quickly special packages above the furiously regular deposits wake carefully pending requests. carefully regular courts above the pending, even accounts haggle blithely even |
```

我们将创建 100 张表，将这 1.2G 的数据插入每张表，相当于 200M(2亿)条，120GB 的原始数据。表的结构参见附注。

测试程序使用 [MyRocksTest](https://github.com/Terark/MyRocksTest)。

## 压缩率

|      | 原始数据 | 数据库大小 | 压缩比 |
|:----:|:-------:|:---------:|:------:|
| TerarkDB          | 120 GB | 35 GB  | 3.4 |
| InnoDB（未开压缩）  | 120 GB | 210 GB | 0.6 |
| InnoDB（打开压缩）  | 120 GB | 76 GB  | 1.6 |

## 插入测试

插入测试进行了顺序插入和随机插入两种。顺序插入测试使用 **4** 个线程，**每条 insert 语句 100 个 value**；随机插入测试使用 **32** 个线程，**每条 insert 语句 1 个 value**。因索引数量对 InnoDB 的插入速度影响较大，这里也分别对 **10 个索引**和 **4 个索引**的情况进行了测试。

|                   | 顺序插入（10 索引）| 随机插入（10 索引）| 顺序插入（4 索引）| 随机插入（4 索引）|
|:-----------------:|:----------------:|:-----------------:|:----------------:|:---------------:|
| TerarkDB | 25k | 13k  | 37k | 14.5k |
|  InnoDB  | 11k | 5.4k | 30k | 9.2k  |

## 查询测试

查询测试进行了主键等值查询、次级索引等值查询、次级索引范围查询、混合查询。

等值查询：
  - ```select * from ? where L_ORDERKEY = ? and L_PARTKEY = ?;```
  
次级索引等值查询：
  - ```select * from ? where L_PARTKEY = ? limit 1;```
  - ```select * from ? where L_SUPPKEY = ? limit 1;```
  
次级索引范围查询：
  - ```select * from ? where L_PARTKEY < ? limit 1;```
  - ```select * from ? where L_PARTKEY >= ? limit 1;```
  - ```select * from ? where L_SUPPKEY > ? limit 1;```
  - ```select * from ? where L_SUPPKEY <= ? limit 1;```
  
并分别在 

- 192G（不限制，都能将数据库放入内存）；
- 40G（TerarkDB 能把全部数据放到内存中，InnoDB 不能把全部数据放到内存中）；
- 12G（TerarkDB 和 InnoDB 都不能把全部数据放到内存中，且原始数据和内存比为 10 ：1） 

查询测试均使用 **80** 个线程。

注：MySQL 的 innodb_buffer_pool_size 在不同内存下分别设置为可用内存的 70%

### 不使用 prepared statement

| 内存 | 数据库 | 主键等值查询 |	 次级索引等值查询 |	 次级索引范围查询 |	混合查询 |
|:----:|:-----:|:-----------:|:-----------:|:-----------:|:-------:|
| 192G | TerarkDB | 114k |	93k |	95k |	99k |
| 192G |  InnoDB  | 117k |	55k |	96k |	80k |
| 40G  | TerarkDB | 114k |	93k |	95k |	99k |
| 40G  |  InnoDB  | 42k |	18.6k |	7.9k |	15k |
| 12G  | TerarkDB | 91k |	17.2k |	31k  |	24k |
| 12G  |  InnoDB  | 38k |	11.6k |	3.3k |	6.8k |

### 使用 prepared statement

| 内存 | 数据库 | 主键等值查询 |	 次级索引等值查询 |	 次级索引范围查询 |	混合查询 |
|:----:|:-----:|:-----------:|:-----------:|:-----------:|:-------:|
| 192G | TerarkDB | 118k |	98k |	99k |	104k |
| 192G |  InnoDB  | 128k |	58k |	102k |	85k |
| 40G  | TerarkDB | 118k |	98k |	99k |	104k |
| 40G  |  InnoDB  | 48k |	19.0k	| 8.0k |	15.3k |
| 12G  | TerarkDB | 92k |	15.5k |	26k  |	19.2k |
| 12G  |  InnoDB  | 39k |	11.8k |	3.3k |	6.6k  |

对测试结果解读如下：

- 内存192G时：InnoDB 主键可以全部放入 buffer pool。Terark 的数据在访问时候解压，故部分查询性能比 InnoDB 略低
- 内存40G时：TerarkDB 仍然有4G空闲内存，故与192G相比没有变化
- 内存12G时：此场景内存太小，PreparedStatement 占用的内存不可忽略，挤占缓存，可能导致性能下降

## 附注

10 索引的表结构如下，从 lineitem1 到 lineitem100 共 100 张表，

```
CREATE TABLE lineitem  (
             L_ORDERKEY    	BIGINT NOT NULL,
             L_PARTKEY     	BIGINT NOT NULL,
             L_SUPPKEY     	INTEGER NOT NULL,
             L_LINENUMBER  	INTEGER NOT NULL,
             L_QUANTITY    	DECIMAL(15,2) NOT NULL,
             L_EXTENDEDPRICE    DECIMAL(15,2) NOT NULL,
             L_DISCOUNT    	DECIMAL(15,2) NOT NULL,
             L_TAX         	DECIMAL(15,2) NOT NULL,
             L_RETURNFLAG  	CHAR(1) NOT NULL,
             L_LINESTATUS  	CHAR(1) NOT NULL,
             L_SHIPDATE    	DATE NOT NULL,
             L_COMMITDATE  	DATE NOT NULL,
             L_RECEIPTDATE 	DATE NOT NULL,
             L_SHIPINSTRUCT 	 CHAR(25) NOT NULL,
             L_SHIPMODE     	 CHAR(10) NOT NULL,
             L_COMMENT      	 VARCHAR(512) NOT NULL,
             PRIMARY KEY        (L_ORDERKEY, L_PARTKEY),
             KEY 		(L_SHIPDATE, L_ORDERKEY),
             KEY 		(L_ORDERKEY, L_SUPPKEY),
             KEY 		(L_COMMITDATE, L_PARTKEY),
             KEY 		(L_PARTKEY, L_ORDERKEY),
             KEY 		(L_PARTKEY, L_SUPPKEY),
             KEY 		(L_SUPPKEY, L_RECEIPTDATE),
             KEY 		(L_SUPPKEY, L_ORDERKEY),
             KEY 		(L_SUPPKEY, L_PARTKEY)
);
```

4 索引的表结构如下：
```
CREATE TABLE lineitem  (
             L_ORDERKEY    	BIGINT NOT NULL,
             L_PARTKEY     	BIGINT NOT NULL,
             L_SUPPKEY     	INTEGER NOT NULL,
             L_LINENUMBER  	INTEGER NOT NULL,
             L_QUANTITY    	DECIMAL(15,2) NOT NULL,
             L_EXTENDEDPRICE    DECIMAL(15,2) NOT NULL,
             L_DISCOUNT    	DECIMAL(15,2) NOT NULL,
             L_TAX         	DECIMAL(15,2) NOT NULL,
             L_RETURNFLAG  	CHAR(1) NOT NULL,
             L_LINESTATUS  	CHAR(1) NOT NULL,
             L_SHIPDATE    	DATE NOT NULL,
             L_COMMITDATE  	DATE NOT NULL,
             L_RECEIPTDATE 	DATE NOT NULL,
             L_SHIPINSTRUCT 	 CHAR(25) NOT NULL,
             L_SHIPMODE     	 CHAR(10) NOT NULL,
             L_COMMENT      	 VARCHAR(512) NOT NULL,
             PRIMARY KEY (`L_ORDERKEY`,`L_PARTKEY`),
             KEY `L_ORDER` (`L_ORDERKEY`),
             KEY `PART` (`L_PARTKEY`),
             KEY `SUPP` (`L_SUPPKEY`)
);
```

MySQL on Terark 的 my.cnf 配置

```
rocksdb
default-storage-engine=rocksdb
skip-innodb
default-tmp-storage-engine=MyISAM
character-set-server=utf8
collation-server=utf8_bin
user = mysql
bind-address = 0.0.0.0
port = 3307
back_log = 600
max_connections = 6000
secure_file_priv=""
rocksdb_commit_in_the_middle=ON
rocksdb_bulk_load_size=1000
table_open_cache = 21397
rocksdb_allow_concurrent_memtable_write=1
rocksdb_bulk_load_index_type=skiplist
```

Terark 的环境变量设置

```
TerarkZipTable_localTempDir=$PWD/terark-temp \
TerarkZipTable_keyPrefixLen=4 \
TerarkZipTable_offsetArrayBlockUnits=128 \
TerarkZipTable_extendedConfigFile=$PWD/license \
TerarkUseDivSufSort=1 \
TerarkZipTable_max_background_compactions=5 \
TerarkZipTable_max_subcompactions=1 \
TerarkZipTable_min_merge_width=3 \
TerarkZipTable_max_merge_width=7 \
TerarkZipTable_level0_file_num_compaction_trigger=4 \
TerarkZipTable_softZipWorkingMemLimit=2G \
TerarkZipTable_hardZipWorkingMemLimit=4G \
TerarkZipTable_write_buffer_size=1G \
TerarkZipTable_target_file_size_base=2G \
TerarkZipTable_target_file_size_multiplier=1 \
TerarkZipTable_indexCacheRatio=0 \
TerarkZipTable_warmUpIndexOnOpen=false \
TerarkZipTable_sampleRatio=0.015 \
TerarkZipTable_disableFewZero=true \
TerarkZipTable_enable_partial_remove=true \
Terark_enableChecksumVerify=0 \
```

MySQL 的 my.cnf 配置

```
character-set-server=utf8
collation-server=utf8_bin
user = mysql
bind-address = 0.0.0.0
port = 3336
back_log = 600
max_connections = 6000
innodb_buffer_pool_size = 48G
secure_file_priv=""
thread_cache_size = 300
innodb_log_buffer_size = 512M
innodb_flush_log_at_trx_commit = 1
innodb_thread_concurrency = 0
innodb_io_capacity = 20000
innodb_buffer_pool_instances = 16
innodb_doublewrite = 0
innodb_adaptive_hash_index = 0
```

查看 MyRocks 各索引压缩率
```
select TABLE_SCHEMA,
       TABLE_NAME,
       INDEX_NAME,
       sum(DATA_SIZE) as DATA_SIZE,
       sum(FILE_SIZE) as FILE_SIZE,
       sum(FILE_SIZE) / sum(DATA_SIZE) as ZIP_RATIO
  from information_schema.ROCKSDB_DDL
  left join information_schema.ROCKSDB_INDEX_FILE_MAP
    on information_schema.ROCKSDB_DDL.INDEX_NUMBER = information_schema.ROCKSDB_INDEX_FILE_MAP.INDEX_NUMBER
  group by TABLE_SCHEMA, TABLE_NAME, INDEX_NAME;
```
注：该压缩率未包含字典尺寸  
若 INDEX_NAME 无法展示，请执行一次下面的查询
```
select * from information_schema.STATISTICS;
```
