---
title: "初探 blade layout"
date: 2025-02-06 14:30:00 +0800
categories: 
  - laravel
tags:
  - laravel
---

使用 extends 的方式讓頁面套用 template。

## 建立 layouts

在專案根目錄建立 `resources/views/layouts/app.blade.php`

{{< alert type="info">}}
`app.blade.php` 為 layout 檔名，可自訂。
{{< /alert >}}

```php
<!DOCTYPE html>
<html lang="zh-tw">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Laravel 10 Task List App</title>
</head>
<body>
    <h1>@yield('title')</h1>
    <div>
        @yield('content')
    </div>
</body>
</html>

```

`@yield()` 用來設定 section name，套用此 layout 的頁面可以使用 `@section()... @endsection` 區塊來插入內容。

例如建立一個 task list 頁面檔案 `tasks.blade.php`

```php
@extends('layouts.app')

@section('title', 'This list of tasks')

@section('content')
    <div>
        @forelse($tasks as $task)
        <div>
            <a href="{{ route('tasks.show', ['id' => $task->id]) }}">{{ $task->title }}</a>
        </div>
        @empty
        <div>There are no tasks!</div>
        @endforelse
    </div>
@endsection

```

- `@extend(layouts.app)` 代表繼承 layouts/app.blade.php。
- `@section('title', 'some text')` 代表將 some text 帶入到 layouts.app 的 `@yield('title')` 區塊。因為提供第二個參數 some text，所以不需要 `@endsection`。
- `@section('content') ... @endsection` 則會將內容帶入到 layouts.app `@yield('content')` 區塊。
