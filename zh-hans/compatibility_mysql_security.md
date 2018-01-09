
涉及到安全性测试表现异常的 case，基本都与编译使用的 ssl 版本相关。这些 case 分布在原生 MySQL 不同的 suite 中，共 10 个，摘录如下：

### main

1. SSL 加密算法与预期不同

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

错误信息类似如下：
```
SHOW STATUS LIKE 'ssl_Cipher'；
Variable_name	Value
-Ssl_cipher	ECDHE-RSA-AES128-GCM-SHA256
+Ssl_cipher	ECDHE-RSA-AES256-GCM-SHA384
```

2. 测试中登录使用的 SSL 加密算法与预期不同

涉及测试：
```
main.openssl_1
```

测试中使用了如下语句：
```
grant select on test.* to ssl_user2@localhost require cipher "ECDHE-RSA-AES128-GCM-SHA256";
```

将其改为：
```
grant select on test.* to ssl_user2@localhost require cipher "ECDHE-RSA-AES256-GCM-SHA384";
```

便能正确的进行测试。同样因为以上原因，结果会与预期不一致，但这不会对功能有影响。

### rpl

```
rpl.rpl_ssl
```
错误信息类似如下：
```
-Master_SSL_Actual_Cipher = 'ECDHE-RSA-AES128-GCM-SHA256'
+Master_SSL_Actual_Cipher = 'ECDHE-RSA-AES256-GCM-SHA384'
```

### auth_sec

```
auth_sec.cert_verify
```
错误信息类似如下：
```
-Ssl_version    TLS_VERSION
+Ssl_version    TLS_VERSION.2
```



