---
title: "初探 Rate Limiting"
date: 2025-03-03 16:30:00 +0800
categories: 
  - laravel
tags:
  - laravel
---

Rate Limiting 是用來限制資源的存取次數，例如 1 小時內僅可以對 /resource 存取 10 次。實現 RateLimiter 搭配 middleware 用來實現 Rate Limiting 機制。

本文件的範例來自 Udemy 課程

Master Laravel 11 & PHP: From Beginner to Advanced: \#4.56

{{< alert type="waring" >}}
課程 #4.56 影片是 laravel 10 的教學，laravel 11 有所異動，本文是根據 laravel 11 的做法進行摘要。
{{< /alert>}}

官方文件[Rate Limiting](https://laravel.com/docs/11.x/routing#rate-limiting)。

## Rate Limiting 概述

laravel 使用與 cache 同樣的機制來實現 Rate Limiting，需要注意以下幾點：

- Rate Limiting 會儲存在 Cache 所設定的 Cache Storage。
- 使用 `RateLimiter::for('[KeyName]', ...)` 其中 KeyName 是用來作為 Rate Limiting 的主要索引鍵。使用相同 KeyName 的 route 會計算在同一個 rate limiting 內。(詳見以下範例說明)

## 加入 RateLimiter

編輯 `app/Providers/AppServiceProvider.php`  
在 `boot()` 加入 `RateLimiter` 設定

```php
use Illuminate\Support\Facades\RateLimiter;
use Illuminate\Cache\RateLimiting\Limit;
use Illuminate\Http\Request;

public function boot(): void
{
    RateLimiter::for('reviews', function (Request $request) {
        return Limit::perHour(3)->by($request->user()?->id ?: $request->ip());
});
}
```

說明

- `RateLimiter::for('reviews', ... )` 加入一個名稱為 "reviews"(名稱可自訂) 的 RateLimiter middleware。
- `return Limit::perHour...` 回傳一個 Rate Limiting 條件，在此針對套用 laravel 權限機制的使用者 id 或者是來源 IP 進行 Rate ，並且設定每小時只能存取 3 次。

### 在 route 加入 middleware

若要針對特定的 route action 進行速率限制，則使用 `middlewareFor()` 進行設定

```php
Route::resource('books.reviews', ReviewController::class)
    ->scoped(['review' => 'book'])
    ->only(['create', 'store'])
    ->middlewareFor(['store'], ['throttle:reviews']);


//若要針對指定的 route 進行 Rate Limiting，則可直接在 route 設定 middleware
Route::get('xyz', [XyzController::class, 'index'])
    ->middleware('throttole:reviews');
```

說明：

在 Route::resource 設定 Rate Limiting 機制的語句為 `middlewareFor(['store'], ['throttle:reviews])`。  
以上範例代表針對 ReviewController::class 的 store 進行上一個步驟加入的 RateLimiter `reviews`。

{{< alert type="warning" >}}
`throttle:reviews` 代表使用 KeyName 為 reviews 的 rate limiting。  
以上範例而言，代表在一個小時內，ReviewController::store() 和 XyzController::index() 的存取次數，二者加起來不能超過 3 次。
{{< /alert >}}
