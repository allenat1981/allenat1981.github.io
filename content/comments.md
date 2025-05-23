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

在 \`\`\` 宣告區塊可用 `{}` 加上不同的標記，來替程式碼區塊加上不同的效果。標記之間可用 `,` 分開。

範例：

\`\`\`PHP {linenos=true, hl_Lines="1 3 5"}

### 加入行號

加上標記 `{linenos=true}`。

範例：

\`\`\`PHP {linenos=true}

### 加入 highlight

要在指定的行號加入 highlight，可加入 `{hl_Lines="行號"}`。行號可用空白分開，也可用 `-` 連接，例如 `10-15` 行。

範例：

\`\`\`PHP {hl_Lines="1 3 5 10-15"}

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

## 新增文章無法產出內容

若新增文章無法在頁面上看到，請先檢查 date 是否為目前時間之後，hugo 會根據該欄位決定是否產出。也就是說若 date 設定的時間還沒到，hugo 會略過該文章。

若 date 沒有問題，但仍然沒有辦法即時在本機端看到新增的文章，此時可以嘗試加上 `--disableFastRender` 停用快取。

```bash
hugo server -D --disableFastRender
```
