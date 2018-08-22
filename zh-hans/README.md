## TerarkSQL(原 MySQL on TerarkDB) 使用手册

### 1.产品简介

MySQL on TerarkDB 是一款依托于 Terark(terark.com) 公司研发的 TerarkDB 存储引擎实现的 MySQL 修改版，该产品目前对于非商业用途完全免费。

MySQL on TerarkDB 由 TerarkDB 存储引擎和 MyRocks 组成：

- TerarkDB 使用了 RocksDB 的上层框架，我们实际上实现了一个 RocksDB 的 `SSTable` 并且命名为 `TerarkZipTable`. 基于上面的环境变量，我们可以让 RocksDB 使用我们的 SSTable. 我们所有的算法均封装在 `TerarkZipTable ` 并且不影响现有的 `SSTable`, 也就是说您可以不启动我们版本的 SSTable，继续使用默认版本。
- MyRocks 是 Facebook 发布并在使用的 MySQL 修改版，它使用了 RocksDB 作为存储引擎，所以自然我们也可以通过它将 TerarkDB 嵌入 MySQL

### 2.产品优势
- 更好的压缩率，通常可以比使用了 InnoDB/RocksDB 存储引擎的 MySQL 节省至少一倍的存储空间
- 更强的随机读性能，通常会比其他存储引擎快 3～5 倍


### 3.注意事项
#### 3.1.MyRocks
- MyRocks 基于 MySQL-5.6，同时也 Back port 了部分 MySQL-5.7 特有的功能
- MyRocks 功能上相对原生 MySQL-5.6 有一些限制：[MyRocks-limitations](https://github.com/facebook/mysql-5.6/wiki/MyRocks-limitations)


### 4.联系我们
- contact@terark.com
- [www.terark.com](http://www.terark.com)
