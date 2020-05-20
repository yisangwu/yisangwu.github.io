---
layout: post
title: php 使用array_walk生成新的数组
tags: php, array
categories: php
---

# **array_walk(**_array_**,**_myfunction_**,**_userdata_**...)**

> **array_walk() 函数对数组中的每个元素应用回调函数。如果成功则返回 TRUE，否则返回 FALSE。**
> 
> **典型情况下 ** _myfunction_ ** 接受两个参数。**
> 
> _array_ ** 参数的值作为第一个，键名作为第二个。**
> 
> **如果提供了可选参数 ** _userdata_ ** ，将被作为第三个参数传递给回调函数。**

```php
$a = ['a' => 'red', 'b' => 'green', 'c' => 'blue'];
$data = [];

$myfunction = function ($value, $key) use (&$data) {
$data[] = sprintf('"The key %s has the value %s', $key, $value);
};

array_walk($a, $myfunction);
var_dump($data);
```

执行结果，$data 输出如下：

```bash
$ php -f depakin.php
array(3) {
  [0]=>
  string(27) "The key a has the value red"
  [1]=>
  string(29) "The key b has the value green"
  [2]=>
  string(28) "The key c has the value blue"
}
```