---
title: "多版本 PHP composer 設定"
date: 2024-12-31 12:15:00 +0800
categories: 
  - php
tags:
  - PHP
  - Archive
---

{{< alert type="warning" >}}
此文章是專案時的筆記，建議到官方網站檢視相關設定。
{{< /alert >}}

本文件說明在 Windows 環境下多版本 PHP 搭配 composer 時的設定。

## 參考資料

- [Composer Official Website](https://getcomposer.org/)

## 說明

進行本文件的實作前假設已經完成以下項目：

1. 下載並設定開發環境所使用各版本的 PHP。
2. 設定好網站伺服器(apache)以及要 VirtualHost。
3. 開發與測試所使用的 SSL 憑證(請參考自簽憑證的文件)。

### 設定 composer

使用 composer 安裝套件時，composer 會依照所使用的 PHP 的版本下載符合該版本的套件版本。在使用多版本 PHP 的開發環境底下，若只使用一個預設版本的 PHP 來調用 composer，有可能會造成某些 PHP 版本較高或較低的專案，無法透過 composer 安裝相對應下較為穩定的套件版本。

在多版本的 PHP 開發環境中，可以透過建立不同版本的批次檔、指令檔、ShellScript 檔案的方式來進行多版本的設定。因為概念相同，以下均用「腳本檔」代表。

本文件的內容在我的電腦架構下下進行操作，若在其他電腦則只要更改到正確的檔案路徑，基本上可以用同樣的設定運行。

#### 設定環境變數 PATH

1. 建立目錄 `C:\bin` ，用來統一管理各版本的腳本檔。
2. 在系統的環境變數 PATH 加入目錄 `C:\bin`。

#### 設定 composer.phar

請連線到 [Composer 官網的下載頁面](https://getcomposer.org/download/)，到 Manual Download 的地方下載穩定版本。將 composer.phar 存放到 `C:\bin` 目錄底下。

假設各版本的 PHP 已經放置於 `C:\php` 目錄，各版本的目錄名稱可能如下：

- php-5.6.9-Win32-VC11-x64
- php-7.1.30-Win32-VC14-x64
- php-7.2.19-Win32-VC15-x64
- php-7.3.6-Win32-VC15-x64
- php-7.4.29-Win32-VC15-x64

依照各版本建立相對應的 composer 腳本檔，檔名格式為 composer-php-v{版號簡稱}.bat。例如：composer-php-v74.bat 代表 php v7.4 的版本。

腳本檔案內容如下：

```bash
# 以 php 7.4 為例
# 檔名為 composer-php-v74.bat
@C:\php\php-7.4.29-Win32-VC15-x64\php.exe "%~dp0composer.phar" %*
```

建立完成後可在 command line 呼叫腳本檢視是否可成功執行：

```bash
# 顯示 composer 的版本
composer-php-v74 --version
```

{{< alert type="info" >}}
請注意！各 composer 搭配的 PHP 版本可能不同，但是原理基本上都一樣，因此若無法正確執行，請至 Composer 官網查詢各版本可使用的最低版本 PHP，並自行設定腳本。
{{< /alert >}}

設定成功後即可在各專案目錄下，使用 CLI 呼叫符合專案 PHP 版本的腳本來安裝套件。Composer 會依照 PHP 的版本自動抓取相對的穩定套件版本。

### 在 composer.json 中降版本

若專案要使用較高版本的 composer 安裝較低版本的套件時，另一種方式是透過設定 composer.json 的 config.platform.php 來進行降版。

例如若要在 PHP v5.6 的專案使用 PHP v7.1 的 composer 安裝套件時，可能會產生如以下的錯誤：

`Composer detected issues in your platform: Your Composer dependencies require a PHP version ">= 7.1.3".`

此時可以在專案的 composer.json 進行以下設定：

```conf
"config": {
    "platform": {
        "php": "5.6"
    },
    ...
},
```

此設定會讓 composer 在此專案安裝套件時，以 PHP v5.6 的版本作為基準來下載套件。設定完成後在專案的 CLI 使用 `composer update` 指令更新套件即可降版。

## 結論

在多版本的開發環境下，自行設定 PHP 搭配 composer 的腳本，可以避免掉安裝專案套件的一些版本控制問題。
