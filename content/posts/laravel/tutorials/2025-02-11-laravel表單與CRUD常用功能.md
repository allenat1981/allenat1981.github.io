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

Master Laravel 11 & PHP: From Beginner to Advanced: \#3.22 ~ \#3.25

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

- 在欄位的 value 屬性使用 `{{ old('fieldname') }}`，保留使用者輸入內容。`old()` 僅能保留 POST 方法送出的欄位。
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

## route 綁定 model

*請參考《Laravel 啟動與運行》 p.49 ~ p.51。*

若 route 的 controller 會使用到 model，若不想在 controller 內總是重覆撰寫 `$Model::find()` 或 `$Model::findOrFail()`，則可以在 route 的參數設定要綁定的 model 之索引鍵(通常為主鍵 id)，並對 controller 的參數指定該 model 的 type hint，即可讓 laravel 自動。

例如在 route 要綁定 model Task

```php
Route::get('/tasks/{task}', function(Task $task) {
    return view("show", [
        'task' => $task
    ]);
})->name('tasks.show');
```

- Route 參數 uri 設定 {task} 用來代表傳入 Models\\Task 的索引鍵(通常為主鍵 id)。
- closure 方法參數則 type hint 類別名稱 Task。

以上設定 laravel 即會自動綁定 $task，該 $task 即為 id 為 {$task} 的資料。

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
