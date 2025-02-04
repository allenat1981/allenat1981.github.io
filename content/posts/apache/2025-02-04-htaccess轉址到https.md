---
title: "使用 .htaccess 轉址到 https"
date: 2025-02-04 16:20:00 +0800
categories: 
  - apache
tags:
  - apache
---

使用 .htaccess 將 http 強制轉址到 https。

## 參考資料

[Apache Rewrite with Htaccess 理解與技巧](https://medium.com/%E6%B5%A6%E5%B3%B6%E5%A4%AA%E9%83%8E%E7%9A%84%E6%B0%B4%E6%97%8F%E7%BC%B8/htaccess-with-rewrite-3dba066aff11)

## 使用htaccess將http轉址到https

```conf
RewriteCond %{SERVER_PORT} ^80$
RewriteRule ^(.*)$ https://%{SERVER_NAME}%{REQUEST_URI} [L,R]
```
