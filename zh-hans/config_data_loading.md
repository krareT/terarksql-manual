## 启动脚本说明 - 数据导入

在安装包的根目录，通过 `start_for_bulk_load.sh` 可以以灌数据的模式启动服务：

```
#!/bin/bash

DIR="$(cd $(dirname $0);pwd)"

MemTable=8G # **Large, for bulk load only**

env TerarkZipTable_localTempDir=/ssd/mysql/terark-temp \
    TerarkZipTable_keyPrefixLen=4 \
    TerarkZipTable_offsetArrayBlockUnits=128 \
    TerarkZipTable_base_background_compactions=1 \
    TerarkZipTable_max_background_compactions=1 \
    TerarkZipTable_max_subcompactions=4 \
    TerarkZipTable_min_merge_width=1000 \
    TerarkZipTable_max_merge_width=2000 \
    TerarkZipTable_level0_file_num_compaction_trigger=1000 \
    TerarkZipTable_softZipWorkingMemLimit=32G \
    TerarkZipTable_hardZipWorkingMemLimit=48G \
    TerarkZipTable_write_buffer_size=$MemTable \
    TerarkZipTable_target_file_size_base=48G \
    TerarkZipTable_indexCacheRatio=0.001 \
    TerarkZipTable_sampleRatio=0.015 \
    TerarkZipTable_extendedConfigFile=$DIR/license \
    $DIR/support-files/mysql.server start --defaults-file=$DIR/my.cnf
```

### 说明

该配置默认是依据 96GB 物理内存的基础来配置的，如果您的内存很小，可以根据[完整配置](full_config_options.md)中的配置项说明，修改配置项目

以上配置和您的数据、负载是强相关的，所以您可以多次尝试获得一个理想配置
