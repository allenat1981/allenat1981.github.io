---
title: "MyShop Email Verification 信箱驗證"
date: 2025-04-25 14:30:00 +0800
draft: false
categories: 
  - laravel
tags:
  - laravel
  - myshop
---

網站會員註冊後，可以建立一個信箱認證的機制與流程，確保會員註冊的信箱有效。

## 概述

流程如下：

- 建立 Middleware: EnsureEmailIsVerified 用來驗證會員已通過信箱認證。
- 將受保護的 route 加上 EnsureEmailIsVerified。
- 使用者註冊完成後，寄送認證信到使用者 email。
- 使用者點擊 email 的認證連結，通過認證後即可造訪受保護的 route。

## 前置作業

使用者的資料表必須有 `email` 以及 `email_verified_at`(timestamp, nullable) 欄位。

## 建制流程

1. 建立 Middleware: EnsureEmailIsVerified 用來保護 route。
2. 建立 Notification: UserVerifiyEmail 用來設定認證信通知。
3. 設定與使用者信箱認證機制相關的 route。

### Middleware:EnsureEmailIsVerified

```bash
php artisan make:middleware EnsureEmailIsVerified
```

編輯 `app/Http/Middleware/EnsureEmailIsVerified.php`

```php{linenos=true}
use Illuminate\Contracts\Auth\MustVerifyEmail;

class EnsureEmailIsVerified
{
    public function handle(Request $request, Closure $next, ?string $guard = null): Response
    {
        $user = $request->user($guard) ?? null;

        if (
            $user != null &&
            $user instanceof MustVerifyEmail &&
            $user->hasVerifiedEmail()
        ) {
            return $next($request);
        }

        return redirect()->route("{$guard}.verification.notice");
    }
}

```

說明：

- `handle()` 加入自訂參數 `?string guard = null`，可在 route 或 controller 加入 middleware，傳入參數。在此參數 `guard` 是用來指定使用那一個 Authentication Guard(user 或 admin)。
