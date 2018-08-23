## 配置参数说明

### 1.简介
MySQL on TerarkDB 的配置分为两部分：
- MySQL 自身的配置选项，依然通过 my.cnf 进行配置（包括 MyRocks 新支持的部分配置）
- TerarkDB 相关的配置，均通过环境变量进行配置

由于 `Terark RocksDB` 和官方的 RocksDB 是二进制兼容的，您并不需要重新编译自己的应用层代码

**Note:** `官方的 RocksDB` 不同版本的动态库一般互不兼容，目前我们兼容的版本是 [release v5.9.2](https://github.com/facebook/rocksdb/releases/tag/v5.9.2)，只要使用的是这个官方版，就可以使用 TerarkDB 的动态库直接替换。

### 2.使用方法

参数的设置可以通过直接传递环境变量的方式，如下：

```bash
# If you have installed files in `terark-zip-rocksdb-XXX/lib` into
# system lib path(such as /usr/lib64), `LD_LIBRARY_PATH` is not needed.
export LD_LIBRARY_PATH=terark-zip-rocksdb-XXX/lib:$LD_LIBRARY_PATH

env LD_PRELOAD=libterark-zip-rocksdb-r.so \
    TerarkZipTable_localTempDir=/path/to/some/temp/dir \
    TerarkZipTable_indexNestLevel=2 \
    TerarkZipTable_smallTaskMemory=1G \
    TerarkZipTable_softZipWorkingMemLimit=16G \
    TerarkZipTable_hardZipWorkingMemLimit=32G \
    app_exe_file app_args...
```

这些环境变量中，只有 `LD_PRELOAD` 和 `TerarkZipTable_localTempDir` 是必须的，其他的环境变量可以留空使用默认值即可.
