## 使用 YCSB 进行测试

### 1. 安装并运行 TerarkSQL

请先到官网下载并安装 TerarkSQL

测试时所使用的配置如下：(MyRocks 部分设置使用官方[推荐配置](https://github.com/facebook/mysql-5.6/wiki/my.cnf-tuning))
```ini
[mysqld]
rocksdb
default-storage-engine=rocksdb
skip-innodb
default-tmp-storage-engine=MyISAM
binlog_format=ROW
collation-server=latin1_bin
transaction-isolation=READ-COMMITTED

datadir=path/to/myrocks/data
port=3306
user=mysql

rocksdb_max_open_files=-1
rocksdb_base_background_compactions=1
rocksdb_max_background_compactions=8
rocksdb_max_total_wal_size=4G
rocksdb_max_background_flushes=4

# TerarkDB 引擎不需要这三个参数
#rocksdb_block_size=16384
#rocksdb_block_cache_size=32G
#rocksdb_table_cache_numshardbits=6

# rate limiter
rocksdb_bytes_per_sync=4194304
rocksdb_wal_bytes_per_sync=4194304
rocksdb_rate_limiter_bytes_per_sec=104857600 #100MB/s

# triggering compaction if there are many sequential deletes
rocksdb_compaction_sequential_deletes_count_sd=1
rocksdb_compaction_sequential_deletes=199999
rocksdb_compaction_sequential_deletes_window=200000

# read free replication
rocksdb_default_cf_options=level0_file_num_compaction_trigger=4;level0_slowdown_writes_trigger=10;level0_stop_writes_trigger=15;memtable_prefix_bloom_size_ratio=0.05;
```

### 2. 创建测试所需的数据库以及表
```sql
mysql -uroot -h127.0.0.1
create database ycsb;
use ycsb;
create table usertable(
  YCSB_KEY          varchar(255) primary key,
  productproductId  varchar(255),
  reviewuserId      varchar(255),
  reviewprofileName varchar(255),
  reviewhelpfulness varchar(255),
  reviewscore       varchar(255),
  reviewtime        varchar(255),
  reviewsummary     text,
  reviewtext        text);
```

### 3. 下载、处理测试所需数据
我们测试使用的是 [Amazon movie data](https://snap.stanford.edu/data/web-Movies.html)，下载存为 movies.txt，然后使用 [parser](https://github.com/Terark/amazon-movies-parser.git) 将源数据文件转换成行文本
```
git clone https://github.com/Terark/amazon-movies-parser
cd amazon-movies-parser
g++ -o parser amazon-moive-parser.cpp -std=c++11
./parser /path/to/movies.txt /path/to/movies_flat.txt
```
movies_flat.txt 即为转换后的行文本文件

### 4. 获取并编译 Terark 修改版的 YCSB
```bash
git clone git@github.com:Terark/YCSB.git
cd YCSB
checkout dev
mvn clean package
```

解压编译生成的可执行文件
```bash
cd jdbc/target
tar -zxvf ycsb-jdbc-binding-0.13.0-SNAPSHOT.tar.gz
cd ycsb-jdbc-binding-0.13.0-SNAPSHOT
```
### 5. 下载 mysql-connector-java-x.x.xx-bin.jar
下载 [mysql-connector-java-x.x.xx-bin.jar](https://dev.mysql.com/downloads/connector/j/5.1.html)，并置于 `lib/` 下

### 6. 配置测试以及连接信息
新建 movies.conf 配置测试信息
```ini
# movie.conf
recordcount=7911683
operationcount=200000

workload=com.yahoo.ycsb.workloads.FileWorkload

jdbc.driver=com.mysql.jdbc.Driver
db.url=jdbc:mysql://127.0.0.1:3306/ycsb
db.user=root
db.passwd=

datafile=/path/to/movies_flat.txt

fieldnames=productproductId,reviewuserId,reviewprofileName,reviewhelpfulness,reviewscore,reviewtime,reviewsummary,reviewtext
fieldnum=8
delimiter=\t
usecustomkey=flase

writeinread=flase
readproportion=0.95
writeproportion=0.05
```
相关选项说明在[这里](https://github.com/Terark/YCSB/blob/dev/README-terark.md)

### 7. 加载数据
```bash
bin/ycsb load jdbc -s \
    -P movies.conf \
    -threads 16 \
```
`-threads 16` 指定线程数为 16

### 8. 进行测试
```bash
bin/ycsb run jdbc -s \
    -P movies.conf \
    -threads 16 \
```

