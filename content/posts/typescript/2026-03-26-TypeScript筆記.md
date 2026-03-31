---
title: "TypeScript 筆記"
date: 2026-03-26 15:00:00 +0800
categories: 
  - TypeScript
tags:
  - TypeScript
  - JavaScript
---

TypeScript 的一些心得筆記

## 使用 ! 強制略過 null 檢查

使用 `document.getElementXXX` 的情境下，IDE 預設會需要使用者自行檢查，否則會噴警告，例如

```TypeScript
const inputEl = document.getElementById('user-name');

// 若沒有以下這行，IDE 會噴警告
if (inputEl === null) {
    throw new Error('element user-name not found.');
}

console.log(inputEl.value);

```

若保證前端 document 含有該 element，則可以使用 `!` 來略過檢查步驟

```TypeScript
// 在最後加上 !，告訴 TS 元素 user-name 的存在與否由 developer 自行負責。
const inputEl = document.getElementById('user-name')!;

// 可以略過此檢查
// if (inputEl === null) {
//     throw new Error('element user-name not found.');
// }

// 但若 inputEl 不存在，則會噴 Runtime Error
console.log(inputEl.value);
```

## 使用 ?? 針對 null 和 undefined 設定填充值

JavaScript 有 falsy 的概念，以下狀態在進行 if(boolean) 判斷時會視為 false

- false
- 0 數字 0
- '' 空字串
- null
- undifined
- NaN 非數字(Not a Number)
- document.all

若想要確認值會被判定為 true or false，通常擇一以下二種方法

```TypeScript
// 使用 !!
const result = !!""; //空字串 => false

// 使用 Boolean() 方法
const result = Boolean(0); // result => false

```

有時需要針對變數為 null 或 undefined 時，給予填充值，可以使用 ??，用法如下

```TypeScript
let str = null; //或者 undefined

const message = str ?? 'Nothing to say'; //message => 'Nothing to say'

// ?? 僅可用來判斷 null 和 undefined，不包含 falsy
// 以下為錯誤的例子

let str = '';

const message = str ?? 'Noting to say'; //message => ''(空字串)

// 若要對 falsy 給予填充值

let str = '';

const message = str || 'Nothing to say';
// 或者用 !! 將 str 轉 boolean 再使用 ... ? ... : ... 三元運送子
const message = !!str ? str : 'Nothing to say';

```
