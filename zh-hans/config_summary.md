## 配置参数说明

### 1.简介
MySQl on TerarkDB 通过环境变量获取相应的配置信息，同时支持 TerarkDB 存储引擎的配置参数以及 MyRocks 的配置参数

由于 `Terark RocksDB` 和官方的 RocksDB 是二进制兼容的，您并不需要重新编译自己的应用层代码

**Note:** `官方的 RocksDB` 不同版本有可能互不兼容，目前我们经过测试兼容 [rocksdb 5.3.fb](https://github.com/facebook/rocksdb/tree/5.3.fb)

### 2.使用方法

参数的设置可以通过直接传递环境变量的方式，如下：

```bash
# If you have installed files in `terark-zip-rocksdb-XXX/lib` into
# system lib path(such as /usr/lib64), `LD_LIBRARY_PATH` is not needed.
export LD_LIBRARY_PATH=terark-zip-rocksdb-XXX/lib:$LD_LIBRARY_PATH

env LD_PRELOAD=libterark-zip-rocksdb-r.so \
    TerarkZipTable_localTempDir=/path/to/some/temp/dir \
    TerarkZipTable_indexNestLevel=2 \
    TerarkZipTable_indexCacheRatio=0.005 \
    TerarkZipTable_smallTaskMemory=1G \
    TerarkZipTable_softZipWorkingMemLimit=16G \
    TerarkZipTable_hardZipWorkingMemLimit=32G \
    app_exe_file app_args...
```

这些环境变量中，只有 `LD_PRELOAD` 和 `TerarkZipTable_localTempDir` 是必须的，其他的环境变量可以留空使用默认值即可.
