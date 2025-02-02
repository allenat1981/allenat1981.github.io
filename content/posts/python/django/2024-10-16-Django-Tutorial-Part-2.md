---
title: "Django Tutorial 系列(2)"
date: 2024-10-16 16:40:40 +0800
categories: 
  - django
tags:
  - django
---

Django Tutorial Part 2

## 說明

1. 安裝 mariadb 和 mysqlclient(驅動)
2. 初探 migrations 機制
3. Django Admin 管理介面

## 安裝 mariadb

安裝必要套件

Rocky Linux 必須先啟用 crb 才可以安裝 mysql-devel。

```bash
# 安裝 yum-utils
sudo dnf install yum-utils

# 啟用 crb
sudo dnf config-manager --enable crb
```

安裝 mysqlclient 必要系統套件。

```bash
# 安裝系統必要套件
sudo dnf install python3-devel mysql-devel pkgconfig
```

安裝 mysqlclient。

```bash
# 安裝 mysqlclient
pip install mysqlclient
```

## 設定專案資料庫連線

開啟 `mysite/settings.py`

```conf
# 可同時設定語系/時區/資料庫連線
#...

# 設定語系
LANGUAGE_CODE = 'zh-hant'

# 設定時區
TIME_ZONE = 'Asia/Taipei'

# 設定資料庫(mysql/mariadb)
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': '<db-name>',
        'USER': '<user>',
        'PASSWORD': '<password>',
        'HOST': '<db-host>',
        'PORT': '<db-port>'
    }
}
```

### 執行 django 預設的 migrate

`migrate` 會根據 `settings.py` 的 `INSTALLED_APPS` 區塊在資料庫產生相關的資料表。

首次執行 migrate 來產生 django 預設提供的 apps 之資料表。

```bash
# 工作目錄 mysite
python manage.py migrate
```

執行成功，可在資料庫中檢視已產生的資料表。

## 建立 App:polls 的 model

預設 app 目錄中有一個 models.py 檔案，用來建立 app 所使用的資料庫 model。

```python
# Tutorial 的範例程式碼
from django.db import models


class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField("date published")


class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)
```

1. model 均繼承 django.db.models.Model 類別。
2. model 內的每個 field 均為 Field 類別的實例。
3. model 類別中每個 Field 的資料成員名稱即為資料表中的欄位名稱。
4. ForeignKey 可用來替資料表建立外鍵關聯。

### 將 polls 註冊至專案的 apps

編輯 `mysite/settings.py`

```python
# ...

# 將 polls.apps.PollsConfig 加入 INSTALLED_APPS
INSTALLED_APPS = [
    "polls.apps.PollsConfig",
    "django.contrib.admin",
    "django.contrib.auth",
    "django.contrib.contenttypes",
    "django.contrib.sessions",
    "django.contrib.messages",
    "django.contrib.staticfiles",
]
```

#### makemigrations 建立 migration 檔案

執行 `makemigrations` 指令建立 polls 的 migration。

```bash
# python manage.py makemigrations <app-name>
python manage.py makemigrations polls

# 成功後將產生 polls/migrations/0001_initial.py
```

`makemigrations` 會告知 Django 關於 models 產生變化，並將這些變化建立成 migration 檔案儲存在 app 的 `migrations` 目錄。

#### sqlmigrate 檢視 migration 將進行的 SQL 語句

在執行 migrate 指令更新資料庫結構前，可以透過 `sqlmigrate` 指令來檢視 migration 將在資料庫執行的 SQL 語句。

```bash
# python manage.py sqlmigrate <app-name> <migration-no>
python manage.py sqlmigrate polls 0001
```

請注意！sqlmigrate 僅輸出即將執行的 SQL 語句。必須執行 migrate 指令才會真正去異動資料庫。

#### migrate 異動資料庫

```bash
python manage.py migrate
```

`migrate` 指令會執行之前尚未執行的 migrations 檔案，進行資料庫的異動。

補充：在專案資料庫中有一個 django_migrations 資料表專門用來記錄 migrations 是否被執行過。

### Migrations 概略說明

Migrations 機制可以用來協助專案更新資料庫結構，而不需要刪除資料庫或資料表。在更新資料庫結構也不會造成資料遺失。

Migrations 機制主要的步驟如下：

1. 修改 models (編輯 models.py)。
2. 執行 `python manage.py makemigrations` 建立 migrations 檔。
3. 執行 `python manage.py migrate` 進行資料庫結構更新。

## django python shell

若要在 python shell 中對 django 專案進行指令化的操作，可以透過 `python manage.py shell` 開啟 shell，將會自動載入專案的 settings.py 的設定。

```bash
python manage.py shell
```

### model 的 __str__() 方法

在 model 類別中複寫 __str__ 方法，類似其他語言的 toString() 方法。

```python
# 開啟 polls/models.py
# 在 model 類別中加入 __str__ 方法
class Question(models.Model):
    # ...
    def __str__(self):
        return self.question_text

class Choice(models.Model):
    # ...
    def __str__(self):
        return self.choice_text
```

沒有異動到 model 的結構，因此不需要進行 migration。

## Django Admin

Django 提供常用的後台管理者介面。

### 建立 admin user

```bash
python manage.py createsuperuser

# 依照提示輸入帳號、email、密碼...
```

### 啟用網站伺服器進入登入頁面

```bash
python manage.py runserver
```

預設管理者介面位址: http://127.0.0.1/admin

### 加入 poll app 管理功能

編輯 `<app>/admin.py`

```python
# 編輯 polls/admin.py
# 將 Question 註冊到 admin 介面
from django.contrib import admin
from .models import Question

admin.site.register(Question)
```

重新進入管理者介面，即可看到 Question 編輯功能。
