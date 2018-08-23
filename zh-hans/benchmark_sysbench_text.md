## 简介
sysbench 是一个模块化的、跨平台、多线程基准测试工具，主要用于评估测试各种不同系统参数下的数据库负载情况。本测试使用修改版的 sysbench 分别向官方原版 MySQL 和 TerarkSQL 导入 **38,508,221** 条 [wikipedia](https://dumps.wikimedia.org/backup-index.html) 文章数据，并测试在不同内存下两者的读写性能。

非常值得注意的是，sysbench 测试中默认的数据分布是 `special`，表示热点非常集中的数据，从而**测试结果主要体现的是缓存的性能**。这就是为什么大家经常发现 sysbench 测试出来性能很高，一到生产环境，性能就跪了。

好在 sysbench 也有对其它数据分布的支持，例如 `uniform`，即`均匀分布`，其测试结果主要体现的是`随机访问`的性能。本文中所有的测试均使用 `uniform` 分布。

测试程序使用 [terark/sysbench 1.0.1](https://github.com/Terark/sysbench)，我们在原版 sysbench 的基础上添加了读取文本文件作为数据源的功能，以及一个次级索引范围查询测试。

测试的数据库有：[TerarkSQL](http://terark.com/docs/terarksql-manual/zh-hans/installation.html) （下简称 TerarkDB），官方原版 MySQL（下简称 InnoDB）。MySQL 开启压缩。

## 测试平台
<table>
  <tr>
    <th>CPU</th>
    <td>Intel(R) Xeon(R) CPU E5-2630 v3 @ 2.40GHz x2 （共 16 核 32 线程）</td>
  </tr>
  <tr>
    <th>内存</th>
    <td>DDR4 16G @ 1866 MHz x 12 （共 <strong>192 G</strong>）</td>
  </tr>
  <tr>
    <th>SSD</th>
    <td>INTEL SSDSC2BP48 0420 <strong>IOPS 89000</strong></td>
  </tr>
  <tr>
    <th>操作系统</th>
    <td>CentOS 7</td>
  </tr>
</table>
  

测试中使用的官方原版 MySQL 版本为 Ver 5.6.35 for linux-glibc2.5 on x86_64。

下文 G, GB 指 2<sup>30</sup>，而非 10<sup>9</sup>。

## 数据导入
sysbench 原版只能导入**自动生成**的数据，这样的数据无法体现压缩算法的优劣，所以我们修改了 sysbench 源码，以支持**导入指定文件**中的数据。

我们使用 [wikipedia](https://dumps.wikimedia.org/backup-index.html) dump 出来的文章数据，并提取出其中的文章**标题**和文章**内容**作为数据源（数据示例可见**附录1**），这些数据共有 **38,508,221** 条，总大小为 **94.8GB**，平均每条约 **2.6KB**。

每张表（表结构可见**附录2**）中还有一个自增主键以及一个 Secondary Index，也会占用空间，因为**辅助索引列**和**主键索引列**都是 int32，所以数据源的大小为 `94.8G+8*38,508,221=95.1G`。另外，对于每条数据，辅助索引的空间占用也是 8 字节，从而辅助索引的逻辑空间占用就是 `8*38,508,221=0.29G`。所以，数据源的等效尺寸为 `95.4G`。

数据导入后，数据库尺寸大小比较如下：
<table>
<tr>
  <th colspan="2" align="right">数据库尺寸</th>
  <th>压缩率</th>
  <th rowspan="3"></th>
  <th>数据条数</th>
  <th>单条尺寸</th>
  <th>总尺寸</th>
  <th>索引+数据</td>
</tr>
<tr>
  <td align="right">InnoDB</td>
  <td align="right">55.4 G</td>
  <td align="right">58.1% 或 1.72倍</td>
  <td align="center" rowspan="2">38,508,221</td>
  <td align="center" rowspan="2">2.6 KB</td>
  <td align="center" rowspan="2">95.1 G</td>
  <td align="center" rowspan="2">95.4 G</td>
</tr>
<tr>
  <td align="right">TerarkDB</td>
  <td align="right">20.7 G</td>
  <td align="right">21.7% 或 4.61倍</td>
</tr>
</table>

导入数据所使用的 sysbench 命令如下：

```
sysbench --report-interval=1 --db-driver=mysql --mysql-port=3306 \
         --mysql-user=root --mysql-db=sysbench --mysql-host=127.0.0.1 \
         --tables=1 --mysql_storage_engine=innodb \
         --table-size=38508221 --rand-type=uniform --create_secondary=on \
         --use-file=on --filename=/path/to/wikipedia-article.txt \
         /path/to/share/sysbench/oltp_insert.lua prepare
```

**注1**：插入时一定要指定 **--rand-type** 为 **uniform**，因为其默认值 special 为热点分布，其辅助索引以及查询分布为热点分布，从而不能反映出**随机**读写的性能。

## 测试结果

我们进行了四种测试：
* 主键等值查询（point_select）
* 读写混合查询（point_select90_update10）
* 次级索引等值查询（secondary_random_points100）
* 次级索引范围查询（secondary_random_limit100）

这四种测试分别在 192G、32G、24G、8G 的内存限制下运行。不同的内存限制使用内存挤占工具实现，内存挤占工具挤占一定数量的内存（不可换出）确保数据库所能使用的内存为以上指定值。

每次测试中 InnoDB 的 **innodb_buffer_pool_size** 总是设置为可用内存的 **70%**，TerarkDB 的 **softZipWorkingMemLimit** 和 **hardZipWorkingMemLimit** 分别设置为可用内存的 **1/8** 和 **1/4**.

所有的测试均使用 **32** 个线程，每次测试前先 warm up **30 秒**，每次测试持续 **15 分钟**。

下表中仅记录各测试结果的 **RPS**（**R**ows Per Second）。
<table>
    <tr>
             <th>内存</th><th>测试类型</th><th>TerarkDB</th><th>InnoDB</th>
    </tr>
    <tr align="right">
             <td rowspan="4">192G</td> <td align="left">point_select</td> <td>151,250</td> <td>269,841</td>
    </tr>
    <tr align="right">
             <td align="left">point_select90_update10</td> <td>90,764</td> <td>21,387</td>
    </tr>
    <tr align="right">
             <td align="left">secondary_random_points100</td> <td>484,432</td> <td>712,815</td>
    </tr>
    <tr align="right">
             <td align="left">secondary_random_limit100</td> <td>791,500</td> <td>2,284,200</td>
   </tr>
    <tr align="right">
             <td rowspan="4">32G</td><td align="left">point_select</td> <td>151,730</td> <td>95,090</td>
    </tr>
    <tr align="right">
             <td align="left">point_select90_update10</td> <td>68,299</td> <td>8,098</td>
    </tr>
    <tr align="right">
             <td align="left">secondary_random_points100</td> <td>473,500</td> <td>107,348</td>
    </tr>
    <tr align="right">
             <td align="left">secondary_random_limit100</td> <td>782,300</td> <td>152,804</td>
    </tr>
    <tr align="right">
             <td rowspan="4">24G</td> <td align="left">point_select</td> <td>151,844</td> <td>80,775</td>
    </tr>
    <tr align="right">
             <td align="left">point_select90_update10</td> <td>54,843</td> <td>6,411</td>
    </tr>
    <tr align="right">
             <td align="left">secondary_random_points100</td> <td>476,500</td> <td>87,863</td>
    </tr>
    <tr align="right">
             <td align="left">secondary_random_limit100</td> <td>784,500</td> <td>123,700</td>
    </tr>
  <tr align="right">
             <td rowspan="4">8G</td> <td align="left">point_select</td> <td>96,688</td> <td>57,570</td>
    </tr>
    <tr align="right">
             <td align="left">point_select90_update10</td> <td>28,673</td> <td>5,477</td>
    </tr>
    <tr align="right">
             <td align="left">secondary_random_points100</td> <td>112,237</td> <td>51,651</td>
    </tr>
    <tr align="right">
             <td align="left">secondary_random_limit100</td> <td>143,800</td> <td>72,800</td>
    </tr>
</table>

将上表中的数据做成更直观的图表，如下：
<hr/>

![rps_192g](../images/benchmark_sysbench/text_rps_192g.svg)

192G 内存对 TerarkDB 和 InnoDB 都**够用**，实际上，TerarkDB 只使用了大约 21G，InnoDB 则使用了 134G 内存（进程内存 + 系统缓存）。

TerarkDB 的**只读**性能低于 InnoDB，主要是因为 MyRocks 适配层带来的性能损失（相比引擎层损失了 **10** 倍以上的性能）。*MyRocks 适配层的开发一直很活跃，版本更新经常会发生性能骤变（提高或降低都会发生，该测试使用的这个版本性能比之前就有所下降，但随着时间的推移，总的趋势是向上的）*。

TerarkDB 的**读写混合**性能高于 InnoDB，是因为 TerarkDB 通过 RocksDB 使用了 LSM Tree，**随机写**性能从根上就远优于 InnoDB 的 BTree。

<hr/>

![rps_32g](../images/benchmark_sysbench/text_rps_32g.svg)

32G 内存，TerarkDB **够用**，但 InnoDB **不够用**，TerarkDB 尽管有 MyRocks 适配层带来的性能损失，但 InnoDB 因为内存不够受限于 IO 瓶颈 ，从而 TerarkDB 的性能远高于 InnoDB。

<hr/>

![rps_24g](../images/benchmark_sysbench/text_rps_24g.svg)

24G 内存，TerarkDB 仍然**够用**，但 InnoDB **很不够用**，InnoDB 性能进一步下降，而 TerarkDB 性能基本不受影响，高压缩率带来的优势由此可见。

<hr/>

![rps_8g](../images/benchmark_sysbench/text_rps_8g.svg)

8G 内存对 TerarkDB 和 InnoDB 都**很不够用**，TerarkDB 的 IO 也成为瓶颈，但 InnoDB 的内存缺口更大，从而 TerarkDB 的性能仍远高于 InnoDB。
<hr/>

### 测试类型说明

#### 1. point_select

主键等值查询，测试程序每次随机生成一个 ID 值，然后查询主键与之相等的记录。测试的每个 transaction 里包含 100 个主键等值查询 query，故每个 transaction 会访问 100 行数据。

- 示例 SQL：

```
select c from sbtest1 where id = ID;
```

- sysbench 命令：

```
sysbench --time=900 --report-interval=1 --db-driver=mysql --mysql-port=3306 \
         --mysql-user=root --mysql-db=sysbench --mysql-host=127.0.0.1 \
         --threads=32 --warmup-time=30 --distinct_ranges=0 \
         --sum_ranges=0 --index_updates=0 --range_size=100 \
         --delete_inserts=0 --tables=1 --mysql_storage_engine=rocksdb \
         --non_index_updates=0 --table-size=38508221 --simple_ranges=0 --secondary_ranges=0\
         --order_ranges=0 --range_selects=off --point_selects=100 \
         --rand-type=uniform --skip_trx=on /path/to/share/sysbench/oltp_read_only.lua run
```

#### 2. point_select90_update10

读写混合测试，除上述主键等值查询外，还会随机生成一个 ID，然后更新主键与该 ID 相等的记录的 c 值为一个随机的 119 字节长的随机字符串。测试的每个 transaction 包含 90 个主键等值查询 query，和 10 个非主键更新 query，故每个 transaction 会访问 100 行数据，并更新 10 行数据。

- 示例 SQL：

```
select c from sbtest1 where id = ID;

update sbtest1 set c = C where id = ID;
```

- sysbench 命令：

```
sysbench --time=900 --report-interval=1 --db-driver=mysql --mysql-port=3306 \
         --mysql-user=root --mysql-db=sysbench --mysql-host=127.0.0.1 \
         --threads=32 --warmup-time=30 --distinct_ranges=0 \
         --sum_ranges=0 --index_updates=0 --range_size=100 \
         --delete_inserts=0 --tables=1 --mysql_storage_engine=rocksdb \
         --non_index_updates=10 --table-size=38508221 --simple_ranges=0 --secondary_ranges=0\
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

- sysbench 命令：

```
sysbench --time=900 --report-interval=1 --db-driver=mysql --mysql-port=3306 \
         --mysql-user=root --mysql-db=sysbench --mysql-host=127.0.0.1 \
         --threads=32 --warmup-time=30 --tables=1 --table-size=38508221 \
         --rand-type=uniform --skip_trx=on \
         --random_points=100 /path/to/share/sysbench/select_random_points.lua run
```

#### 4. secondary_random_limit100

次级索引范围查询，该查询为我们新添加的测试，用来测试主索引的随机读性能：次级 Key 是随机生成的，次级 Key 按大小顺序映射到主键，得到的主键序列就是随机顺序，所以，这样的访问，就是次级 Key 的顺序访问再加主键的随机访问。

测试的每个 transaction 包含 100 个次级索引范围查询 query，每个 query 会访问 100 行数据，从而每个 transaction 会访问 10,000 行数据。

- 示例 SQL：

```
select c from sbtest1 where k >= K limit 100;
```

- sysbench 命令：

```
sysbench --time=900 --report-interval=1 --db-driver=mysql --mysql-port=3306 \
         --mysql-user=root --mysql-db=sysbench --mysql-host=127.0.0.1 \
         --threads=32 --warmup-time=30 --distinct_ranges=0 \
         --sum_ranges=0 --index_updates=0 --range_size=100 \
         --delete_inserts=0 --tables=1 --mysql_storage_engine=rocksdb \
         --non_index_updates=0 --table-size=38508221 --simple_ranges=0 --secondary_ranges=100 \
         --order_ranges=0 --range_selects=on --point_selects=0 \
         --rand-type=uniform --skip_trx=on /path/to/share/sysbench/oltp_read_only.lua run
```

### 附录1：

数据源示例：

```
'AccessibleComputing'	'#REDIRECT [[Computer accessibility]]\n\n{{Redr|move|from CamelCase|up}}'
'Liberty_Hall_(Lamoni,_Iowa)'	'{{WikiProject National Register of Historic Places|class=Stub|importance=Low}}\n{{WikiProject Iowa|class=Stub|importance=Low}}\n{{reqphoto|in=Decatur County, Iowa}}'
```

更多数据[示例](wikipedia-article-sample.txt)

### 附录2：
表结构：

```
CREATE TABLE sbtest1 (
  id int(11) NOT NULL AUTO_INCREMENT,
  k int(11) NOT NULL DEFAULT '0',
  c varchar(512) NOT NULL DEFAULT '',
  pad mediumtext,
  PRIMARY KEY (id),
  KEY k_1 (k)
);
```
