---
title: "Laravel Blade Component 的 Data Properties 和 Attributes 的差別"
date: 2025-03-13 10:30:00 +0800
categories: 
  - chat-gpt-helper
tags:
  - laravel
---

在 Laravel 的 Blade 樣板引擎中，當建立 **Component（元件）** 時，`Data Properties` 和 `Attributes` 有一些重要的區別：

---

## Data Properties

這些是你在 **元件類別**（`Component Class`）中定義的 **公開屬性**，可以在 Blade 模板中直接使用。這些屬性通常用來傳遞資料給元件，以便在視圖中呈現。

### Data Properties範例

在 `app/View/Components` 目錄下建立一個元件類別，例如 `Alert.php`：

```php
namespace App\View\Components;

use Illuminate\View\Component;

class Alert extends Component
{
    public string $type;
    public string $message;

    public function __construct(string $type, string $message)
    {
        $this->type = $type;
        $this->message = $message;
    }

    public function render()
    {
        return view('components.alert');
    }
}
```

#### 建立 Blade 元件

`resources/views/components/alert.blade.php`：

```blade
<div class="alert alert-{{ $type }}">
    {{ $message }}
</div>
```

#### 在 Blade 模板中使用 Data Properties

```php
<x-alert type="success" message="操作成功！" />
```

### Data Properties 重點

- `type` 和 `message` 是 **Data Properties**，它們來自 `Alert` 類別的建構函式。
- Blade 模板內可直接使用這些變數（`$type`、`$message`）。
- 這些變數在建立元件時，透過 HTML 屬性傳入（例如 `type="success"`）。

---

## Attributes

Attributes 是指 Blade 元件內的 **HTML 屬性**，例如 `class`、`id` 或其他自定義的屬性，它們不會自動變成 Data Properties，但可以使用 `$attributes` 物件來操作。

### Attributes 範例

修改 `alert.blade.php`：

```php
<div {{ $attributes->merge(['class' => "alert alert-{$type}"]) }}>
    {{ $message }}
</div>
```

#### 在 Blade 模板中使用 Attributes

```php
<x-alert type="warning" message="警告訊息！" class="font-bold" id="custom-alert" />
```

在渲染後，HTML 結果如下：

```html
<div class="alert alert-warning font-bold" id="custom-alert">
    警告訊息！
</div>
```

### Attributes 重點

- `$attributes` 變數用來 **處理 HTML 屬性**，它包含所有未被定義為 **Data Properties** 的屬性。
- `$attributes->merge(['class' => "alert alert-{$type}"])` 用來合併 CSS 類別。
- 這允許開發者靈活地在使用元件時 **新增或覆蓋 HTML 屬性**。

---

## 總結

| **項目**         | **Data Properties（資料屬性）** | **Attributes（屬性）** |
|----------------|------------------|----------------|
| **來源**        | 來自 `Component` 類別 | 來自 HTML 屬性 |
| **使用方式**    | 直接作為變數使用，如 `$type` | 透過 `$attributes` 操作 |
| **作用**        | 用來傳遞特定的資料（通常是內容） | 用來設定 HTML 標籤屬性（如 `class`、`id`）|
| **擴充性**      | 只能在元件類別中定義的屬性傳入 | 任何未定義的屬性都可用 `$attributes` 取得 |

當你需要傳遞資料並在元件內部邏輯中使用時，應該使用 **Data Properties**；當你希望支援 **HTML 屬性傳遞（如 class, id）** 或允許使用者自訂額外屬性時，應該使用 **Attributes**。

這樣的設計讓 Laravel 的 Blade 元件更具靈活性，也讓開發者更方便地管理樣式與行為。
