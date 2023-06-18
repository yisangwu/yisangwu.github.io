---
layout: post
title: Request请求Content-Type对服务端数据接收的影响
tags: php, request, post, input
categories: php
---
<style type="text/css">
    p{text-indent: 20px}
</style>
<p>
    问题出现场景： 研发同学反馈，对接第三方平台接口（python），可以对方正常收到我们的传参，但是我们却收不到他们的数据，php接口获取$_REQUEST，$_POST 都是空的。
</p>
<p>
    初步判断是header的问题。在接口加上log，打印 $_SERVER 和 file_get_contents( 'php://input'), 观察后，果然对方header中是Content-Type: application/json。
    使用file_get_contents( 'php://input')获取数据，问题解决。
</p>
<p>
    鉴于此，模拟整理了下，Content-Type不同，对php服务端接收数据的影响。
    >> tips: 设置php.ini中的always_populate_raw_post_data值为On。PHP才会总把POST数据填入变量$HTTP_RAW_POST_DATA，否则$HTTP_RAW_POST_DATA为NULL
</p>
#### 1. Content-Type: application/x-www-form-urlencoded

```shell
curl -X POST -H 'Content-Type: application/x-www-form-urlencoded' -i 'http://127.0.0.1/api.php' --data 'a=1&b=2'
```

```php

// php://input
string(7) "a=1&b=2"

// $_REQUEST
array(2) {
  ["a"]=>
  string(1) "1"
  ["b"]=>
  string(1) "2"
}

// $_POST
array(2) {
  ["a"]=>
  string(1) "1"
  ["b"]=>
  string(1) "2"
}

// $_GET
array(0) {
}

```

#### 2. Content-Type: application/json

```shell
curl -X POST -H 'Content-Type: application/json' -i 'http://127.0.0.1/api.php' --data 'a=1&b=2'
```

```php

// php://input
string(7) "a=1&b=2"

// $_REQUEST
array(0) {
}

// $_POST
array(0) {
}

// $_GET
array(0) {
}

```


#### 3. Content-Type: application/x-www-data-urlencoded

```shell
curl -X POST -H 'Content-Type: application/x-www-data-urlencoded' -i 'http://127.0.0.1/api.php' --data 'a=1&b=2'
```

```php

// php://input
string(7) "a=1&b=2"

// $_REQUEST
array(0) {
}

// $_POST
array(0) {
}

// $_GET
array(0) {
}

```