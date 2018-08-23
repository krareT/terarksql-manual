## 安装部署

### 1.环境要求
Linux 操作系统

### 2.下载安装包
[填写表单](http://terark.com/zh/download/mysql_rocksdb/latest)即可获得最新的下载链接
- 目前我们仅提供 Linux 环境下，64 位操作系统的安装版本
- 文件名中的 `bmi2-0` 表示不支持 `bmi2` CPU 指令，`bmi2-1` 表示支持 `bmi2` CPU 指令
  - 使用`cat /proc/cpuinfo  | grep bmi2` 可以查看 CPU 是否支持 `bmi2`

### 3.安装
目前我们提供针对不同平台打包好的压缩包，您可以在下载后解压到任意目录即可直接使用

### 4.启动

在启动之前，请修改 MySQL 目录下的 `my.cnf` 配置文件，将以下几个参数，修改为你的 MySQL 安装路径或者其他对应的路径：

```
basedir : MySQL 的安装路径
datadir : 数据文件存储路径
log-bin : log 存储路径
pid-file : 进程 pid 文件存放位置
```
运行安装目录下的 `start.sh` 即可。
