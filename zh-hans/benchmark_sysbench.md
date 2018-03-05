## 简介
sysbench 是一个模块化的、跨平台、多线程基准测试工具,主要用于评估测试各种不同系统参数下的数据库负载情况。本测试使用 sysbench 分别向官方原版 MySQL 和 MySQL on TerarkDB 导入 **450,000,000** 条数据，测试在不同内存下两者的读写性能。

测试程序使用 [sysbench 1.1.0](https://github.com/akopytov/sysbench)

## 测试平台

- CPU: Intel(R) Xeon(R) CPU E5-2630 v3 @ 2.40GHz x2 （共16核32线程）
- 内存: DDR4 16G @ 1866 MHz x 12 （共 192 G）
- SSD: INTEL SSDSC2BP48 0420 IOPS 89000
- 操作系统: CentOS 7

测试中使用的官方原版 MySQL 版本为 Ver 5.6.35 for linux-glibc2.5 on x86_64，后记为 innodb（MySQL on TerarkDB 记为 terarkdb）。

## 导入

测试中使用 sysbench 导入了 **450，000，000** 条数据。数据均为 uniform 分布。

导入完后各数据库大小如下：

|      | 数据库大小 |
|:----:|:---------:|
| innodb   | 101 G |
| terarkdb | 51 G  |

插入所使用的 sysbench 命令如下：

```
sysbench --report-interval=1 --db-driver=mysql --mysql-port=3306 --mysql-user=root --mysql-db=sysbench --mysql-host=127.0.0.1 --threads=32 --tables=1 --mysql_storage_engine=innodb --table-size=450000000 --rand-type=uniform --create_secondary=on /path/to/share/sysbench/oltp_insert.lua prepare
```

注1：插入时一定要指定 **--rand-type** 为 **uniform**，其默认值 special 为热点分布，导入的数据不能体现数据库真实的随机读写性能。

## 读写测试

读写测试进行了主键等值查询（point_select），读写混合查询（point_select90_update10），次级索引等值查询（secondary_random_points100），次级索引范围查询（secondary_random_limit100）四种测试，并分别在 188G、32G、8G 内存下进行。不同的内存限制使用内存挤占工具实现，内存挤占工具挤占一定数量的内存（不可换出）确保数据库所能使用的内存为以上指定值。

所有的读写测试均使用 **32** 个线程，每次测试前先 warm up **30** 秒，每次测试持续 15 分钟。

|     |     | terarkdb | terarkdb | terarkdb | innodb | innodb | innodb |
|:---:|:---:|:--------:|:--------:|:--------:|:------:|:------:|:------:|
|      |                            | qps     | tps      | rps     | qps     | tps       | rps       |
| 192G | point_select               | 123,615 | 1,236.15 | 123,615 | 178,282 | 1,782.82  | 178,282   |
| 192G | point_select90_update10    | 101,410 | 1,014.10 | 101,410 | 50,695  | 506.95    | 50,695    |
| 192G | secondary_random_points100 | 5,143   | 5,143.00 | 514,300 | 14,278  | 14,278.79 | 1,427,800 |
| 192G | secondary_random_limit100  | 9,139   | 91.41    | 913,900 | 21,164  | 211.64    | 2,116,400 |
| 32G  | point_select               | 89,998  | 899.98   | 89,998  | 22,301  | 223.01    | 22,301    |
| 32G  | point_select90_update10    | 46,122  | 461.22   | 46,122  | 12,445  | 124.45    | 12,445    |  
| 32G  | secondary_random_points100 | 1,309   | 1,309.22 | 130922  | 228     | 227.68    | 22,768    |
| 32G  | secondary_random_limit100  | 1,743   | 17.43    | 174,300 | 232     | 2.32      | 23,200    |
| 8G   | point_select               | 68,864  | 688.64   | 68,864  | 23,829  | 238.29    | 23,829    |
| 8G   | point_select90_update10    | 29,916  | 299.16   | 29,916  | 12,787  | 127.87    | 17,787    |
| 8G   | secondary_random_points100 | 841     | 841.00   | 84,100  | 172     | 171.63    | 17,163    |
| 8G   | secondary_random_limit100  | 925     | 9.25     | 92,500  | 251     | 2.51      | 25,100    |

注2：

上表中 qps 表示 queries per second， tps 表示 transactions per second， rps 表示 rows per second。

注3：

#### 1. point_select

主键等值查询，每个 transaction 里包含 100 个主键等值查询 query。

- 示例 SQL：
```
select c from sbtest1 where id = ID;
```

- sysbench 命令
```
sysbench --time=900 --report-interval=1 --db-driver=mysql --mysql-port=3306 --mysql-user=root --mysql-db=sysbench --mysql-host=127.0.0.1 --threads=32 --warmup-time=30 --distinct_ranges=0 --sum_ranges=0 --index_updates=0 --range_size=100 --delete_inserts=0 --tables=1 --mysql_storage_engine=rocksdb --non_index_updates=0 --table-size=450000000 --simple_ranges=0 --order_ranges=0 --range_selects=off --point_selects=100 --rand-type=uniform --skip_trx=on /path/to/share/sysbench/oltp_read_only.lua run
```

#### 2. point_select90_update10

读写混合测试，每个 transaction 包含 90 个主键等值查询 query，和 10 个非主键更新 query。

- 示例 SQL：
```
select c from sbtest1 where id = ID;

update sbtest1 set c = C where id = ID;
```

- sysbench 命令
```
sysbench --time=900 --report-interval=1 --db-driver=mysql --mysql-port=3306 --mysql-user=root --mysql-db=sysbench --mysql-host=127.0.0.1 --threads=32 --warmup-time=30 --distinct_ranges=0 --sum_ranges=0 --index_updates=0 --range_size=100 --delete_inserts=0 --tables=1 --mysql_storage_engine=rocksdb --non_index_updates=10 --table-size=450000000 --simple_ranges=0 --order_ranges=0 --range_selects=off --point_selects=90 --rand-type=uniform --skip_trx=on /path/to/share/sysbench/oltp_read_write.lua run
```

#### 3. secondary_random_points100

次级主键等值查询，每个 transaction 包含一个含有 100 个值的 where 次级主键等值查询 query。

- 示例 SQL：
```
select id, k, c, pad from sbtest1 where k in (k1, k2, k3, ..., k100);
```

- sysbench 命令
```
sysbench --time=900 --report-interval=1 --db-driver=mysql --mysql-port=3306 --mysql-user=root --mysql-db=sysbench --mysql-host=127.0.0.1 --threads=32 --warmup-time=30 --tables=1 --table-size=450000000 --rand-type=uniform --skip_trx=on --random_points=100 /path/to/share/sysbench/select_random_points.lua run
```

#### 4. secondary_random_limit100

次级主键范围查询，每个 transaction 包含一个次级主键范围查询 query。

- 示例 SQL：
```
select c from sbtest1 where k >= K limit 100;
```

- sysbench 命令
```
sysbench --time=900 --report-interval=1 --db-driver=mysql --mysql-port=3306 --mysql-user=root --mysql-db=sysbench --mysql-host=127.0.0.1 --threads=32 --warmup-time=30 --distinct_ranges=0 --sum_ranges=0 --index_updates=0 --range_size=100 --delete_inserts=0 --tables=1 --mysql_storage_engine=rocksdb --non_index_updates=0 --table-size=450000000 --simple_ranges=100 --order_ranges=0 --range_selects=on --point_selects=0 --rand-type=uniform --skip_trx=on /path/to/share/sysbench/oltp_read_only.lua run
```
