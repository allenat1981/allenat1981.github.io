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

## 程式碼區塊

### 加入行號

若要在程式碼區塊加入行號，可在程式碼區塊宣告的三個 \` 符號前加上 `{linenos=true}`。  
例如

\`\`\`PHP {linenos=true}

## 跳脫 hugo shortcode

若要跳脫 hugo 的 shortcode，可加上 `/* ... */`，例如 `{{</*/* ref "..." */*/>}}`。

## 超連結語法

請參考[Links and cross references](https://gohugo.io/content-management/cross-references/)

若要建立 markdown 之間的超連結，則可使用以下語法

```markdown
[Comments]({{</* ref "/comments" */>}})
```

請注意！ "/" 在 hugo 代表以 `content/` 目錄為超連結檔案的根目錄，故以上範例連結至檔案`<專案目錄>/contents/comments.md`。

若要加入錨點 section，則在後面加上 `#section`

```markdown
[Comments]({{</* ref "/comments#section" */>}})
```

若是在同一篇文章內的錨點 section，則直接使用

```markdown
[Comments]({{</* ref "#section" */>}})
```

注意事項：

若章節名稱包含大寫字母，轉為錨點名稱時，會轉為小寫字母。

若章節名稱包含空白(space)，則需使用`-`符號替代空白處，例如

```markdown
[自訂 alert 文字區塊]({{</* ref "#自訂-alert-文字區塊" */>}})
```

範例：[自訂 alert 文字區塊]({{< ref "#自訂-alert-文字區塊" >}})

## 自訂 alert 文字區塊

使用 `{{</* alert type="[type]" title="[title]" */>}}`

{{< alert type="info" title="Info" >}}
資訊
type="info"  
`Code Style`
{{< /alert >}}

{{< alert type="notice" title="Notice" >}}
提醒
type="notice"  
`Code Style`
{{< /alert >}}

{{< alert type="warning" title="Warning" >}}
警告
type="warning"  
`Code Style`
{{< /alert >}}

{{< alert type="danger" title="Danger" >}}
危險
type="danger"  
`Code Style`
{{< /alert >}}
