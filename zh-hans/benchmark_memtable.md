

## 测试简介

rocksdb 原生 memtable（基于 skiplist）在数据大量插入时，因为其随机性而导致高度的不可预期性，在我们的测试里成为了性能瓶颈之一。
是故我们基于 trbtree 实现了自己的 memtable，以提高 MyRocks on Terark 的插入性能。本测试即为 skiplist 和 trbtree 性能对比报告。

## 测试结果

使用的数据是由 tpch 程序产生的 lineitem 数据，共 100K(10万) 条，总字节数 59MB，平均每条约 600 字节，单条数据格式如下：

```
1|16605260|822776|1|17|19795.31|0.04|0.02|N|O|1996-03-13|1996-02-12|1996-03-22|DELIVER IN PERSON|TRUCK|fluffily ironic pinto beans sleep daringly pending instructions. fluffily permanent foxes mold along the furiously express ideas. ironic pinto beans cajole fluffily unusual theodolites. carefully regular theodolites across the pending pinto beans haggle even, ironic deposits. quickly special packages above the furiously regular deposits wake carefully pending requests. carefully regular courts above the pending, even accounts haggle blithely even |
```

我们将创建 16 张表（表的结构参看附注），将这 59MB 的数据插入每张表，相当于 1.6M（160万）条，944MB 的原始数据。
插入是通过 16 个客户端同时进行的。结果如下，

|  数据结构  | 共计插入时间(秒)|
|-----------|-------------:|
|  trbtree  |   189.025    |
|  skiplist |  1385.903    |


## 附注

表的结构如下，

```
CREATE TABLE lineitem
          ( L_ORDERKEY       BIGINT        NOT NULL,
            L_PARTKEY        BIGINT        NOT NULL,
            L_SUPPKEY        INTEGER       NOT NULL,
            L_LINENUMBER     INTEGER       NOT NULL,
            L_QUANTITY       DECIMAL(15,2) NOT NULL,
            L_EXTENDEDPRICE  DECIMAL(15,2) NOT NULL,
            L_DISCOUNT       DECIMAL(15,2) NOT NULL,
            L_TAX            DECIMAL(15,2) NOT NULL,
            L_RETURNFLAG     CHAR(1)       NOT NULL,
            L_LINESTATUS     CHAR(1)       NOT NULL,
            L_SHIPDATE       DATE          NOT NULL,
            L_COMMITDATE     DATE          NOT NULL,
            L_RECEIPTDATE    DATE          NOT NULL,
            L_SHIPINSTRUCT   CHAR(25)      NOT NULL,
            L_SHIPMODE       CHAR(10)      NOT NULL,
            L_COMMENT        VARCHAR(512)  NOT NULL,

            PRIMARY KEY (L_ORDERKEY, L_PARTKEY),

            KEY (L_ORDERKEY  , L_SUPPKEY),
            KEY (L_ORDERKEY  , L_LINENUMBER),
            KEY (L_ORDERKEY  , L_SHIPDATE),
            KEY (L_ORDERKEY  , L_COMMITDATE),
            KEY (L_ORDERKEY  , L_RECEIPTDATE),
            KEY (L_ORDERKEY  , L_SUPPKEY),

            KEY (L_COMMITDATE, L_ORDERKEY),
            KEY (L_COMMITDATE, L_PARTKEY),
            KEY (L_COMMITDATE, L_SUPPKEY),
            KEY (L_COMMITDATE, L_LINENUMBER),
            KEY (L_COMMITDATE, L_SHIPDATE),
            KEY (L_COMMITDATE, L_RECEIPTDATE),

            KEY (L_PARTKEY   , L_ORDERKEY),
            KEY (L_PARTKEY   , L_SUPPKEY),
            KEY (L_PARTKEY   , L_LINENUMBER),
            KEY (L_PARTKEY   , L_SHIPDATE),
            KEY (L_PARTKEY   , L_COMMITDATE),
            KEY (L_PARTKEY   , L_RECEIPTDATE),

            KEY (L_SHIPDATE  , L_ORDERKEY),
            KEY (L_SHIPDATE  , L_PARTKEY),
            KEY (L_SHIPDATE  , L_SUPPKEY),
            KEY (L_SHIPDATE  , L_LINENUMBER),
            KEY (L_SHIPDATE  , L_COMMITDATE),
            KEY (L_SHIPDATE  , L_RECEIPTDATE),

            KEY (L_SUPPKEY   , L_ORDERKEY),
            KEY (L_SUPPKEY   , L_PARTKEY),
            KEY (L_SUPPKEY   , L_LINENUMBER),
            KEY (L_SUPPKEY   , L_SHIPDATE),
            KEY (L_SUPPKEY   , L_COMMITDATE),
            KEY (L_SUPPKEY   , L_RECEIPTDATE),

            KEY (L_SHIPINSTRUCT, L_ORDERKEY)

            );
```

其他相关配置，

```
rocksdb_bulk_load_size=10000
TerarkZipTable_write_buffer_size=8G

```

