---
title: "Email Verification 自訂信箱認證"
date: 2025-04-24 09:25:00 +0800
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

- LINE 5 `handle()` 加入自訂參數 `?string guard = null`，可在 route 或 controller 加入 middleware，傳入參數。在此參數 `guard` 是用來指定使用那一個 Authentication Guard(user 或 admin)。
- LINE 7 調用 `$request->user($guard)` 取得造訪的 user 資料。
- LINE 9 ~ 15 若通過驗證項目，則進入下個 action。
- LINE 17 若未通過認證，則轉址到 `route("{$guard}.verification.notice")`。

{{< alert type="info" >}}
若使用 Laravel 內建的 middleware `verified`，在認證失敗時，會轉指到 `verification.notice`。  
因為在 myshop 中，我們希望能夠在前綴為 user.* 的 route 結構進行所有的路由設定，所以使用自訂的 Middleware:EnsureEmailIsVarified 來完成。
{{< /alert >}}

### 將 EnsureEmailIsVerified 加入到 middleware

接著將 EnsureEmailIsVerified 加入到 middleware，並取別名為 `customVerified`。編輯 `bootstrap/app.php`:

```php
->withMiddleware(function (Middleware $middleware) {
  $middleware->alias([
    'customVerified' => EnsureEmailIsVerified::class,
  ]);
}
```

### 替 route 加上 middleware

接下來替要保護的 route 加上 `customVerified`，在 myshop 專案中是在 UserController 設定 auth 相關的 middleware，信箱認證須搭配 `auth`，故編輯 `app/Http/Controllers/User/UserController.php` 如下：

```php{linenos=true}
use Illuminate\Routing\Controllers\HasMiddleware;
use Illuminate\Routing\Controllers\Middleware;

class UserController extends Controller implements HasMiddleware
{

    public static function middleware()
    {
        return [
            new Middleware(['auth:user', 'customVerified:user'])->except(['create', 'store']),
            new Middleware('guest:user')->only(['create', 'store'])
        ];
    }
}
```

說明：

- LINE 4 替 UserController 加上 implememts HasMiddleware。
- LINE 10 替 UserController 加上 middleware `auth:user` 和 `customVerified:user`，參數傳入同樣的 auth guard: user。

設定完成後，進入 route，若沒有通過認證，即會被轉址到 `user.verification.notice`。

## Notification:UserVerifyEmail

建立 UserVerifiyEmail 類別來發送認證信通知

```bash
php artisan make:notification UserVerifyEmail
```

編輯 `app/Notifications/UserVerifyEmail.php`

```php{{linenos=true}}
use Illuminate\Notifications\Messages\MailMessage;
use Illuminate\Notifications\Notification;
use Illuminate\Support\Carbon;
use Illuminate\Support\Facades\URL;

class UserVerifyEmail extends Notification
{
    public function via(object $notifiable): array
    {
        return ['mail'];
    }

    public function toMail(object $notifiable): MailMessage
    {
        $verifyUrl = $this->verificationUrl($notifiable);

        return (new MailMessage)
            ->subject('請驗證您的會員信箱')
            ->view('user.verify', ['url' => $verifyUrl]);
    }

    private function verificationUrl($notifiable): string
    {
        $temporarySignedURL = URL::temporarySignedRoute(
            'user.verification.verify',
            Carbon::now()->addMinutes(config('auth.verification.expire', 60)),
            [
                'id' => $notifiable->getKey(),
                'hash' => sha1($notifiable->getEmailForVerification()),
            ]
        );
        return $temporarySignedURL;
    }
}

```

說明：

- LINE 15 調用 `verificationUrl()` 方法建立認證的 URL。
- LINE 17 ~ 19 設定認證信的主旨和內容，調用 `view()` 可以讀取 blade 樣板內容，並且同樣傳入參數。
- LINE 22 ~ 33 用 `URL::temporarySignedRoute()` 建立認證信箱 URL。

### Model User 設定 MustVerifyEmail

編輯需要認證信箱的 Model(本例為 `app/Models/User.php`)：

```php{linenos=true}
use App\Notifications\UserVerifyEmail;
use Illuminate\Contracts\Auth\MustVerifyEmail;
use Illuminate\Notifications\Notifiable;

class User extends Authenticatable implements MustVerifyEmail
{
  use HasFactory, Notifiable;

  public function sendEmailVerificationNotification()
  {
      $this->notify(new UserVerifyEmail());
  }
}
```

說明：

- LINE 5 將 User 加上 implements MustVerifyEmail
- LINE 7 User 加入 Notifiable
- LINE 9 ~ 12 覆寫 `sendEmailVerificationNotification()`，改為寄送自訂的 UserVerifyEmail。

## 設定認證的 route

編輯自訂的路由群組檔案 `routes/user.php`

```php
Route::get('/email/verify', [VerifyEmailController::class, 'index'])
    ->middleware('auth:user')
    ->name('verification.notice');

Route::get('/email/verify/{id}/{hash}', [VerifyEmailController::class,'verify'])
    ->middleware(['auth:user', 'signed'])
    ->name('verification.verify');

Route::post('/email/verifycation-notification', [VerifyEmailController::class, 'resendNotification'])
    ->middleware(['auth:user','throttle:6,1'])
    ->name('verification.send');
```

因為已經在 `bootstrap/app.php` 的 `withRouting()` 設定 `routes/user.php`，該群組會自動替 route 與 name 加上前綴 `user`，因此會有以下 route 名稱可供使用

- `user.verification.notice` 使用者為通過信箱認證，會轉到此路由，該路由的頁面內容為通知使用者須進行信箱認證。
- `user.verification.verify` 夾帶在使用者認證信中的連結，點擊此連結後，透過 Laravel 提供的內建功能驗證夾帶的 hash，若驗證成功則會更新使用者的 email_verified_at 欄位，代表已通過信箱認證。
- `user.verification.send` 若使用者未收到認證信，或認證信已到期，可 POST 到此路由重新取得認證信。

### Controller:VerifyEmailController

建立 VerifyEmailController 來處理信箱認證 route

```bash
php artisan make:controller User/VerifyEmailController
```

編輯 `app/Http/Controllers/User/VerifyEmailController.php`

```php{linenos=true}
use Illuminate\Foundation\Auth\EmailVerificationRequest;
use Illuminate\Http\Request;

class VerifyEmailController extends Controller
{

    public function index()
    {
        return view("user.verificationNotice");
    }

    public function verify(EmailVerificationRequest $request)
    {
        $request->fulfill();

        return redirect(route("user.home"));
    }

    public function resendNotification(Request $request)
    {
        $request->user('user')->sendEmailVerificationNotification();

        return back()->with('message', 'Verification link sent!');
    }
}

```

說明：

- LINE 7 ~ 10 `index()` 是 route: `user.verification.notice` 的 action method，該頁面實作的內容可提醒使用者尚未通過信箱驗證，並提供一個表單按鈕重寄通知信。
- LINE 12 ~ 17 `verify()` 是 route: `user.verification.verify` 的 action method，其中 `EmailVerificationRequest` 會驗證路由夾帶的 id 與 hash，若驗證成功，則透過 `$request->fulfill()` 更新使用者的認證狀態。
- LINE 19 ~ 24 `resendNotification()` 實作內容會調用目前使用者的`sendEmailVerificationNotification()`，重新寄送認證信。

### 實作 view

使用者未通過認證，會轉指到 `route("user.verification.notice")`，故該路由頁面可顯示基本的資訊和重寄驗證信的按鈕，基本可實作內容如下：

```bash
php artisan make:view user/verificationNotice
```

編輯 `resources/views/user/verificationNotice.php`

```php
<x-layout>
尚未驗證您的E-Mail
    <form action="{{ route('user.verification.send') }}" method="POST">
        @csrf
        <x-form.button type="submit">
            重寄驗證信
        </x-form.button>
    </form>
</x-layout>
```

## 補充：註冊完成後寄送認證信

信箱認證信通常會在使用者註冊完成時寄出，因此可在註冊的完成時調用 Laravel 內建的 `Registered` 事件，系統即會寄送通知信。

例如在註冊的功能

```php{linenos=true}
use Illuminate\Auth\Events\Registered;

class UserController extends Controller
{
  public function store(RegisterRequest $request)
  {
    $user = User::create($request->all());
      
    event(new Registered($user));

    return redirect()->route('user.auth.create')->with('success','會員註冊完成，請進行電子信箱驗證');
  }

```

說明：

- LINE 7 ~ 9 user 註冊完成代表在資料庫新增一筆 user 資料，接著即可透過 `event()` 方法調用 `Registered` 事件，讓系統寄出認證信。

## 結論

之所以會需要自訂信箱認證機制，包含以下幾個因素

1. 希望獨立不同角色的認證，例如 user 和 admin。
2. 根據不同角色，轉址會需要對應不同的 route group，但內建的 verified 無法調整轉址的 route，必須自行處理。
3. 想要自訂認證信的內容。
4. 需要更客制化的認證機制與流程。
