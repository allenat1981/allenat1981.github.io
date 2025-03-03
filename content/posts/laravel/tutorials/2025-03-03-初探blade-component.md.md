---
title: "初探 blade components"
date: 2025-03-03 14:30:00 +0800
categories: 
  - laravel
tags:
  - laravel
---

利用 blade component 建立可重複使用的網頁元件。

本文件的範例來自 Udemy 課程

Master Laravel 11 & PHP: From Beginner to Advanced: \#4.54

官方文件[Components](https://laravel.com/docs/11.x/blade#components)

## 建立 component

使用以下指令建立 component

```bash
php artisan make:component StarRating
```

將會生成

- `resources/views/components/start-rating.blade.php` component 的 blade 檔案
- `app/View/Components/StarRating.php` component 的對應資料類別

### 編輯 component blade 檔案

可在 blade 檔案內設計 component 的內容。  
範例：

```php
@if($rating)
    @for($i = 1; $i <= 5; $i++)
    {{ $i <= round($rating) ? '★' : '☆' }}
    @endfor
    {{ number_format($rating, 1) }}
@else
    No rating yet
@endif
```

### 傳入資料至 component

編輯資料類別，可在 construct 傳入 component 接收的參數。  
範例：

```php
public function __construct(
    public readonly ?float $rating
)
{
    //
}
```

以上代表可傳入一個 readonly 的 $rating 到 component Star-Rating

### 在其他 blade 中使用 component

在其他 blade 可使用 `<x-component-name></x-component-name>` 標籤嵌入 component。  
若需傳入參數則可使用 `<x-componanet-name :paramName="$param"></x-componanet-name>`。  
範例：

```php
// 在其他 blade 檔案中嵌入 StarRating
<div class="book-rating">
  <x-star-rating :rating="$book->reviews_avg_rating"></x-star-rating>
</div>
```
