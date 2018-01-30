
## 测试数据

测试机型为H43机型，64核CPU，内存512G，PCI-E接口SSD。数据库采用 MongoDB 分别对接 WiredTiger，RocksDB，TerarkDB作为测试参照。数据集使用线上业务真实数据内容，原始数据总容量约 3.8T，条目数大概70亿，每条平均500字节；数据内容上数字整型为主，整体数据模型上属于较难压缩的OLTP型业务数据。

## 稳定性

在构架MongoDB测试样本时，大量灌入数据和构建索引操作验证（7*24持续Batch Random Write 50MB/s写入，TPS波谷低于均值的30%）。构建速度稳定，没有因为 TerarkDB 自身问题引起不稳定现象。

## 压缩率对比

灌入数据后进 Full-Compaction，使 RocksDB 与 TerarkDB 均达到最佳状态，即数据无冗余放大。在RocksDB的块压缩原理下，基本接近块压缩的理论极限 (snappy压缩算法)。最终数据容量如表所示，整体压缩率上，TerarkDB 最优，相比前两者，分别有 45% 和 30% 的空间结余。在索引存储部分，TerarkDB 压缩率低于 WiredTiger，原因为数据内容小，且值为相对随机的整型。由于 RocksDB 状态统计问题，无法有效给出索引和数据分别容量占比。






