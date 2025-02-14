---
title: "composer-dev 小知識"
date: 2024-12-31 12:05:00 +0800
categories: 
  - php
tags:
  - PHP
---

## autoload-dev

參考資料：[dump-autoload / dumpautoload](https://getcomposer.org/doc/03-cli.md#dump-autoload-dumpautoload)

在開發環境中，可能會需要使用到一些測試用的類別，為了方便使用與管理，可以替這些類別另外設定開發或測試時使用的命名空間。

例如在 composer.json 加入 "autoload-dev" 來指定測試類別專用的命名空間 BeyondTest，將其指向單元測試使用的測試類別。

```json
{
    "autoload": {
        "psr-4": {
            "Beyond\\": "app/src/"
        }
    },
    "autoload-dev": {
        "psr-4": {
            "BeyondTest\\": "app/tests/src/"
        }
    },
}
```

設定完成後使用 `composer dump-autolod` 更新 autoload，即可在開發與測試階段使用 BeyondTest 命名空間。

## 發佈 production

要注意的是若要發佈到 production 環境，則可以使用指令 `composer dump-autoload --no-dev` ，composer 會將 autoload-dev 的命名空間設定從 autoload 檔案中移除，避免測試的命名空間不小心部署到 production 環境。

> 可以在 `vendor\composer\autoload_psr4.php` 檔案內實際看到有沒有 `--no-dev` 的內容變化。

## 安裝 dev 使用的 package

參考資料：

- [install / i](https://getcomposer.org/doc/03-cli.md#install-i)
- [update / u / upgrade](https://getcomposer.org/doc/03-cli.md#update-u-upgrade)

在 dev 階段若要使用某些外部套件，則可以將該套件加入 `require-dev` 區塊。

例如若需要使用 laminas/laminas-diactoros 在測試階段模擬 psr-7 檔案上傳。

```bash
composer require --dev laminas/laminas-diactoros
```

同樣會將 laminas/laminas-diactoros 加入到專案的 vendor 目錄，提供給測試階段使用。

若要發佈到 production 環境，則可以使用：

```bash
composer update --no-dev
# 或者
composer install --no-dev
```

加上 `--no-dev` 參數，composer 會自動排除 autoload-dev 和 require-dev 的項目，讓 vendor 保持僅有 production 的 autoload 和相關套件。

## 結論

`composer dump-autload` 在沒有加上 `--no-dev` 的情況下，會依照上一次 `composer update` 或 `composer install` 時，是否有加上 `--no-dev` 作為輸出 autoload 內容的依據。

簡單來說若前一次的 install 或 update 有 `--no-dev`，則 dump-autoload 就會有 `--no-dev`；反之則沒有。

{{< alert type="notice" >}}
若不太確定，則建議 `composer update` 或 `composer dump-autoload` 時，直接加上 `--dev` 或 `--no-dev` 來明確指定是否為 dev 環境。
{{< /alert >}}
