---
title: "Laravel 的 Event 機制"
date: 2026-04-08 11:32:00 +0800
categories: 
  - laravel
tags:
  - laravel
---

簡單記錄 Laravel Event 與 Listener 的使用

## 基本介紹

事件類別 Event

- 基本上檔案置於 app/Events 目錄。
- 可透過 public 屬性裝載事件觸發時，傳遞的資訊(Payload)或方法。
- 可透過 `dispatch(...)` (官方推薦方式)派送事件到監聽該事件的 Listener。
- 通常在 Controller 或 Service 進行 `dispatch()`。

監聽器類別 Listener

- 基本上檔案置於 app/Listeners 目錄。
- 用來監聽事件並進行處理。
- 包含一個 `public function handle(type-hint $event): void` 方法，type-hint 為事件類別。

事件處理流程

1. 建立 Event 類別與 Listener 類別。
2. Listener 的 handle() 方法指定 type-hint 為 Event 類別，Laravel 會自動將 Listener 監聽 Event。
3. 在 Controller 或 Service 需要觸發事件的地方，調用 Event::dispatch(...)，所有監聽該 Event 的 Listener 的 handle() 將會執行。

## 實做範例

以下實做範例

- Event Class `SecurityActivity`: 安全性操作事件，會在使用者進行有關各種安全性操作時觸發(登入/登出/修改密碼/忘記密碼... 等)
- Listener Class `RecordSecurityActivity`：監聽 SecurityActivity 事件的類別，用來將事件夾帶的資訊寫入 Log

### 建立 Event

使用指令 `artisan make:event` 建立 Event Class: SecurityActivity。

```bash
php artisan make:event

# 依照指示輸入事件名稱:SecurityActivity
# 會生成 app/Events/SecurityActivity
```

編輯 `app/Events/SecurityActivity`，使用 public 屬性設定事件的 Payload。

```php
class SecurityActivity
{
    use Dispatchable, InteractsWithSockets, SerializesModels;

    public string $ip;
    public string $userAgent;

    public function __construct(
        public User $user,
        public string $action,
        public array $data = []
    )
    {
        $this->ip = request()->ip();
        $this->userAgent = request()->userAgent();
    }
}
```

### 建立 Listener

使用指令 `artisan make:listener` 建立 Listener Class: RecordSecurityActivity。

```bash
php artisan make:listener
# 依照指示輸入監聽器名稱:RecordSecurityActivity
# 依照指示輸入要監聽的事件類別:SecurityActivity
# 會生成 app/Listeners/RecordSecurityActivity
```

編輯 `app/Listeners/RecordSecurityActivity`。

handle() 的 type-hint 會自動帶入指定的事件，可在 handle 進行事件觸發時的操作。

```php
class RecordSecurityActivity
{
    public function __construct()
    {
        //
    }

    /**
     * Handle the event.
     */
    public function handle(SecurityActivity $event): void
    {
        // 這裡使用 eloquent model 將事件的 payload 寫入資料庫
        ActivityLog::create([
            'user_id' => $event->user->id,
            'action' => $event->action,
            'data' => empty($event->data) ? null : $event->data,
            'ip' => $event->ip,
            'user_agent' => $event->userAgent
        ]);
    }
}

```

### 觸發事件

在 user 進行安全性操作的動作用 SecurityActivity::dispatch() 觸發事件，例如在登入的 Controller Action Method

```php

public function login(Request $request)
{
    // ... 一系列的登入流程
    // 取得 User $user
    SecurityActivity::dispatch($user, 'user-login', [ ... ]);

    // return response
}

```

當調用 SecurityActivity::dispatch(...) 時，所有監聽 SecurityActivity 事件的 Listener 類別的 handle() 都會被觸發。

## 重點補充

Laravel 預設會對 app/Listeners 目錄內 Listener Class 的 handle(type-hint $event) 的 type-hint 自動進行監聽。

監聽同一事件的 Listeners，處理事件都應該是獨立的操作，彼此之間沒有時間耦合。例如 ListenerA 和 ListenerB 同樣監聽 EventX 時，無法要求 ListenerA 先處理完後才處理 ListenerB(除非在 ListenerA 內調用 ListenerB，但這會造成 ListenerA 與 ListenerB 之間的耦合)。

Laravel 事件可加入 Queue，把工作排入佇列，避免因為觸發事件造成回應延遲(適合處理寄信/發送訊息)。

若事件需要在資料庫操作 commit 成功後才觸發，可在事件類別 implements `ShouldDispatchAfterCommit`。請參考官方文件[Dispatching Events After Database Transactions](https://laravel.com/docs/13.x/events#dispatching-events-after-database-transactions)。
