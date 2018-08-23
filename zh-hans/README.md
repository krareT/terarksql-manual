## TerarkSQL 使用手册

### 1.产品简介

TerarkSQL 是一款依托于 Terark(terark.com) 公司研发的 TerarkDB 存储引擎实现的 MySQL 修改版，该产品目前对于非商业用途完全免费。

TerarkSQL 由 TerarkDB 存储引擎和 MyRocks 组成：

- **TerarkDB** 使用 RocksDB 的上层框架，在底层实现了一个的 `SSTable`，从而，TerarkDB 完全兼容 RocksDB API，应用程序甚至无需重新编译，只需要替换 RocksDB 的动态库(`librocksdb.so`) 即可，并且，原有的 RocksDB 数据可以透明地迁移到 TerarkDB。
- **MyRocks** 是 Facebook 发布并在使用的 MySQL 修改版，它使用了 RocksDB 作为存储引擎，所以自然我们也可以通过它将 TerarkDB 嵌入 MySQL。

### 2.产品优势
- 更好的压缩率，通常可以比使用了 InnoDB/RocksDB 存储引擎的 MySQL 节省至少一倍的存储空间
- 更强的随机读性能，通常会比其他存储引擎快 3～5 倍


### 3.注意事项
- MyRocks 基于 MySQL-5.6，同时也 Back port 了部分 MySQL-5.7 特有的功能
- MyRocks 功能上相对原生 MySQL-5.6 有一些限制：[MyRocks-limitations](https://github.com/facebook/mysql-5.6/wiki/MyRocks-limitations)


### 4.联系我们
- contact@terark.com
- [www.terark.com](http://www.terark.com)
