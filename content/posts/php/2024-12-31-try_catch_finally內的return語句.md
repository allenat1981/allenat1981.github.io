---
title: "try catch finally 內的 return 語句釋疑"
date: 2024-12-31 12:00:00 +0800
categories: 
  - php
tags:
  - PHP
---

PHP 的 try catch finally 區塊內的 return 語句執行順序釋疑。

## 參考資料

[PHP try…catch…finally](https://www.phptutorial.net/php-oop/php-try-catch-finally/)

## finally 總是會在最後執行

若在函式 try catch finally 區塊中均設有 return 語句，則最後必定是會回傳 finally 區塊內的 return。

```php
function doSomething()
{
    try {
        echo "Do something in Try Block\n";
        return "Return in Try Block\n";
    } catch(Exception $ex) {
        echo "Do something in Catch Block\n";
        return "Return in Catch Block\n";
    } finally {
        echo "Do something in Finally Block\n";
        return "Return in Finally Block\n";
    }
    echo "Do something in Outer Block\n";
    return "Return in Outer Block\n";
}
```

以上範例總是會回傳 "Return in Finally Block"。

## 結論

根據參考資料：

>The result will be returned after the finally block is executed. Also, if the finally block has a return statement, the value from the finally block will be returned.

1. finally 區塊總是會在一組 try catch finally 的組合中最後執行。
2. 若 finally 區塊含有 return 語句，則最後總是執行該 finally 區塊的 return 語句。
3. 若一組 try catch finally 組合中，try 或 catch 區塊中含有 return 語句，則在任一路徑的 return 語句執行前，finally 區塊仍會先被執行。
