 ## 简介

TPC-C 是一个联机事物处理基准，tpcc-mysql 是 percona 基于 TPC-C 衍生出来的产品，专用于 mysql 基准测试。TPC-C 测试规范中模拟了一个比较复杂并具有代表意义的 OLTP 应用环境：假设有一个大型商品批发商，它拥有若干个分布在不同区域的商品库；每个仓库负责为 10 个销售点供货；每个销售点为 3000 个客户提供服务；每个客户平均一个订单有 10 项产品；由于一个仓库中不可能存储公司所有的货物，有一些请求必须发往其它仓库，因此，数据库在逻辑上是分布的。

该系统需要处理的交易为以下几种： 

| 交易类型         | 说明                |
| ------------ | ----------------- |
| New-Order    | 客户输入一笔新的订货交易      |
| Payment      | 更新客户账户余额以反映其支付状况  |
| Delivery     | 发货（模拟批处理交易）       |
| Order-Status | 查询客户最近交易的状态       |
| Stock-Level  | 查询仓库库存状况，以便能够及时补货 |

对于前四种类型的交易，要求响应时间在 5 秒以内；对于库存状况查询交易，要求响应时间在 20 秒以内。

测试程序使用 [terark/tpcc-mysql](https://github.com/Terark/tpcc-mysql)，我们在原版 tpcc-mysql 的基础上添加了读取文本文件作为数据源的功能。

测试的数据库有：[MySQL on TerarkDB](http://terark.com/docs/mysql-on-terarkdb-manual/zh-hans/installation.html) （下简称 TerarkDB），官方原版 MySQL（下简称 InnoDB）。MySQL 开启压缩。

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
   <td>INTEL SSDSC2BP48 0420 <strong>IOPS 49000</strong>（共 <strong>480 G</strong>）</td>
  </tr>
  <tr>
    <th>操作系统</th>
    <td>CentOS 7</td>
  </tr>
</table>

测试中使用的官方原版 MySQL 版本为 Ver 5.6.35 for linux-glibc2.5 on x86_64。

下文 G, GB 2<sup>30</sup>，而非 10<sup>9</sup>。

## 数据导入

tpcc-mysql 原版只能导入自动生成的数据，这样的数据无法体现压缩算法的优劣，所以我们修改了 tpcc-mysql 源码，以支持读取指定文件中的数据。数据导入后，数据库尺寸大小如下：

<table>
<tr>
  <th colspan="2" align="right">数据库尺寸</th>
  <th>压缩率</th>
  <th rowspan="3"></th>
  <th>索引+数据大小</th>
</tr>
<tr>
  <td align="right">InnoDB</td>
  <td align="right">66 G</td>
  <td align="right">60.8% 或 1.64倍</td>
  <td align="center" rowspan="2">108.5 G</td>
</tr>
<tr>
  <td align="right">TerarkDB</td>
  <td align="right">39 G</td>
  <td align="right">35.9% 或 2.78倍</td>
</tr>
</table>

导入数据所使用的命令如下：

```shell
./tpcc_load 127.0.0.1:3346 tpcc_test root '' 1000 data.txt
```

参数分别代表 hostname:port, database_name, user, password, warehouse, filename。

更改 warehouse 会改变数据库的整体大小。

## 测试结果

每次测试中 InnoDB 的 **innodb_buffer_pool_size** 总是设置为可用内存的 **70%**，TerarkDB 的 **softZipWorkingMemLimit** 和 **hardZipWorkingMemLimit** 分别设置为可用内存的 **1/8** 和 **1/4**. 事务隔离级别设置为 **Read committed**.

所有的测试均使用 **32** 个线程，每次测试前先 warm up **300 秒**，每次测试持续 **30 分钟**。

测试记录了 New-Order、Payment、Delivery、Order-Status、Stock-Level 五种业务的总业务次数，单位为 **T**（**T**ransactions）。

下表同时列出了更通用的 **TPS**(**T**ransactions **P**er **S**econd) 指标（直接从上述 TPC-C 指标换算过来）。
<table>
    <tr>
        <th rowspan="2">内存</th><th rowspan="2">业务类型</th><th colspan="2">30 分钟总计</th><th colspan="2"> TPS </th>
    </tr>
    <tr>
        <th>TerarkDB</th><th>InnoDB</th><th>TerarkDB</th><th>InnoDB</th>
    </tr>
    <tr align="right">
        <td rowspan="6">192G</td> <td align="left">New-Order</td> <td>1,882,344</td> <td>625,808</td> <td>1,046</td> <td>348</td>
    </tr>
    <tr align="right">
        <td align="left">Payment</td> <td>1,882,347</td> <td>625,506</td> <td>1,046</td> <td>348</td>
    </tr>
    <tr align="right">
        <td align="left">Order-Status</td> <td>188,234</td> <td>62,583</td> <td>105</td> <td>35</td>
    </tr>
    <tr align="right">
        <td align="left">Delivery</td> <td>188,233</td> <td>62,574</td> <td>105</td> <td>35</td>
    </tr>
    <tr align="right">
        <td align="left">Stock-Level</td> <td>188,233</td> <td>62,587</td> <td>105</td> <td>35</td>
    </tr>
    <tr align="right">
    </tr>
    <tr align="right">
        <td rowspan="6">32G</td> <td align="left">New-Order</td> <td>1,568,769</td> <td>472,355</td> <td>872</td> <td>262</td>
    </tr>
    <tr align="right">
        <td align="left">Payment</td> <td>1,568,773</td> <td>472,398</td> <td>872</td> <td>262</td>
    </tr>
    <tr align="right">
        <td align="left">Order-Status</td> <td>156,878</td> <td>47,234</td> <td>87</td> <td>26</td>
    </tr>
    <tr align="right">
        <td align="left">Delivery</td> <td>156,878</td> <td>47,238</td> <td>87</td> <td>26</td>
    </tr>
    <tr align="right">
        <td align="left">Stock-Level</td> <td>156,877</td> <td>47,240</td> <td>87</td> <td>26</td>
    </tr>
    <tr align="right">
    </tr>
    <tr align="right">
        <td rowspan="6">24G</td> <td align="left">New-Order</td> <td>1,400,550</td> <td>437,695</td> <td>778</td> <td>243</td>
    </tr>
    <tr align="right">
        <td align="left">Payment</td> <td>1,400,549</td> <td>437,740</td> <td>778</td> <td>243</td>
    </tr>
    <tr align="right">
        <td align="left">Order-Status</td> <td>140,055</td> <td>43,775</td> <td>78</td> <td>24</td>
    </tr>
    <tr align="right">
        <td align="left">Delivery</td> <td>140,056</td> <td>43,774</td> <td>78</td> <td>24</td>
    </tr>
    <tr align="right">
        <td align="left">Stock-Level</td> <td>140,054</td> <td>43,775</td> <td>78</td> <td>24</td>
    </tr>
    <tr align="right">
    </tr>
    <tr align="right">
        <td rowspan="6">8G</td> <td align="left">New-Order</td> <td>499,297</td> <td>306,670</td> <td>277</td> <td>170</td>
    </tr>
    <tr align="right">
        <td align="left">Payment</td> <td>499,307</td> <td>306,671</td> <td>277</td> <td>170</td>
    </tr>
    <tr align="right">
        <td align="left">Order-Status</td> <td>49,930</td> <td>30,668</td> <td>28</td> <td>17</td>
    </tr>
    <tr align="right">
        <td align="left">Delivery</td> <td>49,929</td> <td>30,665</td> <td>28</td> <td>17</td>
    </tr>
    <tr align="right">
        <td align="left">Stock-Level</td> <td>49,936</td> <td>30,669</td> <td>28</td> <td>17</td>
    </tr>
    <tr align="right">
    </tr>
</table>

将该表格中的 New-Order **TPS** 指标做成更直观的图表，如下：
<hr/>


![TPCC](../images/benchmark_tpcc/New-Order-TPS.svg)

<hr/>

### 附录1：

[tpcc-mysql 表结构](https://github.com/Ashlzw/mysql-on-terarkdb-manual/blob/master/zh-hans/tpcc-mysql-table-struct.md)
