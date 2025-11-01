---
title: "Authentication"
date: 2025-11-01 13:00:00 +0800
categories: 
  - laravel
tags:
  - laravel
---

官方網站文件摘錄

[Authentication](https://laravel.com/docs/12.x/authentication)

## Introduction

Guards 定義 users 在每次請求(request)中如何被驗證。例如 Laravel 自帶 `session` guard，使用 session storage 和 cookie 來維護狀態。

Providers 定義如何從 persistent storage 取回 users 資料。Laravel 自帶使用 Eloquent 和 database query builder 取得 users。你也可以自訂 providers。

authentication 設定檔位於 `config/auth.php`。

{{< alert type="notice" >}}
`guards` 與 `providers` 應避免與 `roles` 和 `permissions` 混淆。
{{< /alert >}}

### Database Considerations

Lpppp

若要建立可以驗證的資料表作為 Provider 請注意

- 需要有 `password` 欄位，至少 60 字元。
- 需要有 `remember_token` 欄位，至少 100 字元。

### Ecosystem Overview

當 user 通過驗證後，瀏覽器會將 session ID 存在 cookie 當中，並透過 session ID 取得存在 session 中的 user 資料。

若是需要去存取必須通過認證的 API ，若不是使用瀏覽器，一般不會使用 cookie 來進行認證，此時會在 request 加上與使用者關聯的 token，利用 token 來進行認證。

#### Laravel's Built-in Browser Authentication Services

Laravel 透過 facades： `Auth` 和 `Session` 進行認證和 session 服務的存取。這些功能為由 broswer 啟動的 request 提供 cookie-based 的認證。基於 session 和 session cookie 的 user 認證。

### Laravel's API Authentication Services

`Passport` 與 `Sanctum` 是用來提供 API token 的套件。這些套件與 Laravel 內建的 cookie base 認證可以共存(並非互相排斥）。

API 認證套件聚焦在 API token 認證。cookie based 認證聚焦在基於使用瀏覽器的認證。許多 application 會同時採用此二種認證方式。

#### Passport

`Passport` 是 OAuth2 認證機制的 Provider。此套件適合需要較複雜的驗證機制之情境。大部分的 applicatoin 並不需要如此複雜的驗證機制。

#### Sanctum

`Sanctum` 可以同時處理來自 web browser 的 request 和 API token 驗證的情境。對於同時具有 Web 與 API，或 SPA 架構的情境，`Laravel Sanctum` 將是首選。

`Sanctum` 認證流程

1. 首先透過 Laravel 內建的機制，判斷 request 是否包含已通過認證的 Session Cookie。若已具有通過驗證的 Session Cookie，則視為通過認證。
2. 若沒有通過 Session Cookie 的認證，則檢查 request 是否包含 API Token(通常位於 HTTP Header 的 Authorization 欄位)，若存在有效的 API Token ，則 Sanctum 就用此 API Token 進行認證。

### Summary and Choosing Your Stack

根據不同的 application 型態，可考慮使用不同的認證機制

- Web 網站：內建的 session cookie 機制。
- API application：`Sanctum`(簡單 API Token 機制)　或 `Passport`(OAuth2 機制)。
- SPA application：`Sanctum` 機制(同時適用 session cookie 和 API Token)。
