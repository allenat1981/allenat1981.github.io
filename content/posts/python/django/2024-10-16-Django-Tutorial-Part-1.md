---
title: "Django Tutorial 系列(1)"
date: 2024-10-16 16:40:39 +0800
categories: 
  - django
tags:
  - django
---

# Django Tutorial Part 1

## 說明

1. 安裝 Django。
2. 建立 Django 專案。
3. 建立範例 app: Polls。

## 建議

可在 venv 環境運行。

## 安裝 Django

```bash
# 透過 pip 安裝 django
pip install django

# 檢視已安裝項目
pip list

# 檢視 django 版本
python -m django --version
```

## 建立 django project

```bash!
# 建立名稱為 mysite 的 django project
# django-admin startproject <project-name>
django-admin startproject mysite
```

### 運行本機開發用網站伺服器

```bash
# 工作目錄 ~/py/django/mysite
python manage.py runserver
```

運行成功後，開啟瀏覽器輸入 http://127.0.0.1:8000 即可顯示 django 預設首頁。

## 建立範例 app: polls

```bash
# 建立 Django App
# python manage.py startapp <app-name>
python manage.py startapp polls
```

建立後會產生 polls 目錄結構。

### 在 views.py 處理 HttpResponse

polls/views.py 類似 controller，用來設計處理路由的函式。

編輯 polls/views.py

```python
from django.http import HttpResponse

# 建立 index 函式用來處理 index 的 HttpResponse
def index(request):
    return HttpResponse("Hello world")
```

### 建立 polls 的 URLConf

建立 polls/urls.py 檔案作為 polls 的 url config(URLConf)。

```python
from django.urls import path
from . import views

# 建立 urlpatterns 用來設定路由表
urlpatterns = [
    path("", views.index, name = "index"),
]
```

path 的三個參數：

1. 路由的路徑 "" 或 "/"。
2. 設定負責處理路由的 HttpResponse 函式。此處為匯入之views.index。
3. 路由的名稱。

#### 將 polls/urls.py 註冊到專案

編輯 mysite/urls.py

```python
# 異動的部分
# 1. django.urls 匯入 include
# 2. 將 polls/ 加入 urlpatterns
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path('polls/', include("polls.urls")),
    path('admin/', admin.site.urls),
]
```

### 測試 polls

```bash
# 啟動測試伺服器
python manage.py runserver
```

連線到 http://127.0.0.1:8000/polls，檢視是否有輸入 views.index 內容。

## 結論

1. `django-admin startproject <project-name>`建立專案。
2. 在專案目錄用 `python manage.py startapp <app-name>` 建立 app。
3. 在 app 目錄的 views.py 建立 HttpResponse 的處理函式。
4. 在 app 目錄建立 urls.py 檔案，用來設定 app 的URLConf。
5. 在專案目錄的 urls.py 註冊 app 的 URLConf。
6. `python manage.py runserver` 測試頁面。
