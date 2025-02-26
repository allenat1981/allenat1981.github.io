---
title: "Route綁定Model的進階用法"
date: 2025-02-26 12:00:00 +0800
categories: 
  - laravel
tags:
  - laravel
---

Route 綁定 Model 的方式，可以快速地在 Controller 內存取 Model instance，但若涉及到 Model 的關聯資料，則需要使用進階的作法。

本文件的範例來自 Udemy 課程

Master Laravel 11 & PHP: From Beginner to Advanced: \#4.50

## 使用 load() 讀取關聯資料

原始的狀況下，若 route 綁定 Model Book，則會在其負責的 controller 內設計如以下方法

```php
public function show(Book $book)
{

    return view(
        'books.show',
        [
            'book' => $book
        ]
    );
}
```

在 blade 內使用 loop 輸出關聯的 reviews 如下

```html
<ul>
  @forelse ($book->reviews as $review)
    <li class="book-item mb-4">
      <div>
        <div class="mb-2 flex items-center justify-between">
          <div class="font-semibold">{{ $review->rating }}</div>
          <div class="book-review-count">
            {{ $review->created_at->format('M j, Y') }}</div>
        </div>
        <p class="text-gray-700">{{ $review->review }}</p>
      </div>
    </li>
  @empty
    <li class="mb-4">
      <div class="empty-book-item">
        <p class="empty-text text-lg font-semibold">No reviews yet</p>
      </div>
    </li>
  @endforelse
</ul>
```

當 `@forelse ($book->reviews as $review)` 此語句取用到 $book->reviews，將會以 lazy-loading 的方式從資料庫讀取 reviews 資料列。

{{< alert type="notice" >}}
Model 的 lazy-loading 可參考[使用Model建立一對多關聯資料表]({{< ref "/posts/laravel/tutorials/2025-02-24-使用Model建立一對多關聯資料表#使用-lazy-loading-的方式取得資料" >}})。  
lazy-loading 機制不僅發生在 php，在 blade 渲染同樣有效。
{{< /alert >}}

若要在 controller 內就把需要的關聯資料讀入，則可使用 load() 方法

```php
public function show(Book $book)
{

    return view(
        'books.show',
        [
            'book' => $book->load('reviews')
        ]
    );
}
```

{{< alert type="notice" >}}
使用 load() 語句時，相當於從資料庫將關聯的資料列讀入 Model 的關聯欄位。  
在此範例中，仍然是讀取了 2 次資料庫，第一次是 route 綁定 Book $book 被 request 的時候，第二次則是 $book->load('reviews)。  
因此若是在需要讀取 n 次 $book 的情境中，查詢資料庫次數會是 n * 2 次，在效能考量上須特別注意。
{{< /alert >}}

若需要針對 `reviews` 的查詢與法進行設定，則可透過傳入陣列與 closure 進行處理。  

例如若要取得排序後的 reviews 資料列：

```php
public function show(Book $book)
{

    return view(
        'books.show',
        [
            'book' => $book->load([
                'reviews' => fn($query) => $query->latest()
            ])
        ]
    );
}
```
