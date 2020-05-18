---
layout: post
title: php使用RSA私钥公钥签名校验
tags: php, rsa
categories: php
---
<style type="text/css">
    p{text-indent: 20px}
</style>

#### 生成 RSA 私钥
```shell
$ openssl genrsa -out rsa_private_key.pem 1024
```
#### 生成 RSA 公钥(php和java都用私钥生成公钥)
```shell
$ openssl rsa -in rsa_private_key.pem -pubout -out rsa_public_key.pem
```
####PHP 版本加密，认证：
```php
<?php
$privateKeyString=
<<<EOF
-----BEGIN RSA PRIVATE KEY-----
MIICXAIBAAKBgQDRHaNFj+BW4g6A+0NOrPq1XOACEsCZUuJNJsq6JbcvN9JgpPKG
C6zlvEwiibsAk5NqLvYmdbqiIC41VJ/T4aq7RFc/k5eo2FJlfzoRKMIG9yVsp6mr
NLOkrDrbbN+GS02463L3oDhlZVncXfXWZNbeC0vrBzjM3QsZIqHYVQQc6QIDAQAB
AoGAO6fA1C9fSGnkyVbktKUUQHjmTrEa0KKcfHX8j24C/C1ojrl/lk3uXPuCnLe9
6UQwYbJT9lTPkUCs7fnePou6MH4acPsOHnwJ3PpQrO5H01Z8sZQkD/KIyk1LPaGl
8zvKlTOgrSg4h8JLk7i9PI8q2Yp75rquVZglQ7m6H1GXNiECQQDszsnMjAptFvnb
bs29wiuRSV7iqkigZTokLtJjJ1qw5fpNipJbAN/jhwAFEe1aMAno/rpru0l3EuGG
Z85M+pfzAkEA4hBODEnRU5D1fwUcp4LY5spOm2dk5h/LZJbI/5et0y2Qah+YFe0k
hgzV70dNUmjteYg3axzksUuipZE4bSoqswJBALlFSCjaX8XdtfnyBNGzunZe2ven
lk63I/fvEfc1cQT5yQ0lnz/HvWK72k4dKn/nGbnKoXtr+hxJD10ilgsv+/UCQGnt
g/TkHg8PTMmxJoUjnek/AOh24WOnoFHJCfQiKdRbdGEV3tjfXw7lMtXFTmkAO86H
0pgBWPPu4g685njYmlsCQGHxbxNr2ybRqgIuqmiNO7jldtcXMqzaF4w1BHS5C0mk
A6+i/KEJ5yLVv6lmDPO1WDbbt81cYO8RGEQ7DblIRoc=
-----END RSA PRIVATE KEY-----
EOF;

// 私钥加密
$privateKey = openssl_pkey_get_private($privateKeyString);
// 签名字符串
$message = '123456';

$signature = null;
if (openssl_sign($message, $signature, $privateKey, OPENSSL_ALGO_SHA256)) {
    $signature = base64_encode($signature);
    echo 'signature:', $signature,PHP_EOL;
} else {
    echo openssl_error_string();
}

// 公钥验证签名
$publicKeyString = 
<<<EOF
-----BEGIN PUBLIC KEY-----
MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQDRHaNFj+BW4g6A+0NOrPq1XOAC
EsCZUuJNJsq6JbcvN9JgpPKGC6zlvEwiibsAk5NqLvYmdbqiIC41VJ/T4aq7RFc/
k5eo2FJlfzoRKMIG9yVsp6mrNLOkrDrbbN+GS02463L3oDhlZVncXfXWZNbeC0vr
BzjM3QsZIqHYVQQc6QIDAQAB
-----END PUBLIC KEY-----
EOF;

// 公钥的第二种格式，也是可以的
$publicKeyString = "-----BEGIN PUBLIC KEY-----\nMIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQDRHaNFj+BW4g6A+0NOrPq1XOAC\nEsCZUuJNJsq6JbcvN9JgpPKGC6zlvEwiibsAk5NqLvYmdbqiIC41VJ/T4aq7RFc/\nk5eo2FJlfzoRKMIG9yVsp6mrNLOkrDrbbN+GS02463L3oDhlZVncXfXWZNbeC0vr\nBzjM3QsZIqHYVQQc6QIDAQAB\n-----END PUBLIC KEY-----";

$publicKey = openssl_pkey_get_public($publicKeyString);

$return = [
    'code' => 0,
    'message' => 'error'
];

// 执行验证
$success = openssl_verify($message, base64_decode($signature), $publicKey, OPENSSL_ALGO_SHA256);
if ($success === -1) {
    $return['message'] = ''.openssl_error_string();
} elseif ($success === 1) {
    $return['code'] = 1;
    $return['message'] = 'success';
} else {
    $return['message'] = openssl_error_string();
}
echo 'openssl_verify result:';
var_dump($return);
```

#### 执行脚本，结果如下：
```shell
>php -f depakin.php
signature:q6nCWEEXHRubgZEFT3wkjVd5gmmDucBCAiX20HBrpSBxPevlxCTwkMZ+35nVpVoj+Rmz3m+5qWBZ2m0q8POoDFr5YPsANSos0cM1Nr1zC9ju6SRCBpRmiGKLxzniuehkrRyxbWf+rLthmiSDnQa/peWw5Y7hsVT68yR8AoCovRY=
openssl_verify result:
array(2) {
  'code' =>
  int(1)
  'message' =>
  string(7) "success"
}
```