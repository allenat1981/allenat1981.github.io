---
title: "Resetting Passwords 重設密碼"
date: 2025-04-29 17:25:00 +0800
categories: 
  - laravel
tags:
  - laravel
  - myshop
---

當使用者忘記密碼時，會發送重設密碼的連結，提供使用者重設密碼。

參考資料：[Resetting Passwords](https://laravel.com/docs/12.x/passwords)

## 流程概述

1. 使用者進入忘記密碼頁面，輸入註冊時的 email。
2. 系統產生一組 token 存入到資料表 password_reset_tokens(Laravel 內建)。
3. 系統發送包含 token 與使用者 email 的重設密碼通知信給使用者。
4. 使用者點擊連結後，連結到重設密碼頁面，並輸入新的密碼後送出。
5. 系統驗證使用者的 email, token, password，若驗證成功，則更新使用者密碼，並刪除存在資料庫的 token，同時更新使用者的 remember_token，讓其他裝置的"記住我"失效。

## 實作內容

### 建立 route

編輯 `routes/user.php`

```php
Route::middleware('guest:user')->group(function () {

    Route::get('/forgot-password', [ResetPasswordController::class, 'index'])
        ->name('password.request');

    Route::post('/forgot-password', [ResetPasswordController::class, 'notification'])
        ->name('password.email');
    
    Route::get('/reset-password/{token}', [ResetPasswordController::class, 'show'])
        ->name('password.reset');

    Route::post('/reset-password', [ResetPasswordController::class,'update'])
        ->name('password.update');
});
```

說明：

以群組的方式加入 middleware: guest:user，阻擋已登入的使用者進入此功能。以下為各路由的主要功能：

- `user.password.request` 忘記密碼的主要頁面，提供使用者表單輸入 email 帳號。
- `user.password.email` 接收 `user.password.request` 的表單內容，驗證使用者 email，若驗證成功，則產生重設密碼的 token 與 URL，並寄送重設密碼通知信給使用者。
- `user.password.reset` 重設密碼頁面，提供使用者表單輸入新密碼。
- `user.password.update` 接收 `user.password.reset` 的表單內容，驗證成功後，更新使用者密碼。

### 建立 Controller

```bash
php artisan make:controller User/ResetPasswordController
```

編輯 `app/Http/Controllers/User/ResetPasswordController.php`

```php{linenos=true}
use App\Models\User;
use Illuminate\Auth\Events\PasswordReset;
use Illuminate\Support\Facades\Hash;
use Illuminate\Support\Facades\Password;
use Illuminate\Support\Str;
use Illuminate\Validation\Rules\Password as PasswordRule;

class ResetPasswordController extends Controller
{
    public function index()
    {
        return view("user.forgotPassword");
    }

    public function notification(Request $request)
    {
        $request->validate([
            'email' => 'required|email'
        ]);

        $status = Password::sendResetLink($request->only('email'));

        return $status === Password::ResetLinkSent
        ? back()->with(['success' => __($status)])
        : back()->withErrors(['email' => __($status)]);
    }

    public function show(Request $request, string $token)
    {
        return view('user.resetPassword', [
            'email' => $request->email,
            'token' => $token
        ]);
    }

    public function update(Request $request) 
    {

        $request->validate([
            'token' => ['required'],
            'email' => ['required', 'email'],
            'password' => [
                'required', 
                'confirmed', 
                PasswordRule::min(8)->letters()->numbers()
            ]
        ]);

        $status = Password::reset(
            $request->only('email', 'password', 'password_confirmation', 'token'),
            function (User $user, string $password) {
                $user->forceFill([
                    'password' => Hash::make($password)
                ])->setRememberToken(Str::random(60));

                $user->save();

                event(new PasswordReset($user));
            }
        );

        return $status === Password::PasswordReset 
            ? back()->with(['success'=> __($status) ])
            : back()->withErrors(['email'=> __($status)]);
    }
}
```

說明：

- `index()` 忘記密碼頁面，提供表單讓使用者輸入 email。
- `notification()` 接收忘記密碼表單。若驗證成功，調用 `Password::sendResetLink()` 寄送重設密碼通知信給使用者。
- `show()` 重設密碼頁面，提供表單讓使用者輸入新密碼。
- `update()` 接收重設密碼表單。若驗證成功，調用`Password::reset()` 重設使用者密碼。包含以下內容
  - 再次驗證使用者身份(email 與 token)與密碼。
  - 調用 `$user->forceFill()` 強制更新使用者密碼。
  - 調用 `$user->setRememberToken()` 更新 users.remember_token。
  - 調用 `event()` 發送 `PasswordReset` 事件。

### 建立 View

建立以下三個 blade 樣板

- user/forgotPassword.blade.php 忘記密碼表單
- user/resetPasswordNotification.blade.php 重設密碼通知信內容
- user/resetPassword.blade.php 重設密碼表單

#### forgotPassword.blade.php

```bash
php artisan make:view user/forgotPassword
```

編輯 `resources/views/user/forgotPassword.blade.php`

```php
<x-layout>
    <x-breadcrumbs :links="['忘記密碼' => '#']" />

    <form action="{{ route('user.password.email') }}" method="POST">
        @csrf
        <div class="mb-4">
            <x-form.label for="email" required="true">電子信箱</x-form.label>
            <x-form.text-input
                name="email" 
                placeholder="請輸入您註冊時的電子信箱" />
        </div>
        <x-form.button class="mb-4">送出</x-form.button>
    </form>
</x-layout>
```

說明：

此表單讓使用者輸入註冊的 email 信箱，並送出至 route: `user.password.email` 驗證並寄出重設密碼通知信。

#### resetPasswordNotification.blade.php

```bash
php artisan make:view user/resetPasswordNotification
```

編輯 `resources/views/user/resetPasswordNotification.blade.php`

```php
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>驗證您的 Email</title>
    <style>
        body { font-family: Arial, sans-serif; line-height: 1.6; }
        .button {
            display: inline-block;
            padding: 10px 20px;
            background-color: #1D4ED8;
            color: white;
            text-decoration: none;
            border-radius: 5px;
        }
        .container {
            max-width: 600px;
            margin: auto;
        }
    </style>
</head>
<body>
    <div class="container">
        <h2>您好！</h2>
        <p>請點擊以下連結進行密碼重設：</p>

        <p>
            <a href="{{ $url }}" class="button">重設密碼</a>
        </p>

        <p>若您並未請求重設密碼，請忽略此郵件。</p>

        <p>感謝您！<br>{{ config('app.name') }}</p>
    </div>
</body>
</html>
```

說明：

此頁面將作為 Notification 的郵件樣板。

#### resetPassword.blade.php

```bash
php artisan make:resetPassword
```

編輯 `resources/views/user/resetPassword.blade.php`

```php
<x-layout>
    <x-breadcrumbs :links="['重設密碼' => '#']"></x-breadcrumbs>

    <form action="{{ route('user.password.update') }}" method="POST">
        @csrf
        <div class="mb-4">
            <x-form.label for="password" required>密碼</x-form.label>
            <x-form.text-input type="password" name="password" />
        </div>
        <div class="mb-4">
            <x-form.label for="password_confirmation" required>確認密碼</x-form.label>
            <x-form.text-input type="password" name="password_confirmation" placeholder="請再輸入一次密碼" />
        </div>
        <div>
            <x-form.text-input type="hidden" name="email" :value="old('email', $email ?? '')" />
            <x-form.text-input type="hidden" name="token" :value="$token" />
            <x-form.button type="submit">變更密碼</x-form.button>
        </div>        
    </form>
</x-layout>
```

說明：

提供密碼(password)和確認密碼(password_confirmation)欄位讓使用者重設密碼，並且用 hidden 方式加入 email 和 token 欄位，一併傳至 route: `user.password.update` 進行驗證與處理。

### 設定 Notification

要使用 Resetting Passwords 機制，必須先完成以下項目

Models/User 必須先完成以下實作

- 使用 trait `use Illuminate\Notifications\Notifiable`
- 實作 `Illuminate\Contracts\Auth\CanResetPassword`
- 使用 trait `Illuminate\Auth\Passwords\CanResetPassword`

**Laravel 12 內建的 Models/User 已完成以上實作。**

資料庫必須建立 `password_reset_tokens` 資料表。

請參考 Laravel 12 內建的 `database/migrations/0001_01_01_000000_create_users_table.php`。

#### 建立 UserResetPasswordNotification

```bash
php artisan make:notification UserResetPasswordNotification
```

編輯 `app/Notifications/UserResetPasswordNotification.php`

```php{linenos=true}
class UserResetPasswordNotification extends Notification
{

    public function __construct(
        public string $token
    )
    {

    }

    public function toMail(object $notifiable): MailMessage
    {
        $url = url(route('user.password.reset', [
            'token' => $this->token,
            'email' => $notifiable->getEmailForPasswordReset()
        ], false));

        return (new MailMessage)
            ->subject('重設您的密碼信')
            ->view(
                'user.resetPasswordNotification', 
                [
                    'url' => $url
                ]
            );
    }
}
```

說明：

- LINE 5 傳入認證用的 $token
- LINE 13 ~ 16 建立重設密碼的 URL
- LINE 18 ~ 用 view 的方式設定重設密碼通知信內容

#### 設定 UserResetPasswordNotification

編輯 `app/Models/User.php`

```php{linenos=true}
use App\Notifications\UserResetPasswordNotification;

class User extends Authenticatable implements MustVerifyEmail
{
    use HasFactory, Notifiable;

    public function sendPasswordResetNotification($token)
    {
        $this->notify(new UserResetPasswordNotification($token));
    }
}
```

說明：

- `sendPasswordResetNotification($token)` 覆寫此方法的內容，設定 notify 為 `UserResetPasswordNotification`。

## 結論

以上設定完成後，即可建立一個忘記密碼的連結 route: `user.password.request`，並測試重設密碼流程。
