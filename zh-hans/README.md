## MySQL on TerarkDB 使用手册

### 1.产品简介

MySQL on TerarkDB 是一款依托于 Terark(terark.com) 公司研发的 TerarkDB 存储引擎实现的 MySQL 修改版，该产品目前对于非商业用途完全免费。

### 2.产品优势
- 更好的压缩率，通常可以比使用了 InnoDB/RocksDB 存储引擎的 MySQL 节省至少一倍的存储空间
- 更强的随机读性能，通常会比其他存储引擎快 3～5倍


### 注意事项
#### MyRocks
- MyRocks 基于 MySQL-5.6，同时也 Back port 了部分 MySQL-5.7 特有的功能
- MyRocks 功能上相对原生 MySQL-5.6 有一些限制：[MyRocks-limitations](https://github.com/facebook/mysql-5.6/wiki/MyRocks-limitations)
#### TerarkDB
- TerarkDB 的压缩速度低于官方 RocksDB（因为压缩过程中计算量更大）
- TerarkDB 的顺序读相比**略低于**官方 RocksDB
  - TerarkDB 的**顺序读**比 TerarkDB 自身的**随机读**无明显优势
  - RocksDB/InnoDB 等传统存储引擎的**顺序读**比它们自身的**随机读**要快几百倍，甚至几千倍以上

### 联系我们
- contact@terark.com
- [www.terark.com](http://www.terark.com)
