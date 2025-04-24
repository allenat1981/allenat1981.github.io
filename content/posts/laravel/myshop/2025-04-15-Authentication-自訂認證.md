---
title: "Authentication 自訂認證"
date: 2025-04-15 14:30:00 +0800
categories: 
  - laravel
tags:
  - laravel
  - myshop
---

將使用者分為一般前台使用者(user) 和後台使用者(admin)，並分別為其建立相對應的 Authentication Guard。

## authentication

將購物網分為前台(user)和後台(admin)，若要分別針對前後台進行驗證，則可自訂不同的 Authentication Guard。設計如下：

- `user` 使用 Laravel 專案預設的 `Models\User`，使用 `email` 當作帳號登入。
- `admin` 使用自訂的 `Models\Admin`，使用 `account` 當作帳號登入。
- 密碼均使用預設的 `password` 欄位；`remember_token` 作為記住我。
- 剩餘欄位則根據需求自訂。

若非必要，盡量使用預設的欄位，減少須要自行撰寫的部份。

### 1. 設計 Model Admin

建立 Models\Admin 提供給 Authentication 機制作為 provider。

```bash
php artisan make:model Admin -mf
```

編輯 `database/migrations/xxxxx_create_admins_table.php`

```php
public function up(): void
{
    Schema::create('admins', function (Blueprint $table) {
        $table->id();

        $table->string('account', 60)->unique();
        $table->string('password',255);
        $table->string('name', 100);
        $table->string('email', 100);
        $table->rememberToken();

        $table->timestamps();
    });
}
```

因為使用 `account` 作為登入帳號，故設計為 `unique`。

編輯 `app/Models/Admin.php`

```php
class Admin extends Authenticatable
{
    use HasFactory;

    protected $hidden = [
        'password',
        'remember_token',
    ];

    protected function casts(): array
    {
        return [     
            'password' => 'hashed',
        ];
    }
}
```

說明：

- 衍生自 `Authenticatable`。
- `casts()` 可設定欄位屬性轉換，例如：`password => hashed` 可在存取 password 欄位資料時進行 Hash。

### 2. 設定 auth configuration

編輯 `config/auth.php`:

在 guards 加入 user 和 admin。

```php
'guards' => [
    'user' => [
        'driver' => 'session',
        'provider' => 'users'
    ],
    'admin' => [
        'driver' => 'session',
        'provider' => 'admins'
    ],
],
```

在 providers 加入 `admins`（預設已有 users）。

```php
'providers' => [
    'users' => [
        'driver' => 'eloquent',
        'model' => env('AUTH_MODEL', App\Models\User::class),
    ],

    'admins' => [
        'driver' => 'eloquent',
        'model' => env('AUTH_MODEL', App\Models\Admin::class)
    ],
],
```

完成以上步驟後，即可使用 `Auth::guard('admin')` 和 `Auth::guard('user')` 對不同角色進行驗證。

### 3. 設定驗證失敗的轉址路由

Authentication 在驗證失敗（未登入）時，預設會轉址到名稱為 `login` 的路由。若想要替不同的 guard 轉址到不同路由，則可以如下設定。

編輯 `bootstrap/app.php`

```php
->withMiddleware(function (Middleware $middleware) {
    $middleware->redirectGuestsTo(function (Request $request) {
        
        // if ($request->routeIs('admin.*')) {
        //     return route('admin.login');
        // }
        if ($request->is(['admin','admin/*'])) {
            return route('admin.login');
        }

        // if ($request->routeIs('user.*')) {
        //     return route('user.login');
        // }
        if ($request->is(['user', 'user/*'])) {

            return route('user.login');
        }

        return route('login');
    });
})
```

說明：

編輯 `$middleware->redirectGuestsTo()` 的內容，使用 `$request->is()` 或 `$request->routeIs()`，判斷目前是那一個路由，藉此決定若未登入則轉址到那一個路由位置。

## 自訂路由群組

編輯 `bootstrap/app.php`，並在 `then` 加入自訂的路由設計。

```php
->withRouting(
    web: __DIR__.'/../routes/web.php',
    commands: __DIR__.'/../routes/console.php',
    health: '/up',
    then: function () {
        Route::middleware('web')
        ->prefix('user')
        ->name('user.')
        ->group(base_path('routes/user.php'));

        Route::middleware('web')
        ->prefix('admin')
        ->name('admin.')
        ->group(base_path('routes/admin.php'));
    }
)
```

說明：

- `Route::middleware('web')` 可替路由加入 web 相關的 middleware。
- `prefiex()` 讓路由路徑預設加入 `/prefix`，例如 `/user`, `/admin`。
- `name()` 會自動替路由命名加入前綴，例如 `user.index`, `admin.index`。
- `group()` 可載入自訂的路由檔。

### 建立路由檔

以 user 為例，建立 `routes/user.php`

```php
Route::middleware(['auth:user'])->group(function () {
    Route::get('/', [UserController::class, 'index'])->name('home');
});

Route::get('login', fn () => to_route('user.login.create'))
    ->name('login');

Route::resource('login', LoginController::class)->only(['create', 'store']);
Route::delete('/logout', [LoginController::class, 'destory'])->name('login.destroy');
```

可使用 middleware(`auth:user`) 決定要用那個 Authentication Guard。  
並且在登入或登出時，使用 `Auth::guard('user')->attempt()` 和 `Auth::guard('user')->logout()`。

## Middleware guest

若要讓已經驗證的 user 或 admin 造訪某些路由時轉址，例如已經登入的使用者再次進入 user.login 頁面時，轉址到 user.index，此時可以使用 middleware: guest。

### 1. 設定轉址路由

編輯 `bootstrap/app.php`

```php
->withMiddleware(function (Middleware $middleware) {
    //...

    $middleware->redirectUsersTo(function (Request $request) {
        if ($request->routeIs('admin.*')) {
            return route('admin.home');
        }

        if ($request->routeIs('user.*')) {
            return route('user.home');
        }

        return route('index');
    });
}

```

說明：

`$middleware->redirectUsersTo()` 可設定已登入使用者的轉址路由。

### 2. 替 route 加上 middleware

替 user 或 admin 的 login 路由加上 middleware: guest

```php
Route::middleware('guest:user')->resource('login', LoginController::class);
```

說明：

在需要轉址的路由加上 `guest:user` 或 `guest:admin`，當已登入的使用者造訪該路由，即會自動轉址。
