---
title: "PHP 陣列解包(unpacking)"
date: 2025-03-05 10:00:00 +0800
categories: 
  - php
tags:
  - PHP
---

PHP8 提供陣列 unpacking 運算符號 `...` 進行陣列解包。

## 運用 unpacking 符號合併陣列

若需要將 2 個陣列進行合併，則可如下運用

```php
// 一般陣列合併

$arrayA = [
    'apple',
    'banana'
];

$arrayB = [
    'car',
    'dog'
];

$mergedArray = [...$arrayA, ...$arrayB];
// $mrgedArray 內容如下
// ['apple', 'banana', 'car', 'dog']


// associative array(key => value 類型)

$arrayA = [
    'a' => 'apple',
    'b' => 'banana',
];

$arrayB = [
    'c' => 'car',
    'd' => 'dog'
];

$mergedArray = [...$arrayA, ...$arrayB];
// $mergedArray 內容如下
// ['a' => 'apple', 'b' => 'banana', 'c' => 'car', 'd' => 'dog']

```
