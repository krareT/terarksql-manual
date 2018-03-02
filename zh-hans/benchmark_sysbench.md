## 简介
sysbench 是一个模块化的、跨平台、多线程基准测试工具,主要用于评估测试各种不同系统参数下的数据库负载情况。本测试使用 sysbench 分别向官方原版 MySQL 和 MySQL on TerarkDB 导入 **450,000,000** 条数据，测试在不同内存下两者的读写性能。

测试程序使用 [sysbench 1.1.0](https://github.com/akopytov/sysbench)

## 测试平台

- CPU: Intel(R) Xeon(R) CPU E5-2630 v3 @ 2.40GHz x2 （共16核32线程）
- 内存: DDR4 16G @ 1866 MHz x 12 （共 192 G）
- SSD: INTEL SSDSC2BP48 0420 IOPS 89000
- 操作系统: CentOS 7

测试中使用的官方原版 MySQL 版本为 Ver 5.6.35 for linux-glibc2.5 on x86_64，后记为 innodb（MySQL on TerarkDB 记为 terarkdb）。

## 导入

