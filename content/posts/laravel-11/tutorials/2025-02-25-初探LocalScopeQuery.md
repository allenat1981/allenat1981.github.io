---
title: "初探 Local Scope Query"
date: 2025-02-25 10:50:00 +0800
categories: 
  - laravel
tags:
  - laravel
---

當需要自訂查詢條件取得資料列時，可將常用的查詢條件在 Model 內自訂為一個以 scope 為前綴的方法，可用該方法取得符合查詢條件的資料列，也可用來串接更多查詢條件。

本文件的範例來自 Udemy 課程

Master Laravel 11 & PHP: From Beginner to Advanced: \#4.40

參考資料：

- Scope Query 教學[Learn to master Query Scopes in Laravel](https://laravel-news.com/query-scopes)。
- 官方文件 [Query Scope](https://laravel.com/docs/11.x/eloquent#query-scopes)

## 使用 where() 建立 Builder

若要查詢 title 含有字串 "qui" 的 Book 資料列，可以使用 where 方法自行串接查詢條件

```php
\App\Models\Book::where('title', 'LIKE', '%qui%')->get();
```

{{< alert type="info" >}}
`where()` 方法會回傳 Builder instance，此類別是用來建立 SQL Query，所以最後需要呼叫方法 `get()`，由資料庫取得資料列。  
若要檢視 Builder 產生的 SQL 語句，則可以將 `get()` 改為 `toSql()`。
{{< /alert >}}

## 建立 Local Query Scope

一般在系統邏輯設計中，特定的服務通常呼叫已經設定好的查詢條件，並透過傳入參數到查詢條件來取得資料，若使用 `where()` 方法來進行，在可讀性上較不直觀，此時可以替 Model 建立 Query Scope 方法。以 title 為條件查詢 Book 資料列為例，可建立方法 `scopeTitle()` 來替代 `where('title', ...)`。

編輯 /app/Models/Book.php

```php
//...
use Illuminate\Database\Eloquent\Builder;

public function scopeTitle(Builder $query, string $title): Builder
{
    return $query->where('title', 'LIKE', "%{$title}%");
}

```

{{< alert type="notice" >}}
在 Model 建立 Scope Query 的方法，方法名稱請加上小寫前綴 scope。
{{< /alert>}}

建立完成後即可使用該 Scope Query 方法串接或取得資料列。

```php
\App\Models\Book::title("qui")->where("created_at", "<", "2025-01-01")->get();
```

{{< alert type="info" >}}
以下幾點說明：

1. 呼叫 Scope Query 時，僅需以小寫方法名稱呼叫接在 scope 後的方法名稱，例如方法名稱為 `scopeTitle()`，則呼叫 `Book::title()` 即可。
2. Scope Query 仍然回傳 Builder instance，故可以繼續進行其他查詢條件的串接。
{{< /alert >}}

## 補充：多欄位排序

當多個 Scope Query 方法有設定排序欄位時，在串接時，則會依照串接順序(左->右)進行欄位排序串接。

例如：  
在 Book 新增以下 Scope Query 方法

```php
public function scopePopular(Builder $query): Builder
{
    return $query->withCount('reviews')
        ->orderBy('reviews_count', 'DESC');
}

public function scopeHighestRated(Builder $query): Builder
{
    return $query->withAvg('reviews', 'rating')
        ->orderBy('reviews_avg_rating', 'DESC');
}

```

調用 popular() 和 highestRated() 的順序，會影響排序的順序

```php

\App\Models\Book::popular()->highestRate()->toSql();
// 輸出 ... order by `reviews_count` desc, `reviews_avg_rating` desc

\App\Models\Book::highestRated())->popular()->toSql();
// 輸出 ... order by `reviews_avg_rating` desc, `reviews_count` desc
```
