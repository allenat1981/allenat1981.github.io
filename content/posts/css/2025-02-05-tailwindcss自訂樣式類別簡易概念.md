---
title: "tailwindcss 自訂樣式類別(class)簡單概念"
date: 2025-02-05 11:00:00 +0800
categories:
  - css
tags:
  - tailwindcss
  - css
---

使用 tailwindcss，有時候需要自訂內建預設樣式或自訂樣式類別，tainwindcss 提供以下 layer：

- @layer base
- @layer components
- @layer utilities

以下簡易說明

## layer base

用途：定義 HTML 標籤 的全域樣式，類似於 normalize.css 或 CSS Reset。

優先級：最低，通常影響標籤 (h1, p, a 等)。

優點：

- 可以覆寫 Tailwind 的 preflight（預設基礎樣式）。
- 適用於需要全域樣式的情境。

缺點：

- 如果定義 .my-class { ... } 在 @layer base，其優先級會比 @layer components 低，可能導致無法正確覆寫樣式。

適用：適合用來設定 HTML 標籤的預設樣式，而非特定 class。

範例：

```css
@layer base {
  h1 {
    @apply text-4xl font-bold;
  }
  a {
    @apply text-blue-600 underline hover:text-blue-800;
  }
}
```

## layer components

用途：定義 可重用的 UI 元件（如按鈕、卡片）。

優先級：比 @layer base 高，但比 @layer utilities 低。

優點：

- 適合定義可重用組件，例如 .btn-primary, .card，可以避免每次都寫一堆 @apply。
- purge（樹搾）功能仍可正常運作。

缺點：

- 不能用來定義全域標籤樣式（應該放在 @layer base）。
- 如果想要更高優先級（例如覆蓋 Tailwind 的 utilities 類別），可能會有衝突。

適用：適合用來定義 UI 組件的 class，而非全域樣式或工具類別。

範例：

```css
@layer components {
  .btn {
    @apply px-4 py-2 bg-blue-500 text-white rounded;
  }
  .card {
    @apply p-6 bg-white shadow-lg rounded-lg;
  }
}
```

## layer utilities

用途：定義新的 工具類別（utilities classes），通常是功能性較強的樣式。

優先級：最高，會覆蓋 @layer base 和 @layer components。

優點：

- 最高優先級，適合定義如 .text-shadow 這類的功能性工具。
- 適用於擴展 Tailwind 內建的 utility 類別。

缺點：

- 過度使用可能會影響可維護性，讓 .btn 這類組件的樣式變得混亂。

範例：

```css
@layer utilities {
  .text-shadow {
    text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.2);
  }
  .bg-gradient {
    background: linear-gradient(to right, #4facfe, #00f2fe);
  }
}
```

## 自訂 UI 組件

若要使用 tailwindcss 內建項目自訂組件，則可以使用 @apply 將 tailwindcss 提供的預設樣式組合起來，形成 composition 樣式。

自訂 alert-box 的範例

```css
@layer components {
    .alert-box {
        @apply rounded-lg text-white p-4 my-4;    
    }

    .bg-info {
        @apply bg-blue-400;
    }
    
    .bg-warning {
        @apply bg-amber-400 text-emerald-900;
    }
    
    .bg-danger {
        @apply bg-red-400 text-black;
    }
}

```

### 補充：自訂 utility

若要將自訂的樣式也可以使用 @apply 組成 composition，則必須要先用 @utility 將自訂樣式建立成 utility。

例如修改以上範例：

```css
/* 使用 @utility 建立 alert-box */
@utility alert-box {
  @apply rounded-lg p-4 my-4;
}

/* 在 components 內的 class 即可使用 @apply alert-box */
@layer components {
    .alert-info {
        @apply alert-box bg-blue-400 text-white;
    }
    
    .alert-warning {
        @apply alert-box bg-amber-400 text-emerald-900;
    }
    
    .alert-danger {
        @apply alert-box bg-red-400 text-black;
    }
}

```

{{< alert type="notice" >}}
若是 UI 組件，通常還是會直接將 alert-box 作為類別而非 utility。
{{< /alert >}}
