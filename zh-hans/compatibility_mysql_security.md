
涉及到安全性测试表现异常的 case，基本都与编译使用的 ssl 版本相关。这些 case 分布在原生 MySQL 不同的 suite 中，摘录如下：

### 1. main 涉及到的

```
Failing test(s):
    main.ssl_8k_key
    main.ssl_crl
    main.openssl_1
    main.plugin_auth_sha256_tls
    main.ssl-session-reuse
    main.ssl
    main.ssl_compress
    main.mysql_shutdown_logging
    main.mysqld--help-notwin-profiling
    main.mysqlshow
    main.ssl_ca
```
共 11 个，按失败原因分类可分为以下几类

#### 1.1.1 SSL 加密算法与预期不同

编译当前测试使用的 MyRocks 时使用的 openssl 版本（1.0.1e）较新，其使用的加密算法与测试预期不同，故导致结果不一致。

涉及测试：
```
main.ssl_8k_key
main.ssl_crl
main.plugin_auth_sha256_tls
main.ssl-session-reuse
main.ssl
main.ssl_compress
main.ssl_ca
```
共 7 个。

错误信息类似如下：
```
SHOW STATUS LIKE 'ssl_Cipher'；
Variable_name	Value
-Ssl_cipher	ECDHE-RSA-AES128-GCM-SHA256
+Ssl_cipher	ECDHE-RSA-AES256-GCM-SHA384
```

#### 1.1.2 access denied

在测试过程中，测试程序不能连接到服务器，原因有以下两个。

##### 1.1.2.1 测试中登录使用的 SSL 加密算法与预期不同

涉及测试：
```
main.openssl_1
```
共 1 个。

原因同 1.1.1

测试中使用了如下语句：
```
grant select on test.* to ssl_user2@localhost require cipher "ECDHE-RSA-AES128-GCM-SHA256";
```

将其改为：
```
grant select on test.* to ssl_user2@localhost require cipher "ECDHE-RSA-AES256-GCM-SHA384";
```

便能正确的进行测试。同样因为以上原因，结果会与预期不一致，但这不会对功能有影响。

修改后的测试结果与预期结果差异类似如下：
```
 Variable_name	Value
-Ssl_cipher	ECDHE-RSA-AES128-GCM-SHA256
+Ssl_cipher	ECDHE-RSA-AES256-GCM-SHA384
