---
title: "Django Tutorial 系列(6)"
date: 2024-10-23 09:00:00 +0800
categories: 
  - django
tags:
  - django
---

# Writing your first Django app, part 6

靜態檔案(static files)設定。

在多個 App 的情況下，要替每個 App 處理 static files 變得較為麻煩。

`django.contrib.staticfiles` 機制可以集結所有 App 使用的 static files 放置在指定的地方，讓 static files 可以更輕易部署在 production 環境。

## Customize your app’s look and feel

首先**在 polls 建立目錄 static**，Django 將如同 templates 一樣在該目錄找到 static files。

Django 的 STATICFILES_FINDERS 設定包含一組 finders 用來尋找各種 static files 資源。其中一項預設的 AppDirectoriesFinder 會在各 INSTALLED_APP 設定的子目錄下的 static 目錄尋找 static files，例如一開始建立的 `polls/static`。包含 admin 也是同樣的結構。

同樣在 polls/static 建立 polls 目錄，並加入 style.css 檔案。完整的檔案路徑為 `polls/static/polls/style.css`。即可讓 AppDirectoriesFinder 找到此靜態資源。

*如同 templates 機制一樣，為了區分不同 App 的 static files，因此在 app/static 需多加一個子目錄作為區隔。*

接著撰寫簡易的 css 樣式進行測試：

編輯 `polls/static/polls/style.css`

```css
li a {
    color: green;
}
```

接著在 template 頁面載入 static files，以 index.html 為例：

編輯 `polls/templates/polls/index.html

```html
<!-- 在適當的地方加入以下內容 -->
{% load static %}
<link rel="stylesheet" href="{% static 'polls/style.css' %}">
```

`load static` 會載入需要的 module。並用 `{% static %}` 在 html tags 內產生 static file 的 URL 絕對路徑。

## Adding a background-image

接著在 static/polls 下建立一個目錄 images，用來放置所需要的圖檔。完整目錄為 `polls/static/polls/images`，並放置一張名稱為 background.jpg 的圖檔。

接著將背景圖片加入 style.css，編輯 `polls/static/polls/style.css`

```css
body {
    background: white url("images/background.jpg") no-repeat;
}
```

`{% static %}` 標籤不適用於 Django 所讀入的 static files，所以若要在 static files 內相互連接資源，必須使用相對路徑來處理。就如同本範例中的 style.css 所設定。如此你就可以僅更改 prject 的 `settings.py` 的 STATIC_URL 參數，而不需要去變更 static files 內所涉及到的所有檔案路徑。
