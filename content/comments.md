---
title: "Comments"
date: 2025-02-01 20:45:00 +0800
---

## 跳脫 hugo shortcode

若要跳脫 hugo 的 shortcode，可加上 `/* ... */`，例如 `{{</*/* ref "..." */*/>}}`。

## 超連結語法

請參考[Links and cross references](https://gohugo.io/content-management/cross-references/)

若要建立 markdown 之間的超連結，則可使用以下語法

```markdown
[Comments]({{</* ref "/comments" */>}})
```

請注意！ "/" 在 hugo 代表以 `content/` 目錄為超連結檔案的根目錄，故以上範例連結至檔案`<專案目錄>/contents/comments.md`。
