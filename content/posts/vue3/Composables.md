---
title: "Composables 筆記"
date: 2025-06-20 12:00:00 +0800
categories: 
  - vue3
tags:
  - vue3
---

Vue3 Composables 筆記

## 參考資料

- [Composables](https://vuejs.org/guide/reusability/composables.html)

## 說明

Composables 主要是利用 Vue3 的語法特性，將會重複使用的程式邏輯包裝成方法，達到可重用性的效果。

Composables 的實作可參考官網範例。

### Conventions and Best Practices

Composables 的規範與實踐

#### 命名 Naming

對 Composables 方法的命名通常以 camelCase 方式，並且以 'use' 為開頭，例如：useMouse(), useFetch()。

#### 方法參數 Input Arguments

Composeables 方法的參數可以接受

- 響應式參數(ref, reactive)
- computed
- getter
- 原始值(非響應式)

當 composables 有可能被重複使用時，必須確認使用者可能傳入響應式的參數值，並且為了方便提取響應式參數的值，`vue 3.3` 以後版本提供 `toValue()` 方法，可以方便安全地取出參數的值。例如傳入以下參數到 toValue()，均會取得 'abc'。

- ref('abc')
- () => 'abc'
- 'abc'

```ts
import { toValue } from 'vue'

function useFeature(input: MaybeRefOrGetter<string>) {

  const inputValue = toValue(input)
}
```

如果 Composables 方法需要使用到 ref 的響應性效果，請確保使用 watch() 監控 ref，或在 watchEffect() 中調用 toValue() 來正確進行監控。例如

```ts
function useFeature(input: MaybeRefOrGetter<string>) {
  watch(input, () => {
    //...
  });

  // or
  watchEffect(() => {
    console.log(toValue(input))
  });
}
```

#### 回傳值 Return Values

在 Composables 方法內的響應性項目大多使用 ref() 而非 reactive()，主要是在 component 內使用 composables 方法時，可以使用解構賦值方式從 composables 方法取得回傳值，並且仍然保持響應性。

```js
function useFeature() {
  const x = ref(0);
  const y = ref(0);

  return {
    x, y
  }
}

// 解構賦值後，x, y 仍然保持 ref 響應性
const { x, y } = useFeature();

```

因為 composables function 回傳的是一個非響應的 object `{ ... }`，因此不能直接將回傳的物件用在需要響應性的地方。

```ts
const feature = useFeature();

//以下並不具響應性
<span>{{ feature.x }}</span> //❌
<span>{{ feature.y }}</span> //❌
```

必須使用 `reactive()` 將回傳的 object 進行包裝才可具有響應性

```js
//使用 reactive() 包裝 composables 方法的回傳內容
const feature = reactive(useFeature());

// mouse.x is linked to original ref
console.log(mouse.x)

// 在 template 可使用 {{ mouse.x }} {{ mouse.y }} 來達成響應性渲染。
<span>{{ feature.x }}</span> //⭕
<span>{{ feature.y }}</span> //⭕
```

#### 副作用 Side Effects

在 Composables 方法可操作有副作用的程式內容，例如新增 DOM 事件監聽器或 fetch data，但是要注意以下規則：

如果您正在開發使用Server-Side Rendering (SSR) 的應用程序，請確保能對 DOM 產生副作用在生命週期中的 hook，例如必須在 onMounted() 之後才能對 DOM 進行操作。這些 hook 僅在瀏覽器中調用，因此您可以確保在其中對 DOM 的存取。

請確保在 onUnmounted() 中清除副作用。例如在 Composables 方法設置了一個 DOM 的 event listener，請確保在 onUnmounted() 的時候移除該 event listener，可參考官網的 useMouse() 的作法。另外也可以參考官網範例 useEventListener() 的作法也是一種不錯的選擇。官網範例連結[Mouse Tracker Example](https://vuejs.org/guide/reusability/composables.html#mouse-tracker-example)。

{{< alert type="info" >}}
官網的進階範例是進一步把單傳的事件註冊(useEventListener)與會產生 side effects 的方法(useMouse)拆開成 2 個獨立的 Composables 方法。然後再將 2 者結合，達到更好的可重複使用性和程式的職責管控。
{{</ alert >}}

#### 使用限制 Usage Restrictions

Composables 只能在 `<script setup>` 或 `setup() hook` 中同步調用。在某些情況下，您還可以在諸如 onMounted() 之類的 lifecycle hooks 中調用它們。

這些是 Vue 能夠確定當前活動組件實例的上下文。 訪問活動組件實例是必要的，以便：

1. 生命週期鉤子可以註冊到它。
2. 計算屬性和觀察者可以鏈接到它，以便在組件卸載時進行處置。

#### Extracting Composables for Code Organization

對重複的程式邏輯進行 Composeables 的提取不但有利於重複使用，也適用於組織良好的程式結構。隨著 components 複雜性的增加，可能會造成 components 過於龐大而無法追蹤或溯源。Composables API 為您提供了充分的靈活性，可以根據邏輯問題重新組織 Components 的程式碼，或分解成更小的函數：

```vue
<script setup>
import { useFeatureA } from './featureA.js'
import { useFeatureB } from './featureB.js'
import { useFeatureC } from './featureC.js'


const { foo, bar } = useFeatureA()
const { baz } = useFeatureB(foo)
const { qux } = useFeatureC(baz)
</script>
```

在某種程度上，您可以將這些提取的 Composables 視為可以彼此互相通訊的 component-scoped services。

例如以上範例，透過將不同 composables 方法回傳的內容傳遞給專職某個邏輯的 composables 方法進行溝通與重構。

#### 在 Options API 中使用 Composables Using Composables in Options API

請參考官網 [Using Composables in Options API](https://vuejs.org/guide/reusability/composables.html#using-composables-in-options-api)

## 小結

Composables 和一般通用函式庫主要的不同在於，一般通用函式庫是以原生的 JavaScript 撰寫，或是引用與包裝其他通用函式庫，製作出像是工具庫的概念。Composables 則主要是使用 vue 所提供的各種響應性方法與工具，對 vue 的 component 進行重構。
