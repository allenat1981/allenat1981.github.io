---
title: "laravel 表單與 CRUD 常用功能"
date: 2025-02-12 11:00:00 +0800
categories: 
  - laravel
tags:
  - laravel
---

使用表單與資料庫進行 CRUD 時一些常用的功能與技巧。

本文件的範例來自 Udemy 課程

Master Laravel 11 & PHP: From Beginner to Advanced: \#3.22 ~ \#3.25, \#3.29

## 表單 CSRF token

在表單內插入 `@csrf` 可以自動產生 CSRF token

```html
<form ...>
    @csrf
    ... 表單內容
</form>
```

## 表單驗證與錯誤顯示

表單送出後可以使用 `$Request::validate()` 方法來驗證表單。

```php

Route::post('/tasks', function(Request $request) {
    $data = $request->validate([
        'title' => 'required|max:255',
        'description' => 'required',
        'long_description' => 'required'
    ]);
});

```

`$request->validate()` 設定驗證規則，可使用 | 進行規則串接。  
若驗證失敗則會自動回到表單送出的前一個頁面。此時可以在 blade 進行以下處置

- 在欄位的 value 屬性使用 `{{ old('fieldname') }}`，保留使用者輸入內容。`old()` **僅能保留 POST 方法送出的欄位**。
- 在欄位下方使用 `@error` 區塊顯示欄位驗證的錯誤訊息。

{{< alert type="notice" >}}
使用 old() 時，應該僅設定在非敏感資料的欄位，若是密碼或身分證字號等敏感內容，建議不要使用 old()，而是應該讓使用者重新輸入一次。
{{< /alert >}}

```html
<form action="pathtosubmit" method="POST">
<div>
    <label for="title">
        Title
    </label>
    <input type="text" name="title" id="title" value="{{ old('titile') }}>
    @error('title')
        <p class="error-message">{{ $message }}</p>
    @enderror
</div>
... other input field
</form>
```

### 進階：使用 make:request 建立 Request 進行驗證

若有不同的表單會用到重複的驗證方法，為了避免在每個 route 裡面重複撰寫驗證規則，則可以 `make:request` 建立繼承自 FormRequest 的類別，並在該類別的 rules() 方法設定驗證規則。

例如 create 和 edit 表單都會要求使用者輸入同樣的欄位進行驗證，但是後端通常會由不同 route 處理，此時可以建立一個 TaskRequest 設定共用的驗證規則。

```bash
# 建立一個 TaskRequest
php artisan make:request TaskRequest

# 產生 app/Http/Requests/TaskRequest.php
```

接著編輯 TaskRequest.php

```php
class TaskRequest extends FormRequest
{
    public function authorize(): bool
    {
        return true;
    }

    public function rules(): array
    {
        return [
            'title' => 'required|max:255',
            'description' => 'required',
            'long_description' => 'required'
        ];
    }
}
```

- `authorize()` 是驗證使用者 Request 是否有權限訪問，故可以設定為 true。
- `rules()` 可以已陣列方式回傳我們自訂的驗證規則。

設定好驗證規則後，在 route 的 controller 方法，將原本的 type hint 改為我們建立的 TaskRequest，呼叫 `$TaskRequest::validated()` 方法即可取得驗證過後的資料。

```php
// 範例的 post 和 put 來自不同的表單，但因為表單內容相同，故可以共用 TaskRequest 來進行驗證

Route::post('/tasks', function(TaskRequest $request) {
    $data = $request->validated();
    //... 後續處理
});

Route::put('/tasks', function(TaskRequest $request) {
    $data = $request->validated();
    //... 後續處理
})
```

TaskRequest 的驗證會在進入 controller 前完成，故呼叫 `validated()` 取得的已經是通過驗證的資料。

## 表單的 method

一般 html 表單的 method 只能是 GET 和 POST，所以 blade 提供 `@method()` 來變更表單 method。

```html
<form method="POST" action="...">
    @method('PUT')
    ... 表單內容
</form>
```

以上即可讓 route 以 PUT 方法處理表單。

{{< alert type="notice" >}}
若是針對後端資料的異動，無論是新增/修改，或僅僅是切換一個狀態欄位，考量到安全性的情況下，都應該使用表單傳送資料，並使用 @method() 搭配 @csrf 驗證。

GET 方法應該僅運用在從後端獲取資料的情境。
{{< /alert >}}

## flash message

laravel 提供 flash message 的機制，該機制是透過 session 儲存一次性的內容，適合用來在表單送出後，保存處理結果並顯示在頁面上。上述表單驗證的 errors 也是同樣的機制。

要加入 flash message，可以呼叫 `$RedirectResponse::with()` 方法。例如新增/更新資料完成後，可以重新導向該筆資料的 show 頁面，並顯示以 `with()` 設定的 message。

```php
Route::put('/tasks/{task}', function(Task $task, TaskRequest $request) {
    $task->update($request->validated());

    return redirect()
        ->route('tasks.show', ['task' => $task->id])
        ->with('success', 'Task updated successsfully!'); //加入 flash message
})->name("tasks.update");
```

在 `tasks.show` 渲染的 blade 內可使用 `session()` 方法顯示 flash message 的內容。

```html
@if (session()->has('success'))
    <div>{{ session('success') }}</div>
@endif
```

## Route 綁定 Model

*請參考《Laravel 啟動與運行》 p.49 ~ p.51。*

若 route 的 controller 會使用到 model，若不想在 controller 內總是重覆撰寫 `$Model::find()` 或 `$Model::findOrFail()`，則可以將 route parameter 名稱設定為 Action Method 的參數，並且設定 type hint 為 Model 名稱。在 route() 傳入該 Model 的 PK 值或直接傳入 $Model，即可讓 laravel 自動 binding Model 的 instance。

例如在 route 要綁定 model Task

```php
Route::get('/tasks/{task}', function(Task $task) {
    return view("show", [
        'task' => $task
    ]);
})->name('tasks.show');
```

- Route Parameter 名稱與負責的 Action Method 參數名稱相同，例如 `{task}` 與 `$task`。
- - action method 的 type hint 設定為 Model 的類別名稱，例如 Task。
- `route('tasks.show', ['task' => $task])`，$task 可以是 Task 的 instance 或 id。

### 補充：blade route() 傳入綁定 model 索引值

在 blade 使用 `route()` 產生路由路徑時，有二種方法可以傳入綁定 model 的索引值

方法一，明確指定欄位

```html
<a href="{{ route('tasks.edit', ['task' => $task->id]) }}">Link Name</a>
```

此範例在 `route()` 的參數傳入 `$task->id`，即明確指定 `{task}` 的欄位為 `$task->id`。

{{< alert type="info" >}}
此方法亦適用使用其他索引欄位。
{{< /alert >}}

方法二，直接傳入`$Model`

```html
<a href="{{ route('tasks.edit', ['task' => $task]) }}">Link Name</a>
```

此範例在 `route()` 的參數傳入 `$task`，laravel 會隱含地將 `{task}` 傳入 model 的 PK，即 `$task->id`。

{{< alert type="info" >}}
此方法語句較為簡易，但僅適用於 {task} 索引欄位為 PK 的情境。
{{< /alert >}}

### 補充：使用其他索引欄位

若要調整 {task} 索引的欄位，則可以在 Models\\Task 內加入 `getRouteKeyName()` 方法進行設定，例如有一個欄位名稱為 slug，要用來作為綁定的索引：

```php
public function getRouteKeyName()
{
    return 'slug';
}
```

## Shortcut:新增/修改 Model

*請參考《Laravel 啟動與運行》 p.133 ~ p.136。*

laravel 的 model 機制提供快速可新增/修改 model 的方法：`Model::create()` 與 `$Model::update()`。

- `Model::create()` 新增一筆資料到資料庫，並傳回 instance。
- `$Model::update()` 更新 instance 的資料到資料庫。

對 Models\\Task 進行新增與修改，則可使用以下 shortcut 方式

```php
// Task::create() 新增資料
Route::post('/tasks', function(TaskRequest $request) {
    $task = Task::create($request->validated());

    return redirect()->route('tasks.show', ['task' => $task->id])
        ->with('success', 'Task created successsfully!');
})->name("tasks.store");

// $task->update() 修改資料
Route::put('/tasks/{task}', function(Task $task, TaskRequest $request) {
    $task->update($request->validated());

    return redirect()->route('tasks.show', ['task' => $task->id])
        ->with('success', 'Task updated successsfully!');
})->name("tasks.update");
```

傳入 `create()` 和 `update()` 資料為 key/value 的陣列，其中 key 的部分即為 data table 的 column name。

因為安全性的緣故，所以必須在 Model 設定允許新增/更新的欄位。

編輯 Models\\Task

```php
class Task extends Model
{
    protected $fillable = ['title', 'description', 'long_description'];
    
    //protected $guarded = ['password', 'identified'];
}
```

`$fillable` 陣列內代表允許新增/修改的欄位。  
`$guarded` 陣列則是代表禁止新增/修改的欄位。通常用於資料表欄位很多，不想在 $fillable 設定太多欄位時，可改用 $guarded 做反向操作。

## 補充：使用 request() 來保留 GET Query

此範例來自課程 #4.48。

若表單使用 GET 方法進行查詢，並要在送出表單後，保留 input 的值，則可在 blade 使用 `request()` 來取得 Query。

```html
<form method="GET" action="{{ route("books.index") }}">
    <input type="text" name="title" placeholder="Search by title" value="{{ request('title') }}" />
    <input type="hidden" name="filter" value="{{ request('filter') }}" />
</form>
```

若要在 `route()` 使用陣列傳入 query parameters，也可以使用 unpack 語法 `...request()->query()`。

```html
<a href="{{ route('books.index', [...request()->query(), 'filter' => $key]) }}">Link Label</a>
```
