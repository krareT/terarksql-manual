## 简介
sysbench 是一个模块化的、跨平台、多线程基准测试工具,主要用于评估测试各种不同系统参数下的数据库负载情况。本测试使用 sysbench 分别向官方原版 MySQL 和 MySQL on TerarkDB 导入 **450,000,000** 条数据，测试在不同内存下两者的读写性能。

非常值得注意的是，sysbench 测试中默认的数据分布是 `special`，表示热点非常集中的数据，从而**测试结果主要体现的是缓存的性能**。这就是为什么大家经常发现 sysbench 测试出来性能很高，一到生产环境，性能就跪了。

好在 sysbench 也有对其它数据分布的支持，例如 `uniform`，即`均匀分布`，其测试结果主要体现的是`随机访问`的性能。本文中所有的测试均使用 `uniform` 分布。

测试程序使用 [terark/sysbench 1.0.1](https://github.com/Terark/sysbench)，我们在原版 sysbench 的基础上添加了一个次级索引范围查询测试。

## 测试平台

- CPU: Intel(R) Xeon(R) CPU E5-2630 v3 @ 2.40GHz x2 （共16核32线程）
- 内存: DDR4 16G @ 1866 MHz x 12 （共 192 G）
- SSD: INTEL SSDSC2BP48 0420 IOPS 89000
- 操作系统: CentOS 7

测试中使用的官方原版 MySQL 版本为 Ver 5.6.35 for linux-glibc2.5 on x86_64，下文简称为 InnoDB（[MySQL on TerarkDB](http://terark.com/docs/mysql-on-terarkdb-manual/zh-hans/installation.html) 简称为 TerarkDB）。

## 导入

测试中使用 sysbench 导入了 **450,000,000** 条数据。数据均为 **uniform** 分布。

导入完后各数据库大小如下：

|      | 数据库大小 |
|-----:|----------:|
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

**注1**：插入时一定要指定 **--rand-type** 为 **uniform**，因为其默认值 special 为热点分布，导入的数据不能体现数据库真实的随机读写性能。

## 测试结果

我们运行了四种测试：
* 主键等值查询（point_select）
* 读写混合查询（point_select90_update10）
* 次级索引等值查询（secondary_random_points100）
* 次级索引范围查询（secondary_random_limit100）

这四种测试分别在 192G、32G、8G 的内存限制下运行，不同的内存限制使用内存挤占工具实现，内存挤占工具挤占一定数量的内存（不可换出）确保数据库所能使用的内存为以上指定值。

每次测试中 InnoDB 的 **innodb_buffer_pool_size** 总是设置为可用内存的 **70%**，TerarkDB 的 **softZipWorkingMemLimit** 和 **hardZipWorkingMemLimit** 分别设置为可用内存的 **1/8** 和 **1/4**.

所有的测试均使用 **32** 个线程，每次测试前先 warm up **30 秒**，每次测试持续 **15 分钟**。

<table>
    <tr>
             <th rowspan="2">内存</th><th rowspan="2">测试类型</th><th colspan="3">TerarkDB</th><th colspan="3">InnoDB</th>
    </tr>
    <tr align="center">
             <td>QPS</td> <td>TPS</td> <td>RPS</td> <td>QPS</td> <td>TPS</td> <td>RPS</td>
    </tr>
    <tr align="right">
             <td rowspan="4">192G</td> <td align="left">point_select</td> <td>123,615</td> <td>1,236.15</td> <td>123,615</td>
             <td>178,282</td> <td>1,782.82</td> <td>178,282</td>
    </tr>
    <tr align="right">
             <td align="left">point_select90_update10</td> <td>101,410</td> <td>1,014.10</td> <td>101,410</td>
             <td>50,695</td> <td>506.95</td> <td>50,695</td>
    </tr>
    <tr align="right">
             <td align="left">secondary_random_points100</td> <td>5,143</td> <td>5,143.00</td> <td>514,300</td>
             <td>14,278</td> <td>14,278.79</td> <td>1,427,800</td>
    </tr>
    <tr align="right">
             <td align="left">secondary_random_limit100</td> <td>9,139</td> <td>91.39</td> <td>913,900</td>
             <td>21,164</td> <td>211.64</td> <td>2,116,400</td>
    </tr>
    <tr align="right">
             <td rowspan="4">32G</td><td align="left">point_select</td> <td>89,998</td> <td>899.98</td> <td>89,998</td>
             <td>22,301</td> <td>223.01</td> <td>22,301</td>
    </tr>
    <tr align="right">
             <td align="left">point_select90_update10</td> <td>46,122</td> <td>461.22</td> <td>46,122</td>
             <td>12,445</td> <td>124.45</td> <td>12,445</td>
    </tr>
    <tr align="right">
             <td align="left">secondary_random_points100</td> <td>1,309</td> <td>1,309.22</td> <td>130,922</td>
             <td>228</td> <td>227.68</td> <td>22,768</td>
    </tr>
    <tr align="right">
             <td align="left">secondary_random_limit100</td> <td>1,743</td> <td>17.43</td> <td>174,300</td>
             <td>232</td> <td>2.32</td> <td>23,200</td>
    </tr>
    <tr align="right">
             <td rowspan="4">8G</td> <td align="left">point_select</td> <td>68,864</td> <td>688.64</td> <td>68,864</td>
             <td>23,829</td> <td>238.29</td> <td>23,829</td>
    </tr>
    <tr align="right">
             <td align="left">point_select90_update10</td> <td>29,916</td> <td>299.16</td> <td>29,916</td>
             <td>12,787</td> <td>127.87</td> <td>17,787</td>
    </tr>
    <tr align="right">
             <td align="left">secondary_random_points100</td> <td>841</td> <td>841.00</td> <td>84,100</td>
             <td>172</td> <td>171.63</td> <td>17,163</td>
    </tr>
    <tr align="right">
             <td align="left">secondary_random_limit100</td> <td>925</td> <td>9.25</td> <td>92,500</td>
             <td>251</td> <td>2.51</td> <td>25,100</td>
    </tr>
</table>

**注2：**

上表中 **Q**PS 表示 **Q**ueries Per Second，**T**PS 表示 **T**ransactions Per Second，**R**PS 表示 **R**ows Per Second。

将上表中的 **RPS** 数据做成更直观的图表，如下：

![rps_192g](../image/benchmark_sysbench/rps_192g.png)

![rps_32g](../image/benchmark_sysbench/rps_32g.png)

![rps_8g](../image/benchmark_sysbench/rps_8g.png)

### 测试类型说明

#### 1. point_select

主键等值查询，测试程序每次随机生成一个 ID 值，然后查询主键与之相等的记录。测试的每个 transaction 里包含 100 个主键等值查询 query，故每个 transaction 会访问 100 行数据。

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

读写混合测试，除上述主键等值查询外，还会随机生成一个 ID，然后更新主键与该 ID 相等的记录的 c 值。测试的每个 transaction 包含 90 个主键等值查询 query，和 10 个非主键更新 query，故每个 transaction 会访问 100 行数据，并更新 10 行数据。

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

次级索引等值查询，测试程序随机生产 100 个 key，然后使用 where in 语法随机读取这 100 个 key 对应的 100 行数据。

测试的每个 transaction 包含一个 Query，每个 Query 对次级索引进行 100 次随机搜索，每次都需要进行回表操作（先从次级键拿到主键，再用主键取数据），从而每个 transaction 需要对存储引擎进行 200 次随机访问。

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

次级索引范围查询，该查询为我们新添加的测试，用来测试主索引的随机读性能: 次级 Key 是随机生成的，次级 Key 按大小顺序映射到主键，得到的主键序列就是随机顺序，所以，这样的访问，就是次级 Key 的顺序访问再加主键的随机访问。

测试的每个 transaction 包含 100 个次级索引范围查询 query，每个 query 会访问 100 行数据，从而每个 transaction 会访问 10,000 行数据。

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
