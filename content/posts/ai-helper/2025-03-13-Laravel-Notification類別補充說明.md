---
title: "Laravel Notification 類別補充說明"
date: 2025-03-13 10:40:00 +0800
categories: 
  - chat-gpt-helper
tags:
  - laravel
---

在 Laravel 的 `Notification`（通知）系統中，`toArray()` 方法的主要作用是**定義通知被存入資料庫時的結構**。這個方法會將通知轉換為陣列，並且用於**資料庫通知（Database Notification）** 通道。

## toArray() 的用途

當通知透過 `database` 通道傳送時，Laravel 會呼叫 `toArray()`，將通知的內容轉換為 JSON 格式，然後存入資料庫的 `notifications` 資料表。

## toArray() 方法的範例

假設你有一個訂單通知 `OrderShipped`，通知使用 `database` 通道，你可以這樣定義 `toArray()` 方法：

```php
use Illuminate\Notifications\Notification;

class OrderShipped extends Notification
{
    public function via($notifiable)
    {
        return ['database']; // 使用 database 通道
    }

    public function toArray($notifiable)
    {
        return [
            'order_id' => $this->order->id,
            'status'   => 'shipped',
            'message'  => "您的訂單 #{$this->order->id} 已經出貨！"
        ];
    }
}
```

當這個通知被發送時，Laravel 會將 `toArray()` 返回的資料存入 `notifications` 資料表中。

### 存入 notifications 資料表的結果

當通知發送給某個使用者時，資料庫中的 `notifications` 資料表可能會存入類似以下的內容：

| id  | type               | notifiable_type | notifiable_id | data |
|-----|--------------------|----------------|--------------|------|
| 1   | App\Notifications\OrderShipped | App\Models\User | 5 | `{"order_id":123, "status":"shipped", "message":"您的訂單 #123 已經出貨！"}` |

### 如何取得已存的通知

當通知存入資料庫後，可以透過 `notifications` 關聯來獲取該使用者的通知：

```php
$user = App\Models\User::find(5);

// 取得所有通知
$notifications = $user->notifications;

// 取得第一筆通知的內容
$firstNotification = $notifications->first()->data;
```

這樣就可以拿到 `toArray()` 所定義的內容：

```php
[
    'order_id' => 123,
    'status'   => 'shipped',
    'message'  => '您的訂單 #123 已經出貨！'
]
```

## 結論

- `toArray()` **用於將通知轉換為 JSON 儲存到資料庫**，僅適用於 `database` 通道。
- **當你想要在前端顯示通知時，會從資料庫讀取這些 JSON 格式的資料。**
- 若通知使用其他通道（如 `mail` 或 `slack`），`toArray()` **不會被使用**，而是對應 `toMail()` 或 `toSlack()` 方法。

這樣你可以自訂通知的結構，並讓前端根據存入的 JSON 來渲染通知內容。
