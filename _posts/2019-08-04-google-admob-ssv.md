---
layout: post
title: PHP版本Google广告admob服务端回调验证SSV
tags: google admob ssv
categories: php
---
<style type="text/css">
    p{text-indent: 20px}
</style>

<p>
    因业务需要接入Google的激励广告，涉及Google回调的服务器端验证 (SSV) server side verifiy。Python版本的基于第三方包ecdsa开箱即用，PHP版本也有一个ecdsa库，但是过于复杂，想到之前做支付宝支付，google支付的openssl rsa密钥签名校验。还是自己来写个简单实用的。
</p>

### Google公钥的地址：
<https://www.gstatic.com/admob/reward/verifier-keys.json>
> 注意： 
> >> 1. AdMob 密钥服务器提供的公钥会不定期轮换。为确保可以继续按预期验证 SSV 回调，请勿使公钥的缓存时间超过 24 小时。
> >> 2. Google 预计您的服务器会针对 SSV 回调返回 HTTP 200 OK 成功状态响应代码。如果您的服务器无法访问或未提供预期的响应，Google 将重新尝试发送 SSV 回调，每隔 1 秒发送最多 5 次。
> >> 3. 用回调参数中key_id 取对应公钥，进行签名验证。

获取公钥可以使用curl 或 file_get_contents 函数，推荐使用curl。
这里就不再写获取公钥的代码了，直接copy过来使用。

完整代码如下：
```php
// Google admob 公钥
$verifier_keys = '{"keys":[{"keyId":3335741209,"pem":"-----BEGIN PUBLIC KEY-----\nMFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAE+nzvoGqvDeB9+SzE6igTl7TyK4JB\nbglwir9oTcQta8NuG26ZpZFxt+F2NDk7asTE6/2Yc8i1ATcGIqtuS5hv0Q==\n-----END PUBLIC KEY-----","base64":"MFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAE+nzvoGqvDeB9+SzE6igTl7TyK4JBbglwir9oTcQta8NuG26ZpZFxt+F2NDk7asTE6/2Yc8i1ATcGIqtuS5hv0Q=="}]}';

// Google 回调参数字符串，get传参
$query_string = '';

// json 格式公钥字符串，转数组，根据回调参数query_string中的key_id 取对应的公钥验证签名
$verifier_keys_arr = json_decode($verifier_keys, true);
if(empty($verifier_keys_arr) || !is_array($verifier_keys_arr)){
    throw new Exception("wrong google public keys!");
}
// 公钥的两种格式，pem 和 base64 
$publicKey_pem = $verifier_keys_arr['keys'][0]['pem'];
$publicKey_base64 = $verifier_keys_arr['keys'][0]['base64'];
// base64 的格式化
$publicKeyString = "-----BEGIN PUBLIC KEY-----\n" . wordwrap($publicKey_base64, 64, "\n", true) . "\n-----END PUBLIC KEY-----";
// pem转公钥资源对象
$publicKey = openssl_pkey_get_public($publicKeyString);
// 注： publicKey_pem, publicKeyString, publicKey 都是可以正常签名的

// 解析回调参数
parse_str($query_string, $query_arr);
// 签名结果字符串
$signature = trim($query_arr['signature']);
// 重要的是这里的签名结果字符串的替换 和 补位
$signature = str_replace(['-', '_'], ['+', '/'], $signature);
$signature .= '===';

// 进行签名的数据元字符串
$message = substr($query_string, 0, strpos($query_string, 'signature')-1);

$return = [
    'code' => 0,
    'message' => 'error'
];

//验证签名, 这里使用 $publicKey，$publicKey_pem, $publicKeyString 都是可以的
$success = openssl_verify($message, base64_decode($signature), $publicKey, OPENSSL_ALGO_SHA256);
if ($success === -1) {
    $return['message'] = '111111'.openssl_error_string();
} elseif ($success === 1) {
    $return['code'] = 1;
    $return['message'] = 'success';
} else {
    $return['message'] = '222222'.openssl_error_string();
}

var_dump($return);
```
###　执行php脚本:
```shell
$ php -f admob_ssv.php
array(2) {
  'code' =>
  int(1)
  'message' =>
  string(7) "success"
}

```
success 校验成功。

附：
composer包：composer require depakin/admobssv  
github地址：<https://github.com/yisangwu/google_admob_ssv>