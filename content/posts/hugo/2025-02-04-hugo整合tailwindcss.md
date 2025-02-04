---
title: "hugo 整合 tailwindcss"
date: 2025-02-04 17:30:00 +0800
categories: 
    - hugo
tags:
    - hugo
    - tailwindcss
---

hugo + PaperMod + tailwindcssV4

## 安裝 tailwindcss

```bash
# 工作目錄: hugo 專案根目錄

# 安裝 tailwindcss @tailwindcss/cli
npm install --save-dev tailwindcss @tailwindcss/cli
```

## 設定 hugo.yaml

編輯 hugo.yaml

```yaml
build:
  cachebusters:
  - source: layouts/.*
    target: css
```

## 加入 css 與資源樣板

在 assest/css 加入 main.css 檔，內容如下

```css
@tailwind base;
@tailwind components;
@tailwind utilities;

@import "tailwindcss";
```

根據 PaperMod 的 [Custom Head / Footer](https://github.com/adityatelange/hugo-PaperMod/wiki/FAQs#custom-head--footer)，需新增 `layouts/partials/extend_head.html` 來將 tailwindcss 加載到 PaperMod 的 head。

{{< warning >}}
若沒有套用 theme，則預設是新增 `layouts/_default/baseof.html`。
{{< /warning >}}

```html
<!-- tailwindcss -->
{{ with resources.Get "css/main.css" }}
  {{ $opts := dict "minify" true }}
  {{ with . | css.TailwindCSS $opts }}
    {{ if hugo.IsDevelopment }}
      <link rel="stylesheet" href="{{ .RelPermalink }}">
    {{ else }}
      {{ with . | fingerprint }}
        <link rel="stylesheet" href="{{ .RelPermalink }}" integrity="{{ .Data.Integrity }}" crossorigin="anonymous">
      {{ end }}
    {{ end }}
  {{ end }}
{{ end }}
```

## 測試

以上設定完成後，使用 hugo 或 hugo server 指令，則可以在編輯 layouts/ 目錄下的檔案時，由 tailwindcss 產出 main.css。
