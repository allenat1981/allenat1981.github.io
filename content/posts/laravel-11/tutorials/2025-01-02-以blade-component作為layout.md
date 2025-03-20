---
title: "初探 blade components"
date: 2025-01-02 14:30:00 +0800
categories: 
  - laravel
tags:
  - laravel
---

本文件若沒有特別指定，blade 檔案位置都位於預設目錄 `專案目錄/resources/views`。

laravel.blade 可以使用 components 和 layouts 二種方式套用模板(template)。  
本文介紹 components。

## blade 樣板

樣板檔名格式為 `[filename].blade.php`

在 Controller 中可以 `return view('檔名')` 的方式渲染到前端。

```php
# 訪問 /about 頁面，會渲染 about.blade.php
Route::get('/about', function () {
    return view('about');
});
```

## layout 元件

在樣板目錄中加入 `components` 子目錄，可以自訂 layout 元件。

### 網站 layout 範例

可建立 `components/layout.blade.php` 作為網站模板，供頁面引入。

```html
<!-- layout.blade.php -->
<!DOCTYPE html>
<html lang="en" class="h-full bg-gray-100">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title></title>
</head>
<body class="h-full">
    <header>
        <!-- 自訂名稱 x-slot:heading -->
        {{ $heading }}
    </header>
    <!-- 預設的 slot -->
    {{ $slot }}
</body>
</html>
```

建立一個套用 layout 的頁面，例如 `about.blade.php`。

```html
<x-layout>
    <x-slot:heading>About Page</x-slot:heading>
    <h1>Hello from the About Page.</h1>
</x-layout>
```

- `<x-layout>` 標籤會引入 layout.blade.php 模板。
- `<x-slot:heading>` 標籤的內容會帶入模板 `{{ $heading }}` 的位置。
- 其餘未指定 slot 的內容會帶入模板 `{{ $slot }}` 的位置。

## 自訂元件

在 components 目錄內也可自訂常用的元件。例如建立固定樣式的動態連結元件。

範例：

建立 `components/nav-link.blade.php` 檔案，裡面包含自訂的 a 標籤。

```html
@props(['active' => false])

<a class="{{ $active ? "bg-gray-900 text-white" : "text-gray-300 hover:bg-gray-700 hover:text-white" }} rounded-md px-3 py-2 text-sm font-medium" aria-current="{{ $active ? 'page' : 'false'}}" {{ $attributes }}>{{ $slot }}</a>
```

- `@props()` 可設定 key/value 陣列，提供嵌入此元件的 prop。
- `{{ $attributes }}` 提供輸出嵌入此元件的 attributes。
- `{{ $slot }}` 即預設帶入嵌入此元件標籤中的內容。

在上述的 layout.blade.php 即可用 `<x-nav-link>` 標籤嵌入元件

```html
<!-- layout.blade.php -->
<!-- html body ... 等省略 -->
<nav>
    <x-nav-link href="/" :active="request()->is('/')">Home</x-nav-link>
    <x-nav-link href="/about" :active="request()->is('about')">About</x-nav-link>
    <x-nav-link href="/contact" :active="request()->is('contact')">Contact</x-nav-link>
</nav>
```

- `<x-nav-link>` 可設定 href 屬性，將會帶入到 nav-link.blade.php 的 a 標籤。
- `:active` 代表傳入一個 expression 到 nav-link.blade.php 所設定的 @props。
  - 請注意！若 active 前面沒有加上 :(colon)符號，代表傳入字串(string)。
  - `request()->is('about')` 代表若 request 的 route 若為 about，則回傳 true，否則為 false。
- `<x-nav-link>` 標籤內的內容會帶入到 nav-link.blade.php 所設定的 {{ $slot }}。

### 補充：attributes 與 props 的差異

attributes 代表 DOM 元素的屬性，例如 id、class、href ...等等。會在渲染成 html 文件時作為標籤的屬性輸出。

props(property複數縮寫)在自訂的元件中則是一種內部使用的項目，傳入後可以提供引入該 props 的 blade 進行運算處理。

### 補充：Reuseable Component 的 class 樣式設定

在 reuseable component 設定樣式，通常只會設定基本的 class，待 component 正式被嵌入到 blade 樣板時，可再根據版面需求加上 `class` 屬性，此時需要使用 $attributes 的方法進行合併。

範例：建立一個 Component: Card

```bash
php artisan make:component Card --view
```

方法一：使用 `merge()`

編輯 `resources/views/components/card.blade.php`

```php
<div {{ $attributes->merge(['class' => 'rounded-md board boarder-slate-300']) }}>
    {{ $slot }}
</div>
```

嵌入 Card 時，加上額外的 class

```php
<x-card class="mb-4">
  Something in slot
</x-card>
```

方法二：使用 `class()`

編輯 `resources/views/components/card.blade.php`

```php
<div {{ $attributes->class(['rounded-md board boarder-slate-300', 'actived' => $actived]) }}>
    {{ $slot }}
</div>
```

嵌入 Card 時，加上額外的 class

```php
<x-card class="mb-4">
  Something in slot
</x-card>
```

說明：

- 方法一可以合併多種 $attributes，使用上較具彈性
- 方法二則可針對 class 屬性進行操作，較為簡單且直覺。

更多內容可參考官方文件[Components](http://laravel.com/docs/12.x/blade#components)
