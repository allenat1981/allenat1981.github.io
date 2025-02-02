---
title: "Django Tutorial 系列(5)"
date: 2024-10-18 17:10:00 +0800
categories: 
  - django
tags:
  - django
---

Writing your first Django app, part 5

Django 的自動化測試

## Writing our first test

Django 預設的測試檔案為 tests.py，撰寫第一個測試案例：

編輯 polls/tests.py

```python
import datetime

from django.test import TestCase
from django.utils import timezone

from .models import Question

class QuestionModelTests(TestCase):
    def test_was_published_recently_with_future_question(self):
        """
        was_published_recently() returns False for questions whose pub_date is in the future.
        """
        time = timezone.now() + datetime.timedelta(days = 30)
        future_question = Question(pub_date = time)
        self.assertIs(future_question.was_published_recently(), False)
```

測試案例撰寫完成後，進行測試

```bash
python manage.py test polls
```

以上說明

- `manage.py test polls` 會運行 polls 的測試案例。
- 測試案例為 `django.test.TestCase` 的子類別。
- 將會建立一個專門用來測試的資料庫。
- 測試方法必須以 `test` 作為方法名稱的前綴。
- 上述的測試案例將會建立一個 Question 實例，並將 pub_date 設定為 30 日後。
- 在測試方法內透過 `assertIs()` 方法對要測試的內容進行斷言。

以上簡述測試流程，tutorial 內關於 測試-修正-測試-... 的流程不進行筆記。

## Test a view

對 view 進行測試，我們會希望就像在瀏覽器的環境下進行測試一樣。

### The Django test client

Django 提供測試用的 `Client` 類別，用來模擬使用者與 view 的程式碼之間的互動。Client 可以適用在 tests.py 或 shell。

在 shell 環境中測試：

```bash
# 載入 manage.py 的 python shell
python manage.py shell
```

```python
# 請在 python shell 內進行以下操作

# 在 shell 載入測試環境
from django.test.utils import setup_test_environment
setup_test_environment()
```

`setup_test_environment()` 將會載入 template renderer 協助我們針對 response.context 進行操作與測試。

接著 import 測試用的 Clinet：

```python
from django.test import Client
client = Client()
```

載入就緒後，即可模擬 client 對 view 發出 request，並取得 response 相關資訊

```python
# 對 / 發出 request

response = client.get("/")
# 因為沒有設定 / 的 view，所以會返回
# Not Found: /

# 查詢 http status code
response.status_code
# 404

# 接著對 polls:index 發出 reuqest
from django.urls import reverse
response = client.get(reverse("polls:index"))

# 順利得到回應，即可調閱相關屬性內容

response.status_code
# 200

response.content
# 輸出 html 原始碼

response.context["latest_question_list]
# 輸出之前練習設定在 context 的 latest_question_list
```

透過 `Client` 可以更方便的對 view 的 reponse 與 context 進行測試。
