---
title: "設定多版本PHP(debian)"
date: 2024-12-31 16:51:00 +0800
categories: 
  - php
tags:
  - PHP
---

若要替不同的 VirtualHost 設定不同版本的 PHP，可以使用 PHP-FPM

## 參考資料

- [Debian/Ubuntu 環境安裝 PHP-FPM 8.2、8.1 或 8.0 版本](https://www.kjnotes.com/devtools/82)
- [DEB.SURY.ORG](https://deb.sury.org/)
  - 此網站是由替 Debian 與 Ubuntu 打包 PHP 版本的負責人所建立，提供不同版本的 PHP-FPM。

## 情境說明

若要在同一台 WebServer 替不同的 VirtualHost 設定不同版本的 PHP，可以透過安裝不同版本的 PHP-FPM 來進行設定。

## 設定流程

透過將 deb.sury.org 提供的 PHP repository for Debian，將套件安裝路徑加入到 apt 的 sources.list，即可使用 apt 來安裝不同版本的 PHP-FPM。

### 將 packages.sury.org/php 加入 apt sources.list

參考官方的 README [https://packages.sury.org/php/](https://packages.sury.org/php/)

```bash
# 更新 apt
sudo apt update

# 安裝需要用到的工具
sudo apt install lsb-release ca-certificates curl

# 用 curl 下載簽名金鑰
sudo curl -sSLo /usr/share/keyrings/deb.sury.org-php.gpg https://packages.sury.org/php/apt.gpg

# 將 packages.sury.org/php 加入到 sources.list
# 若使用 debian 為基礎的發行型，如 Linux Mint，請將 $(lsb_release -sc) 替換為 debian 版本號，例如 debian 13 請替換為 trixie
sudo sh -c 'echo "deb [signed-by=/usr/share/keyrings/deb.sury.org-php.gpg] https://packages.sury.org/php/ $(lsb_release -sc) main" > /etc/apt/sources.list.d/php.list'

# 再更新一次
sudo apt update
```

### 安裝不同版本的 PHP

```bash
# 安裝不同版本的 php-fpm
# sudo apt install php{version}

sudo apt install php7.4
sudo apt install php8.2

# 可安裝 extension，以下範例
sudo apt install php7.4-gd php7.4-pdo-mysql #...
sudo apt install php8.2-gd php8.2-pdo-mysql #...

```

### 安裝不同版本的 PHP-FPM

若要安裝不同版本的 php-fpm

```bash
# 安裝不同版本的 php-fpm
# sudo apt install php{version}-fpm
sudo apt install php7.4-fpm
sudo apt install php8.2-fpm

```

### 替不同的 VirtualHost 設定不同版本的 PHP-FPM

首先要開啟 Apache 的 proxy_fcgi 模組

```bash
sudo a2enmod proxy_fcgi
```

替不同 VirtualHost 設定 .php 檔的 Handler。開啟 VirtualHost 的設定檔(.conf) 加入

```conf
<VirtualHost *:80>
    #...
    # 加入 FilesMatch 讓 .php 結尾的檔案透過 php7.4-fpm 處理
    # 若要用不同版本則改 php{version}-fpm 即可
    <FilesMatch \.php$>
        SetHandler "proxy:unix:/var/run/php/php7.4-fpm.sock|fcgi://localhost"
    </FilesMatch>
    #...
</VirtualHost>
```

重啟 apache 後即可使用 phpinfo() 檢查是否設定成功

### 補充：切換 php-cli 版本

若要在 Command-Line 切換 php 版本，可以使用以下指令切換

```bash
# 將 php 設定為 php7.4
sudo update-alternatives --set php /usr/bin/php7.4

# 檢查目前cli版本是否為 7.4
php -v
```

### 補充：Rocky Linux 參考

Rocky Linux 系列可參考以下連結

[RHEL / Rocky Linux 8 安裝多個 PHP 版本](https://www.ltsplus.com/linux/rhel-rocky-linux-oracle-linux-install-multiple-php)
