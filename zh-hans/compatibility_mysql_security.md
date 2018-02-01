
涉及到安全性测试表现异常的 case，基本都与编译使用的 ssl 版本相关：

### main suite 下的异常 case

1. SSL 加密算法与预期不同

编译当前测试使用的 MyRocks 时使用的 openssl 版本（1.0.1e）较新，其使用的加密算法与测试预期不同，故导致结果不一致。

```
main.ssl_8k_key
main.ssl_crl
main.plugin_auth_sha256_tls
main.ssl-session-reuse
main.ssl
main.ssl_compress
main.ssl_ca
```

错误信息如下：
```
SHOW STATUS LIKE 'ssl_Cipher'；
Variable_name	Value
-Ssl_cipher	ECDHE-RSA-AES128-GCM-SHA256
+Ssl_cipher	ECDHE-RSA-AES256-GCM-SHA384
```

2. 测试中登录使用的 SSL 加密算法与预期不同

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

3. 其他

```
main.mysql_shutdown_logging
```
失败对应的消息是我们将该 case 归入本类的原因。具体原因待查，

```
mysqltest: At line 24: query 'connect  con2,localhost,ssl_user2,,,,,SSL' failed: 1045: Access denied for user 'ssl_user2'@'localhost' (using password: NO)
```

### rpl suite 下的异常 case

```
rpl.rpl_ssl
```
错误信息如下：
```
-Master_SSL_Actual_Cipher = 'ECDHE-RSA-AES128-GCM-SHA256'
+Master_SSL_Actual_Cipher = 'ECDHE-RSA-AES256-GCM-SHA384'
```

### auth_sec suite 下的异常 case

1. SSL 库版本不同
```
auth_sec.cert_verify
```
错误信息类似如下：
```
-Ssl_version    TLS_VERSION
+Ssl_version    TLS_VERSION.2
```

2. 需要 YaSSL 支持
```
auth_sec.server_withoutssl_client_withssl
auth_sec.server_withoutssl_client_withoutssl
```


(DONE)


