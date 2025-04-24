---
title: "初探 Middleware"
date: 2025-04-02 12:00:00 +0800
categories: 
  - laravel
tags:
  - laravel
---

使用 Middleware 機制，可以在 Request 進入到 Controller Action Method 之前，進行自訂的預先作業，例如：驗證使用者權限、寫入日誌等等。

本文件的範例來自 Udemy 課程

Master Laravel 11 & PHP: From Beginner to Advanced: \#5.139

## 建立 Custom Middleware

使用 `artisan make:middleware` 指令建立 Middleware 類別

```bash
php artisan make:middleware Employer
```

將會生成 `app/Http/Middleware/Employer.php`。接著編輯 Middleware 的內容：

```php{linenos=true}
class Employer 
{
    public function handle(Request $request, Closure $next): Response
    {
        if (null === $request->user() || null === $request->user()->employer) {
            return redirect()->route('employer.create')
                ->with('error', 'You need to register as an employer first!');
        }
        return $next($request);
    }
}

```

 說明：

 LINE 5 ~ 8 用來驗證使用者是否具有權限，在此範例中，使用者必須有 employer 身分。若驗證不通過，則直接轉址並傳送相關錯誤訊息。

## 在 app 註冊 Middleware

 編輯 `bootstrap/app.php`，找到 `withMiddleware()` 區塊，調用 `$middleware->alias()` 加入自訂的 Middleware

 ```php{linenos=true}

    ->withMiddleware(function (Middleware $middleware) {
        $middleware->alias([
            'employer' => \App\Http\Middleware\Employer::class,
        ]);
    })

 ```

## 在 Route 設定 Middleware

設定 Route 時，可透過以下方法設定 Middleware 群組或替單一 Route 進行設定

```php{linenos=true}
Route::middleware('auth')->group(function() {
    Route::middleware('employer')
        ->resource('my-jobs', MyJobController::class);
});
```

說明：

- `Route::middleware('auth')->group(...)` 用來替所有 group 內的 Route 設定 `Middleware:auth`。
- `Route::middleware('employer')->resource(...)` 則替 resource route 設定 `Middelware:employer`。
- 根據以上設定，Request 會先進入 `Middleware:auth`，接著再進入 `Middleware:employer`，最後才進入 Controller Action Method。

更多 Middleware 請參考 [Middleware](https://laravel.com/docs/12.x/middleware)。
