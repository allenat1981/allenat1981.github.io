---
title: "Comments"
date: 2025-02-01 20:45:00 +0800
---

## 文件書寫風格

### 靜態方法與成員方法的區別

有時候在文章內不易區別類別的靜態方法和成員方法，故用以下方法區分。

- 靜態方法：`ClassName::MethodName`
- 成員方法：`$ClassName::MethodName`

{{< alert type="info" >}}
成員方法會在類別名稱前面加上 `$`，概念是來自 PHP 的 variable 宣告需要在變數名稱前面加讓 `$`，故用 `$ClassName` 表示 ClassName 類別的 instance。
{{< /alert >}}

## 跳脫 hugo shortcode

若要跳脫 hugo 的 shortcode，可加上 `/* ... */`，例如 `{{</*/* ref "..." */*/>}}`。

## 超連結語法

請參考[Links and cross references](https://gohugo.io/content-management/cross-references/)

若要建立 markdown 之間的超連結，則可使用以下語法

```markdown
[Comments]({{</* ref "/comments" */>}})
```

請注意！ "/" 在 hugo 代表以 `content/` 目錄為超連結檔案的根目錄，故以上範例連結至檔案`<專案目錄>/contents/comments.md`。

## 自訂 alert 文字區塊

使用 `{{</* alert type="[type]" title="[title]" */>}}`

{{< alert type="info" title="Info" >}}
資訊
type="info"
{{< /alert >}}

{{< alert type="notice" title="Notice" >}}
提醒
type="notice"
{{< /alert >}}

{{< alert type="warning" title="Warning" >}}
警告
type="warning"
{{< /alert >}}

{{< alert type="danger" title="Danger" >}}
危險
type="danger"
{{< /alert >}}
