### 1.下载安装包
[填写表单](http://terark.com/zh/download/mysql_rocksdb/latest)即可获得最新的下载链接

下载后解压到任意目录即可直接使用

### 2.启动

#### 2.1. 添加 mysql 用户

```
sudo groupadd mysql
sudo useradd -g mysql mysql
```

#### 2.2. 初始化

```
tar Jxvf terarksql-Linux-x86_64-g++-4.8-bmi2-0.tar.xz
cd terarksql-4.8-bmi2-0/
sudo ./init.sh prepare /path/to/datadir
sudo ./init.sh init
```

#### 2.3. 启动

```
sudo ./start.sh
```
注：数据库一定要通过 **start.sh** 来启动！


#### 2.4. 参数说明

- **TerarkDB** 相关的参数是通过环境变量设置的，我们通常在 start.sh 里设置，完整的参数说明在[这里](http://terark.com/docs/terarksql-manual/zh-hans/full_config_options.html)
- **MyRocks** 相关的参数通过 my.cnf 配置文件设置，具体的说明在[这里](https://github.com/facebook/mysql-5.6/wiki/New-MySQL-RocksDB-Server-Variables)
