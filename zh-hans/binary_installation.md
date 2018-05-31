### 1. 下载

```
wget http://terark-downloads.oss-cn-qingdao.aliyuncs.com/mysql_rocksdb/r5.6.35/mysql-on-terarkdb-Linux-x86_64-g%2B%2B-4.8-bmi2-0.tar.xz
```

### 2. 添加 mysql 用户

```
sudo groupadd mysql
sudo useradd -g mysql mysql
```

### 3. 初始化

```
tar Jxvf mysql-on-terarkdb-Linux-x86_64-g++-4.8-bmi2-0.tar.xz
cd mysql-on-terarkdb-4.8-bmi2-0
sudo ./init.sh prepare /path/to/datadir
sudo ./init.sh init
```

### 4. 启动

```
sudo ./start.sh
```
