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

## Authentication Quickstart

### Retrieving the Authenticated User

若要取得已認證的 user 資料，可透過 facade: Auth 的 user 方法

```php{linenos=true}
use Illuminate\Support\Facades\Auth;
 
// Retrieve the currently authenticated user...
$user = Auth::user();
 
// Retrieve the currently authenticated user's ID...
$id = Auth::id();
```

在處理 request 的機制中(例如 Controller)，可以使用 Illuminate\Http\Request 的 instance 取得 user。

```php{linenos=true}
<?php
 
namespace App\Http\Controllers;
 
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;
 
class FlightController extends Controller
{
    /**
     * Update the flight information for an existing flight.
     */
    public function update(Request $request): RedirectResponse
    {
        $user = $request->user();
 
        // ...
 
        return redirect('/flights');
    }
}
```

若要確認目前的 HTTP request 是否已認證，可以使用 facade 的 `Auth::check()`，若已認證則回傳 `true`。

```php{linenos=true}
use Illuminate\Support\Facades\Auth;
 
if (Auth::check()) {
    // The user is logged in...
}
```

### Protecting Routes

`Route middleware` 可以用來保護路由，只允許已認證的使用者存取。

Laravel 內建名稱為 `auth` 的 middleware，該 middlware 是
`Illuminate\Auth\Middleware\Authenticat` 的別名。Laravel 預設已經加入該 middleware ，只須在需要受保護的 route 加上該 middleware 即可使用。

```php{linenos=true}
Route::get('/flights', function () {
    // Only authenticated users may access this route...
})->middleware('auth');

```

#### Redirecting Unauthenticated Users

當 `auth` 阻擋為認證的使用者，預設會將用戶導向名稱為 `login` 的路由。你可以在 `bootstrap/app.php` 改變此一行為。

```php{linenos=true}
// bootstrap/app.php

use Illuminate\Http\Request;
 
->withMiddleware(function (Middleware $middleware): void {
    $middleware->redirectGuestsTo('/login');
 
    // Using a closure...
    $middleware->redirectGuestsTo(fn (Request $request) => route('login'));
})

```

#### Redirecting Authenticated Users

相對於 middleware `auth`，使用 middleware `guest` 的可以設定僅有未通過認證的用戶可存取路由。在 `guest` 的保護下，已認證的用戶將會被導向名稱為 `dashboard` 或 `home` 的路由。

你樣可以透過編輯 `bootstrap/app.php` 異動行為：

```php{linenos=true}
// bootstrap/app.php

use Illuminate\Http\Request;
 
->withMiddleware(function (Middleware $middleware): void {
    $middleware->redirectUsersTo('/panel');
 
    // Using a closure...
    $middleware->redirectUsersTo(fn (Request $request) => route('panel'));
})
```

#### Specifying a Guard

當使用 middleware `auth` 時，你可以指定要使用那一個 `guard` 進行認證。 guard 必須是在 `config/auth.php` 的 `guard` 陣列中所設定的 key。

```php{linenos=true}
Route::get('/flights', function () {
    // Only authenticated users may access this route...
})->middleware('auth:admin');
```

### Login Throttling

若使用 Laravel 的 application starter kits， 嘗試登入會自動加入 rate limit。

範例為 Fority 預設的 rate limit 相關程式碼

```php{linenos=true}
RateLimiter::for('login', function (Request $request) {
    // rate limit 規則
    // 每個 ip 或使用者名稱(預設為 email) 
    // 每分鐘最多可嘗試登入 5 次
    $throttleKey = Str::transliterate(Str::lower($request->input(Fortify::username())).'|'.$request->ip());

    return Limit::perMinute(5)->by($throttleKey);
});

```

## Manually Authenticating Users

若沒有使用 Laravel 提供的 application starter kit，則必須自行設計 user 認證。

使用 facade `Auth::attempt()` 對 user 傳入的 credentials 進行認證(通常指由 login 表單填入的帳號與密碼欄位)。認證通過後，你必須重建 user 的 session 來避免 session fixation 攻擊。

```php{linenos=true}
<?php
 
namespace App\Http\Controllers;
 
use Illuminate\Http\Request;
use Illuminate\Http\RedirectResponse;
use Illuminate\Support\Facades\Auth;
 
class LoginController extends Controller
{
    /**
     * Handle an authentication attempt.
     */
    public function authenticate(Request $request): RedirectResponse
    {
        $credentials = $request->validate([
            'email' => ['required', 'email'],
            'password' => ['required'],
        ]);
 
        if (Auth::attempt($credentials)) {
            $request->session()->regenerate();
 
            return redirect()->intended('dashboard');
        }
 
        return back()->withErrors([
            'email' => 'The provided credentials do not match our records.',
        ])->onlyInput('email');
    }
}
```

`Auth::attempt()` 接收一組 key/value 陣列參數，透過該組參數可在資料庫找到 user 資料，並將輸入的 password 與 user 的 password 進行比對。要注意的是，輸入的 password 值並不需要進行 hash ，attempt() 內部會自行處理將明碼 password 進行 hash。

Laravel 的認證機制是從 `config/auth.php` 設定的 guard 的 provider 取得 user 資料。預設是 Eloquent 的 user provider 來源是 `App\Models\User`。

若認證成功 attempt() 會回傳 true，否則回傳 false。

`redirect()->intended()` 方法則會用戶嘗試導向在尚未被 `auth` 攔截請求前的頁面。若無法找到或前往該頁面，則可以設定一個備援的位置。

範例：

若用戶登入後的動作為 `redirect()->intended('dashboard')`

1. 假設用戶（未登入）嘗試存取 profile (使用者頁面)。
2. auth 中間件發現使用者未登入，攔截請求，並將 profile 儲存起來。
3. 中間件將使用者導向登入頁面 login。
4. 使用者成功登入。
5. 此時，登入控制器不應該導向到主頁，而應該使用 intended() 方法，將使用者導向回他們一開始想去的 profile。
6. 若用戶無法前往 profile，或者是用戶是直接請求 login(即沒有 1~3 步驟)，則系統會將用戶導向 dashboard。

### 用戶登入實作細節

#### Specifying Additional Conditions

若驗證時希望增加額外的查詢條件，則可以在 attempt 增加欄位條件。

```php{linenos=true}
if (Auth::attempt(['email' => $email, 'password' => $password, 'active' => 1])) {
    // Authentication was successful...
}
```

若是更複雜的查詢條件，則可以提供 closure

```php{linenos=true}
use Illuminate\Database\Eloquent\Builder;
 
if (Auth::attempt([
    'email' => $email,
    'password' => $password,
    fn (Builder $query) => $query->has('activeSubscription'),
])) {
    // Authentication was successful...
}
```

{{< alert type="notice" >}}
`Auth::attempt($credentials)` 是依靠傳入的 $credentials 的欄位名稱對應 user 資料庫中的欄位名稱取得 user。  
要注意的是在比對密碼時，**password 欄位名稱預設是固定的**，若有需要修改密碼欄位名稱，則需要去調整部份實作內容。
{{< /alert >}}

若要在身份和密碼認證成功後，針對用戶進行進一步的檢驗，則可使用 `Auth::attemptWhen()` 方法。  
`Auth::attemptWhen()` 允許傳入第二個 callback function 參數，該 callback function 會在用戶身份和密碼認證成功後，帶入 `User $user` 並執行。

範例：用戶認證成功後，判斷用戶是否未被停權

```php{linenos=true}
if (Auth::attemptWhen([
    'email' => $email,
    'password' => $password,
], function (User $user) {
    return $user->isNotBanned();
})) {
    // Authentication was successful...
}
```

#### Accessing Specific Guard Instances

若要以不同的 guard 來進行認證和取得 user 資料，則使用 `Auth::guard($guard)` 方法，參數 `$guard` 是設定在 `config/auth.php` 的 guards 陣列中的 guard。

{{< alert type="notice" >}}
若 facade `Auth` 預設未使用 `guard()` 指定 guard，預設使用 `config/auth.php` 中的 `defaults.guard` 的設定。  
補充：Laravel 12 中預設為 `env('AUTH_GUARD', 'web')`。
{{< /alert >}}

### Remembering Users

若要為 application 提供**記住我(remember me)**功能，可以在 `Auth::attempt()` 傳入第二個參數 `bool $remember`。

若 $remember 為 true，則 Laravel 將維持用戶登入狀態，直到用戶手動登出。  
若要使用 remember me 機制，users 資料表必須包含欄位 `remember_token`，該欄位會儲存 remember me 機制所需的 token。

```php{linenos=true}
use Illuminate\Support\Facades\Auth;
 
if (Auth::attempt(['email' => $email, 'password' => $password], $remember)) {
    // The user is being remembered...
}
```

若 application 提供 remember me 機制，在用戶認證成功後，會產生一組 remember me token，儲存在資料庫的 `users.remember_token` 以及用戶端的 cookie。

你可以使用 `Auth::viaRemember()` 來判斷當前已認證的 user 是否是透過 remember me cookie 通過認證。

```php{linenos=true}
use Illuminate\Support\Facades\Auth;
 
if (Auth::viaRemember()) {
    // 若是透過 remember me token 認證的使用者
    // 可以要求 user 進行二階段認證等進一步的認證
}
```

### Other Authentication Methods

#### Authenticate a User Instance

若需要將一個已存在的 user instance 設定為已認證

```php
// 將 $user 設為已認證
Auth::login($user);

// 也可加上 remember me
Auth::login($user, $remember = true);

// 若要指定 guard，例如 admin
Auth::guard('admin')->login($user);

```

#### Authenticate a User by ID

若要直接將 users 資料表 primary key 設定已認證

```php
// 將 users.id = 1 的 user 設為已認證
Auth::loginUsingId(1);

// 也可加上 remember me
Auth::loginUsingId(1, remember: true);
```

#### Authenticate a User Once

若要在單一次的 request 將 user 設定為已認證

```php
if (Auth::once($credentials)) {
    // ...
}
```

此認證僅在本次 request 有效，且不會觸發 `Login` event。

## HTTP Basic Authentication

暫無需求，故先略過。

## Logging Out

使用 `Auth::logout()` 可將 user 登出。登出 user 時可視情況同時將 session 清除以及重建 CSRF Token。登出後通常會將 user 導向其他頁面。如範例：

```php{linenos=true}
use Illuminate\Http\Request;
use Illuminate\Http\RedirectResponse;
use Illuminate\Support\Facades\Auth;
 
/**
 * Log the user out of the application.
 */
public function logout(Request $request): RedirectResponse
{
    // 將 user 登出
    Auth::logout();
 
    // 清除 session
    $request->session()->invalidate();
 
    // 重建 CSRF Token
    $request->session()->regenerateToken();
 
    // 轉址到其他頁面(例如首頁)
    return redirect('/');
}
```

### Invalidating Sessions on Other Devices

Laravel 提供一種機制，可在保留目前裝置的 user 認證的同時，將 user 在其他裝置的 session 廢除並登出。此機制通常應用在 user 變更密碼後，會將 user 從其他裝置登出，並保留目前裝置的認證狀態。

使用此機制前，必須確認 middle `Illuminate\Session\Middleware\AuthenticateSession`(alias `auth.session`) 已套用在 route。通常會將此 middleware 設定在 route group，已套用大部分 routes。如範例：

```php{linenos=true}
Route::middleware(['auth', 'auth.session'])->group(function () {
    Route::get('/', function () {
        // ...
    });
});
```

接著你可以使用 `Auth::logoutOtherDevices()` 此方法必須驗證 user 的密碼，通常會提供一個表單讓 user 輸入密碼。如範例：

```php{linenos=true}
use Illuminate\Support\Facades\Auth;

Auth::logoutOtherDevices($currentPassword);
```

當 `Auth::logoutOtherDevices()` 被調用，user 的其它 sessions 將被廢除，代表由其他地方完成的認證將被登出。

## Password Confirmation

Larave 提供內建的機制，若 user 要取得一些敏感資料或進入敏感的功能，可要求 user 再次驗證密碼。要實作此功能，你必須加入 2 個 route：

1. 一個 route 用來顯示讓 user 再次輸入密碼的頁面。
2. 一個 route 用來驗證 user 密碼，並將 user 重新導向他們的目標。

### Configuration

在完成密碼驗證後，user 在 3 小時內不需要在重新驗證。你也可以透過修改 `config/auth.php` 的 `password_timeout` 設定，來變更 user 必須重新驗證密碼的間隔時間。

### Routing

#### The Password Confirmation Form

第一個 route 用來提供 user 輸入密碼的表單。如範例

```php{linenos=true}
Route::get('/confirm-password', function () {
    // auth.confirm-password 為至少包含一個 password 欄位的表單視圖
    return view('auth.confirm-password');
})->middleware('auth')->name('password.confirm');

```

#### Confirming the Password

第二個 route 用來驗證 user 輸入的密碼，並將 user 重新導向目標。如範例：

```php{linenos=true}
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Hash;
 
Route::post('/confirm-password', function (Request $request) {
    // 若密碼不正確，回到上一頁並加入錯誤訊息
    if (!Hash::check($request->password, $request->user()->password)) {
        return back()->withErrors([
            'password' => ['The provided password does not match our records.']
        ]);
    }
    

    // 重要！在 user 的 session 設定最後一次驗證密碼的時間，用來判定何時逾時。
    $request->session()->passwordConfirmed();
 
    // 將 user 導向目標位置。
    return redirect()->intended();
})->middleware(['auth', 'throttle:6,1']);
```

### Protecting Routes(Password Confirmation)

你應該替需要密碼驗證保護的 route 設定 middleware `password.confirm`。Laravel 預設已裝載此 middleware。此 middleware　會自動將 user 的目標位置除存在 session 中，並將 user 重新導向至名稱為 `password.confirm` 的 route。待 user 完成密碼驗證後，即可將 user 重新導向已儲存的目標位置。如範例：

```php{linenos=true}
Route::get('/settings', function () {
    // ...
})->middleware(['password.confirm']);
 
Route::post('/settings', function () {
    // ...
})->middleware(['password.confirm']);
```

## Adding Custom Guards

//TODO 待補
