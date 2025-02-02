---
title: "Django Tutorial 系列(3)"
date: 2024-10-16 16:40:41 +0800
categories: 
  - django
tags:
  - django
---

Django Tutorial Part 3

## 注意事項

因 jekyll 使用 Liquid 進行符號渲染，於 markdown 內使用 {\% 會有衝突，故在本 markdown 文件原始碼本文前後加上 {\% raw %} 與 {\% endraw %} 區塊來略過 Liquid 符號。

### app templates 目錄結構

1. 在 app 目錄建立 templates 目錄。e.g. `polls/templates`。
2. 在 templates 目錄下建立 app 為名的目錄。e.g. `polls/templates/polls`。因為 Django Template 機制的運作模式，是會先找到第一個吻合的 template 檔名，因此若不同的 app 有相同的 template 檔名，可能會載入不正確的 template，因此需要多建一層目錄作為 namespacing 來區分。

### template 範例

建立 `polls/templates/polls/index.html`

```html
<!-- 僅為 template 語法內容 -->
<!-- 請自行補上 html 完整結構標籤 -->

{% if latest_question_list %}
    <ul>
    {% for question in latest_question_list %}
        <li><a href="/polls/{{ question.id }}/">{{ question.question_text }}</a></li>
    {% endfor %}
    </ul>
{% else %}
    <p>No polls are available.</p>
{% endif %}
```

### views 載入 template

編輯 polls/views.py

```python
from django.http import HttpResponse
# import loader
from django.template import loader

from .models import Question

def index(request):
    latest_question_list = Question.objects.order_by("-pub_date")[:5]
    
    # 讀入 template 並設定 context
    template = loader.get_template("polls/index.html")
    context = {
        "latest_question_list": latest_question_list,
    }
    
    # Response 樣板
    return HttpResponse(template.render(context, request))
```

#### A shortcut: render()

使用 render() 來簡化 HttpResponse。

編輯 polls/views.py

```python
# import render
from django.shortcuts import render

from .models import Question

def index(request):
    latest_question_list = Question.objects.order_by("-pub_date")[:5]
    
    # 設定 context
    context = {"latest_question_list": latest_question_list}
    
    # 用 return render() 取代 return HttpResponse
    return render(request, "polls/index.html", context)
```

### Raising a 404 error

若要產生 http status 404，可使用`Http404()`函式。

編輯 polls/views.py

```python
# import Http404
from django.http import Http404
from django.shortcuts import render

from .models import Question

# ...
def detail(request, question_id):
    # 若查無 question 資料，則回應 http 404 
    try:
        question = Question.objects.get(pk = question_id)
    except Question.DoesNotExist:
        raise Http404("Question does not exist")
    return render(request, "polls/detail.html", {"question": question})
```

#### A shortcut: get_object_or_404()

Django 提供 `get_object_or_404()` 方法，當無法從 model 取得資料時，自動回應 http 404。

```python
# import get_object_or_404
from django.shortcuts import get_object_or_404, render

from .models import Question

# ...
def detail(request, question_id):
    question = get_object_or_404(Question, pk = question_id)
    return render(request, "polls/detail.html", {"question": question})
```

另外也有 `get_list_or_404()` 有類似的運作方式。

## Use the template system

template 可以取得後端 render 所傳入的 context 內容。使用 . 可以取得物件的屬性或方法。

編輯 polls/templates/polls/detail.html

```html
<h1>{{ question.question_text }}</h1>
<ul>
{% for choice in question.choice_set.all %}
    <li>{{ choice.choice_text }}</li>
{% endfor %}
</ul>
```

上面程式碼中，question.question_text 可以取得 question 的 question_text 欄位值。

question.choice_set.all 則會調用 question.choice_set.all() 的方法，該方法回傳一個可迭代 choice_set 集合。

## Removing hardcoded URLs in templates

在 templates 內容，可以使用輔助的`url`方法建立吻合 URLConf 名稱的超連結。

編輯 polls/templates/polls/index.html

```html
<li>
    <!-- url '<route-name>' <placeholder> ... -->
    <a href="{% url 'detail' question.id %}">{{ question.question_text }}</a>
</li>
```

以上即相當於

```html
<li>
    <a href="/polls/{{ question.id }}/">{{ question.question_text }}</a>
</li>
```

### Namespacing URL names

在多個 app 的情況下，可能有多個名稱為 detail 的路由，為了區分不同 app 的 detail，替每個 app 的 URLConf 設定 namespacing。

編輯 polls/urls.py

```python
# ...
# 加入 app_name
app_name = 'polls'

```

使用 {% url %} 即可加入 namespace。

編輯 polls/templates/polls/index.html

```html
<li>
    <!-- url '<route-name>' <placeholder> ... -->
    <a href="{% url 'polls:detail' question.id %}">{{ question.question_text }}</a>
</li>
```
