
## MyRocks + Terark

### 插入阶段

插入表结构及配置参见附注。

共 120G 数据，插入耗时 3.5 小时。初次插完空间占用为 57G，经过 compaction 后大小为 35G，插入过程中需要总空间 > 60G。


### 查询阶段

使用测试程序 [MyRocks](https://github.com/Terark/MyRocksTest), 32 线程，


| 内存限制*   |  QPS   | CPU 占用  | TPS(iostat) | kB_read(iostat) |
|-----------|--------|----------|-------------|-----------------|
| 不限内存   | 97,000 | 2400%    |  517        |  2,068          |
| 限制为 32G | 95,000 | 2400%    |  349        |  1,396          |
| 为 24G    | 78,000 | 2000%    |  17,000     |  68,104         |
| 为 16G    | 47,000 | 1400%    |  44,405     |  177,620        |
| 为 8G     | 28,000 | 900%     |  48,000     |  195,016        |

注：这里使用了 cgroup 的方式进行内存限制。

## MyRocks + InnoDB

### 插入阶段

插入表结构及配置参见附注。

共 120G 数据，插入 21 小时共插入 90% 左右的数据，大小 210G 左右。期间 kill -9 后重启，耗时1小时10分钟；（注：没有设置 ```buffer_pool_size``` 也即使用的是默认配置）


### 查询阶段

使用测试程序 [MyRocks](https://github.com/Terark/MyRocksTest), 使用 32个 线程，buffer_cache 设置为 96G，

| 内存限制*  |  QPS   | CPU 占用  | TPS(iostat) | kB_read(iostat) |
|-----------|--------|----------|-------------|-----------------|
| 不限内存**  | 47,000 |  900%   |             |                 |
| 限制为 96G | 28,000 |  470%    |  14,995     |  473,572        |
| 为 32G    | 19,000 |  400%    |  18,326     |  450,656        |
| 为 16G    | 14,500 |  350%    |  19,076     |  438,828        |
| 为 8G     | 14,000 |  400%    |  22,051     |  422,212        |
 
注* : 系统总计内存 188G，不足以装下所有数据。这里 “不限内存” 与 “限制为 96G” 对应的```buffer_pool_size``` 设置为 96G。其余测试的 ```buffer_pool_size````均为限制后系统内存的一半。

注** : 通过在启动界面进行内存限制。

## 附注

建表如下，从 lineitem1 到 lineitem100 共 100 张表，

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
             PRIMARY KEY      (L_ORDERKEY, L_PARTKEY),
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
