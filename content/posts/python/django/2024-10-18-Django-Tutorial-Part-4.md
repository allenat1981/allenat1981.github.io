---
title: "Django Tutorial 系列(4)"
date: 2024-10-18 09:10:00 +0800
categories: 
  - django
tags:
  - django
---

Django Tutorial Part 4

Writing your first Django app, part 4

## Write a minimal form

基本的表單(form)寫法

編輯 `polls/templates/polls/detail.html`

```html
<form action="{% url 'polls:vote' question.id %}" method="post">
{% csrf_token %}
<fieldset>
    <legend>
        <h1>{{ question.question_text }}</h1>
    </legend>
    {% if error_message %}
    <p>
        <strong>{{ error_message }}</strong>
    </p>
    {% endif %}
    {% for choice in question.choice_set.all %}
        <input id="choice {{ forloop.counter }}" 
            type="radio" 
            name="choice" 
            value="{{ choice.id }}">
        <label for="choice{{ forloop.counter }}">{{ choice.choice_text }}</label><br>
    {% endfor %}
</fieldset>
<input type="submit" value="submit">
</form>
```

1. 在 form 內加上 `{% csrf_token %}` 來避免跨域攻擊。
2. `error_message` 用來判斷表單是否驗證錯誤，並輸出錯誤內容。
3. `forloop.counter` 代表 for loop 運行次數。

接著修改 views.py 的 vote()

編輯 polls/views.py

```python
# 加入 import 項目
# import ...
from django.db.models import F
from django.http import HttpRequest, HttpResponse, HttpResponseRedirect
from django.urls import reverse
from .models import Choice, Question

def vote(request: HttpRequest, question_id: int) -> HttpResponse:
    question = get_object_or_404(Question, pk = question_id)
    try:
        selected_choice = question.choice_set.get(pk = request.POST["choice"])
    except (KeyError, Choice.DoseNotExist):
        return render(
            request,
            "polls/detail.html",
            {
                "question": question,
                "error_message": "You didn't select a choice."
            }
        )
    else:
        selected_choice.vote = F("votes") + 1
        selected_choice.save()
        return HttpResponseRedirect(
            reverse("polls:results", args = (question.id, ))
        )
```

1. 透過 `request.POST[<input-name>]` 可取得表單的值。
2. 若`request.POST[<input-name>]` 的值不存在，則觸發 KeyError 錯誤，並由 except 區塊捕捉。
3. `F("votes) + 1` 可將 selected_choice.vote 計數加一。使用 F() 可以避免競爭條件(race conditions)，請參閱 [instructs the database](https://docs.djangoproject.com/en/5.1/ref/models/expressions/#avoiding-race-conditions-using-f)。請參考[補充：F() 注意事項](#補充f-注意事項)
4. `HttpResponseRedirect()` 回應一個進行重新導向的 HttpResponse。
5. `reverse()` 用來將 route name 轉為正式的 url 路徑。

用戶使用表單投完票後，將會轉址到 polls:results 頁面，接著繼續完成後續的程式與樣板。

修改 views 的 results()：

編輯 `polls/views.py`

```python
def results(request: HttpRequest, question_id: int) -> HttpResponse:
    question = get_object_or_404(Question, pk = question_id)
    return render(request, "polls/results.html", {
        "question": question
    })
```

建立 results 頁面：

建立 `polls/templates/polls.results.html`

```html
<h1>{{ question.question_text }}</h1>
<ul>
{% for choice in question.choice_set.all %}
    <li>{{ choice.choice_text }} -- {{ choice.votes }} vote{{ choice.votes | pluralize }}</li>
{% endfor %}
</ul>
<a href="{% url 'polls:detail' question.id %}">Vote again?</a>
```

### 補充：F() 注意事項

**根據 Gemini 表示 F() 只能輔助，不能完全解決 race conditions。**

F() 是直接在資料庫進行 UPDATE 操作，以 vote 例子來說，因為是直接在資料庫進行 vote + 1，而非把 vote 查詢出來存在 memory 後，接著再 memory 中對 vote + 1，再寫回資料庫，藉此避免不同 thread 造成的 race conditions。

**F() 對 Model.save() 的持續作用。**

考慮以下範例

```python
reporter = Reporters.objects.get(name = "Tintin")
reporter.stories_filed = F("stories_filed") + 1
reporter.save()

reporter.name = "Tintin Jr."
reporter.save()
```

如上所示，`reporter.save()` 被呼叫 2 次，因此最後的結果 stories_file 值會被遞增 2 次(+2)，代表若 stories_field 本來為 1，執行完上面程式後結果將為 3。主要原因是在 reporter.save() 之後，F("stories_field") + 1 的效果仍然維持。若要在寫入資料庫後就清除 F() 的效果，則需要用 `refresh_from_db()` 來取代 save()。

## Use generic views: Less code is better

django 提供名為 "generic views" 的機制，用簡短的方式來處理從資料庫取出資料後渲染到視圖。

例如 `ListView` 和 `DetailView` 將"展示物件的列表"以及"展示指定類型物件的細節"的概念分別抽象化。

透過 generic views 的步驟，可以將 polls 程式簡化：

1. 調整 URLConf
2. 刪除不需要的視圖
3. 引進基於 Django's generic views 的新視圖。

通常在正式開發中，會在一開始就評估 generic views 是否適用，而不是在完成複雜的工作後才進行簡化。

### Amend URLconf

調整 URLConf：

編輯 `polls/urls.py`

```python
from django.urls import path
from . import views

app_name = "polls"

urlpatterns = [
    # 將 index 改為 IndexView.as_view()
    # 原本 <int:question_id> 則改為 <int:pk>
    # details 和 results 同樣作法
    path("", views.IndexView.as_view(), name = "index"),
    path("<int:pk>/", views.DetailView.as_view(), name = "detail"),
    path("<int:pk>/results/", views.ResultsView.as_view(), name = "results"),
    path("<int:question_id>/vote/", views.vote, name = "vote"),
]
```

### Amend views

修改 views 對應 URLConf 的程式碼：

編輯 `polls/views.py`

```python
# 加入 import
from django.views import generic

# 將方法 index, details, results
# 改為類別 IndexView, DetailView, ResultsView
# 並分別繼承相應的 generic.ListView 或 generic.DetailView

class IndexView(generic.ListView):
    template_name = "polls/index.html"
    context_object_name = "latest_question_list"

    def get_queryset(self):
        """Return the last five published questions."""
        return Question.objects.order_by("-pub_date")[:5]

class DetailView(generic.DetailView):
    model = Question
    template_name = "polls/detail.html"

class ResultsView(generic.DetailView):
    model = Question
    template_name = "polls/results.html"

```

不同的 generic view 需要定義不同的操作，例如

- `generic.DetailView` 需要設定屬性 `model`。
- `generic.ListView` 則需要定義屬性 `context_object_name` 與方法 `get_queryset()`。
- 二者均可自訂 template_name(亦可使用預設的 template_name，請見說明)。

DetailView 則會在視圖的 html 檔中，自動注入屬性 model 所設定的名稱，例如 `model = Question`，則可使用 question 存取屬性。

請參照 [https://docs.djangoproject.com/en/5.1/intro/tutorial04/#amend-views](https://docs.djangoproject.com/en/5.1/intro/tutorial04/#amend-views)
