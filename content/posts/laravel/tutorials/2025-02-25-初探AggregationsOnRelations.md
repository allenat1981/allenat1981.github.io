---
title: "初探 Aggregations On Relations"
date: 2025-02-25 10:50:00 +0800
categories: 
  - laravel
tags:
  - laravel
---

本文件的範例來自 Udemy 課程

Master Laravel 11 & PHP: From Beginner to Advanced: \#4.41

## 查詢關聯資料列的統計(Aggregating)資料

參考資料：

- 官方文件 [Aggregating Related Models](https://laravel.com/docs/11.x/eloquent-relationships#aggregating-related-models)

### 查詢關聯資料筆數: withCount()

若要查詢 Book 關聯的 Review 筆數，可使用方法 `withCount()`。

```php
\App\Models\Book::withCount('reviews')->get();
```

此方法也可以繼續串接其他條件，例如若只需要查詢最新建立的 3 筆 Book 與其相關的 Review 資料筆數。

```php
\App\Models\Book::withCount('reviews')->latest()->limit(3)->get();
```

說明：

1. `latest()` 方法會根據 created_at 做 DESC 排序。
2. `withCount()` 輸出欄位名稱命名規則為 `[關聯名稱]_count`，例如本例中的 reviews_avg_rating。

### 查詢關聯資料欄位平均值: withAvg()

若要查詢平均值，則使用方法 `withAvg()`。  
範例：取得平均分數(rating)最高的 5 筆 Book 資料列。

```php
\App\Models\Book::withAvg('reviews', 'rating')->orderBy('reviews_avg_rating', 'DESC')->limit(5)->get();
```

說明：

1. 條件串接的順序基本上可以任意排列，但還是建議有固定的習慣性寫法。例如根據 SQL 語法的規則順序進行串接：查詢, 條件, 排序, 筆數限制 ...
2. `withAvg()` 輸出欄位名稱命名規則為 `[關聯名稱]_avg_[欄位名稱]`，例如本例中的 reviews_avg_rating。

### 對 Aggregating 進行篩選: having()

若要針對 Aggregating 進行條件篩選，則使用方法 having()。

範例：取得符合以下條件的 Book 資料列

- 至少有 10 筆 review 資料。
- 取得平均的 rating 最高的前 10 筆 Book 資料。

```php
\App\Models\Book::withCount('reviews')
    ->withAvg('reviews', 'rating')
    ->having('reviews_count', '>=', 10)
    ->orderBy('reviews_avg_rating', 'DESC')
    ->limit(10)
    ->get();
```

## 補充：Aggregating 效能問題

若使用 `toSql()` 輸出上面幾個範例的 SQL 語句，會發現語句內使用多個子查詢來完成整個查詢語句，因此當 laravel 資料庫操作若遇到效能瓶頸時，可能需要檢查一下是否為這些查詢所造成的問題。

## 自訂 Aggregating 查詢條件

本節的內容來自 #4.44 Getting Books with Recent Reviews

若要在 Aggregating 加入自訂的查詢條件，則需要改用陣列的方式傳入參數，並且以 closure 的方式加入查詢條件 Builder。

範例：  
若要查詢 Book 在某個區間內的瀏覽總數或平均分數，則可直接在 Model 進行 Scope Query 設計

```php
public function scopePopular(Builder $query, ?string $from = null, ?string $to = null): Builder
{
    return $query->withCount([
        'reviews' => fn(Builder $q) => $this->dateRangeFilter($q, $from, $to)
    ])->orderBy('reviews_count', 'DESC');
}

public function scopeHighestRated(Builder $query, ?string $from = null, ?string $to = null): Builder
{
    return $query->withAvg([
        'reviews' => fn(Builder $q) => $this->dateRangeFilter($q, $from, $to)
    ], 'rating')->orderBy('reviews_avg_rating', 'DESC');
}

public function scopeMinReviews(Builder $query, int $minReviews): Builder
{
    return $query->having('reviews_count', '>=', $minReviews);
}

private function dateRangeFilter(Builder $query, ?string $from = null, ?string $to = null)
{
    if ($from && !$to) {
        $query->where('created_at', '>=', $from);
        return;
    }

    if (!$from && $to) {
        $query->where('created_at', '<=', $to);
        return;
    }

    $query->whereBetween('created_at', [$from, $to]);
    return;
}

```

說明：

1. `scopePopular()` 和 `scopeHighestRated()` 中，使用 Arrgregating 方法，並在參數的地方以陣列傳入 closure function。
2. 因為使用同樣的日期篩選條件，故將該方法擷取為 `dateRangeFilter()`。
3. `withAvg()` 和 `withCount()` 透過箭頭函式(Arrow Function) fn() 呼叫 `dateRangeFilter()`，並透過 fn() 的特性，不需要使用 use() 可直接調用外部的 $from 與 $to 參數。

使用 tinker 演示：

統計 reviews 的 created_at 介於 2023-01-01 ~ 2023-03-31 的資料筆數和分數平均值。

```php
//取得 2023-01-01 ~ 2023-03-31 建立的
\App\Models\Book::highestRated('2023-01-01', '2023-03-31')
  ->popular('2023-01-01', '2023-03-31')
  ->minReviews(2)
  ->get();
```
