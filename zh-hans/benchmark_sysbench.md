## 简介
sysbench 是一个模块化的、跨平台、多线程基准测试工具,主要用于评估测试各种不同系统参数下的数据库负载情况。本测试使用 sysbench 分别向官方原版 MySQL 和 MySQL on TerarkDB 导入 **450,000,000** 条数据，测试在不同内存下两者的读写性能。

测试程序使用 [sysbench 1.1.0](https://github.com/Terark/sysbench)，我们在原版 sysbench 的基础上添加了一个次级主键范围查询。

## 测试平台

- CPU: Intel(R) Xeon(R) CPU E5-2630 v3 @ 2.40GHz x2 （共16核32线程）
- 内存: DDR4 16G @ 1866 MHz x 12 （共 192 G）
- SSD: INTEL SSDSC2BP48 0420 IOPS 89000
- 操作系统: CentOS 7

测试中使用的官方原版 MySQL 版本为 Ver 5.6.35 for linux-glibc2.5 on x86_64，后记为 InnoDB（[MySQL on TerarkDB](http://terark.com/docs/mysql-on-terarkdb-manual/zh-hans/installation.html) 记为 TerarkDB）。

## 导入

测试中使用 sysbench 导入了 **450,000,000** 条数据。数据均为 **uniform** 分布。

导入完后各数据库大小如下：

|      | 数据库大小 |
|:----:|:---------:|
| InnoDB   | 101 G |
| TerarkDB | 51 G  |

导入数据所使用的 sysbench 命令如下：

```
sysbench --report-interval=1 --db-driver=mysql --mysql-port=3306 \
         --mysql-user=root --mysql-db=sysbench --mysql-host=127.0.0.1 \
         --threads=32 --tables=1 --mysql_storage_engine=innodb \
         --table-size=450000000 --rand-type=uniform --create_secondary=on \
         /path/to/share/sysbench/oltp_insert.lua prepare
```

注1：插入时一定要指定 **--rand-type** 为 **uniform**，其默认值 special 为热点分布，导入的数据不能体现数据库真实的随机读写性能。

## 读写测试

读写测试进行了主键等值查询（point_select），读写混合查询（point_select90_update10），次级索引等值查询（secondary_random_points100），次级索引范围查询（secondary_random_limit100）四种测试，并分别在 188G、32G、8G 内存下进行。不同的内存限制使用内存挤占工具实现，内存挤占工具挤占一定数量的内存（不可换出）确保数据库所能使用的内存为以上指定值。

所有的读写测试均使用 **32** 个线程，每次测试前先 warm up **30 秒**，每次测试持续 **15 分钟**。

<table>
    <tr>
             <th></th><th></th><th colspan="3">TerarkDB</th><th colspan="3">InnoDB</th>
    </tr>
    <tr>
             <td></td> <td></td> <td>qps</td> <td>tps</td> <td>rps</td> <td>qps</td> <td>tps</td> <td>rps</td>
    </tr>
    <tr>
             <td rowspan="4">192G</td> <td>point_select</td> <td>123,615</td> <td>1,236.15</td> <td>123,615</td>
             <td>178,282</td> <td>1,782.82</td> <td>178,282</td>
    </tr>
    <tr>
             <td>point_select90_update10</td> <td>101,410</td> <td>1,014.10</td> <td>101,410</td>
             <td>50,695</td> <td>506.95</td> <td>50,695</td>
    </tr>
    <tr>
             <td>secondary_random_points100</td> <td>5,143</td> <td>5,143.00</td> <td>514,300</td>
             <td>14,278</td> <td>14,278.79</td> <td>1,427,800</td>
    </tr>
    <tr>
             <td>secondary_random_limit100</td> <td>9,139</td> <td>91.39</td> <td>913,900</td>
             <td>21,164</td> <td>211.64</td> <td>2,116,400</td>
    </tr>
    <tr>
             <td rowspan="4">32G</td><td>point_select</td> <td>89,998</td> <td>899.98</td> <td>89,998</td>
             <td>22,301</td> <td>223.01</td> <td>22,301</td>
    </tr>
    <tr>
             <td>point_select90_update10</td> <td>46,122</td> <td>461.22</td> <td>46,122</td>
             <td>12,445</td> <td>124.45</td> <td>12,445</td>
    </tr>
    <tr>
             <td>secondary_random_points100</td> <td>1,309</td> <td>1,309.22</td> <td>130922</td>
             <td>228</td> <td>227.68</td> <td>22,768</td>
    </tr>
    <tr>
             <td>secondary_random_limit100</td> <td>1,743</td> <td>17.43</td> <td>174,300</td>
             <td>232</td> <td>2.32</td> <td>23,200</td>
    </tr>
    <tr>
             <td rowspan="4">8G</td> <td>point_select</td> <td>68,864</td> <td>688.64</td> <td>68,864</td>
             <td>23,829</td> <td>238.29</td> <td>23,829</td>
    </tr>
    <tr>
             <td>point_select90_update10</td> <td>29,916</td> <td>299.16</td> <td>29,916</td>
             <td>12,787</td> <td>127.87</td> <td>17,787</td>
    </tr>
    <tr>
             <td>secondary_random_points100</td> <td>841</td> <td>841.00</td> <td>84,100</td>
             <td>172</td> <td>171.63</td> <td>17,163</td>
    </tr>
    <tr>
             <td>secondary_random_limit100</td> <td>925</td> <td>9.25</td> <td>92,500</td>
             <td>251</td> <td>2.51</td> <td>25,100</td>
    </tr>
</table>

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
sysbench --time=900 --report-interval=1 --db-driver=mysql --mysql-port=3306 \
         --mysql-user=root --mysql-db=sysbench --mysql-host=127.0.0.1 \
         --threads=32 --warmup-time=30 --distinct_ranges=0 \
         --sum_ranges=0 --index_updates=0 --range_size=100 \
         --delete_inserts=0 --tables=1 --mysql_storage_engine=rocksdb \
         --non_index_updates=0 --table-size=450000000 --simple_ranges=0 --secondary_ranges=0\
         --order_ranges=0 --range_selects=off --point_selects=100 \
         --rand-type=uniform --skip_trx=on /path/to/share/sysbench/oltp_read_only.lua run
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
sysbench --time=900 --report-interval=1 --db-driver=mysql --mysql-port=3306 \
         --mysql-user=root --mysql-db=sysbench --mysql-host=127.0.0.1 \
         --threads=32 --warmup-time=30 --distinct_ranges=0 \
         --sum_ranges=0 --index_updates=0 --range_size=100 \
         --delete_inserts=0 --tables=1 --mysql_storage_engine=rocksdb \
         --non_index_updates=10 --table-size=450000000 --simple_ranges=0 --secondary_ranges=0\
         --order_ranges=0 --range_selects=off --point_selects=90 \
         --rand-type=uniform --skip_trx=on /path/to/share/sysbench/oltp_read_write.lua run
```

#### 3. secondary_random_points100

次级主键等值查询，每个 transaction 包含一个含有 100 个值的 where 次级主键等值查询 query。

- 示例 SQL：
```
select id, k, c, pad from sbtest1 where k in (k1, k2, k3, ..., k100);
```

- sysbench 命令
```
sysbench --time=900 --report-interval=1 --db-driver=mysql --mysql-port=3306 \
         --mysql-user=root --mysql-db=sysbench --mysql-host=127.0.0.1 \
         --threads=32 --warmup-time=30 --tables=1 --table-size=450000000 \
         --rand-type=uniform --skip_trx=on \
         --random_points=100 /path/to/share/sysbench/select_random_points.lua run
```

#### 4. secondary_random_limit100

次级主键范围查询，该查询为我们新添加的测试，用来测试次级主键的随机读性能。每个 transaction 包含 100 个次级主键范围查询 query。

- 示例 SQL：
```
select c from sbtest1 where k >= K limit 100;
```

- sysbench 命令
```
sysbench --time=900 --report-interval=1 --db-driver=mysql --mysql-port=3306 \
         --mysql-user=root --mysql-db=sysbench --mysql-host=127.0.0.1 \
         --threads=32 --warmup-time=30 --distinct_ranges=0 \
         --sum_ranges=0 --index_updates=0 --range_size=100 \
         --delete_inserts=0 --tables=1 --mysql_storage_engine=rocksdb \
         --non_index_updates=0 --table-size=450000000 --simple_ranges=0 --secondary_ranges=100 \
         --order_ranges=0 --range_selects=on --point_selects=0 \
         --rand-type=uniform --skip_trx=on /path/to/share/sysbench/oltp_read_only.lua run
```
