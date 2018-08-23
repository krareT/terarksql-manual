## start.sh

使用该脚本启动 mysqld 服务，启动前请确保 `my.cnf` 已经配置完好。

一般情况下，只需要修改 `TerarkZipTable_localTempDir=/some/temp/dir`，其它参数留空使用默认即可，具体说明请查看[所有配置](config_data_loading.md)：

```
#!/bin/bash

DIR="$(cd "$(dirname "$0")" && pwd)"

env TerarkZipTable_localTempDir=$DIR/terark-temp \
    TerarkZipTable_keyPrefixLen=4 \
    TerarkZipTable_offsetArrayBlockUnits=128 \
    TerarkZipTable_extendedConfigFile=$DIR/license \
    $DIR/support-files/mysql.server start --defaults-file=$DIR/my.cnf
```
