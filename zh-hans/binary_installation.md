### 1. 下载

```
wget http://terark-downloads.oss-cn-qingdao.aliyuncs.com/mysql_rocksdb/r5.6.35/terarksql-Linux-x86_64-g%2B%2B-4.8-bmi2-0.tar.xz
```

### 2. 添加 mysql 用户

```
sudo groupadd mysql
sudo useradd -g mysql mysql
```

### 3. 初始化

```
tar Jxvf terarksql-Linux-x86_64-g++-4.8-bmi2-0.tar.xz
cd terarksql-4.8-bmi2-0/
sudo ./init.sh prepare /path/to/datadir
sudo ./init.sh init
```

### 4. 启动

```
sudo ./start.sh
```
注：数据库一定要通过 **start.sh** 来启动！


### 5. 参数说明

- **TerarkDB** 相关的参数是通过环境变量设置的，我们通常在 start.sh 里设置，完整的参数说明在[这里](http://terark.com/docs/terarksql-manual/zh-hans/full_config_options.html)
- **MyRocks** 相关的参数通过 my.cnf 配置文件设置，具体的说明在[这里](https://github.com/facebook/mysql-5.6/wiki/New-MySQL-RocksDB-Server-Variables)
