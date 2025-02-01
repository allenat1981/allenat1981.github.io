---
title: "Rocky Linux 設定多版本 PHP"
date: 2025-01-09 18:00:00 +0800
categories: 
  - php
tags:
  - PHP
  - Rocky Linux
---

## 安裝 EPEL 和 Remi Repository

Rocky Linux 的官方儲存庫不提供多版本 PHP，因此需要先安裝第三方軟體庫。

Add EPEL repository

```bash
sudo dnf -y install epel-release
```

Install REMI repository

```bash
## EL 9-based systems ##
# Rocky Linux 9 安裝此項目
sudo dnf install -y https://rpms.remirepo.net/enterprise/remi-release-9.rpm

## EL 8-based systems ##
sudo dnf install -y https://rpms.remirepo.net/enterprise/remi-release-8.rpm
```

列出所有可用的 PHP 模組

```bash
dnf module list php
```

利用 Remi Repository 安裝指定版本的 PHP。以下範例示範如何安裝 PHP 7.4 和 PHP 8.1：

```bash
sudo dnf install php74 php74-php-cli php74-php-fpm php74-php-mysqlnd -y
sudo dnf install php81 php81-php-cli php81-php-fpm php81-php-mysqlnd -y
```

安裝後，每個 PHP 版本會有獨立的執行檔，可以使用以下命令檢查安裝的版本：

```bash
/usr/bin/php74 --version
/usr/bin/php81 --version
```

## 設定 PHP-FPM

每個 PHP 版本都會有自己的 PHP-FPM 設定檔與服務。例如：

- PHP 7.4 的 PHP-FPM 配置：/etc/opt/remi/php74/php-fpm.d/
- PHP 8.1 的 PHP-FPM 配置：/etc/opt/remi/php81/php-fpm.d/

啟用並啟動 PHP-FPM 服務：

```bash
sudo systemctl enable php74-php-fpm
sudo systemctl start php74-php-fpm

sudo systemctl enable php81-php-fpm
sudo systemctl start php81-php-fpm
```

## 配置 Web Server

設定 Apache 的虛擬主機。例如，將特定目錄的請求分別指向 PHP 7.4 或 PHP 8.1：

```conf
<VirtualHost *:80>
    ServerName php74.example.com
    DocumentRoot /var/www/php74

    <FilesMatch \.php$>
        SetHandler "proxy:unix:/run/php74-php-fpm/www.sock|fcgi://localhost/"
    </FilesMatch>
</VirtualHost>
```

```conf
<VirtualHost *:80>
    ServerName php81.example.com
    DocumentRoot /var/www/php81

    <FilesMatch \.php$>
        SetHandler "proxy:unix:/run/php81-php-fpm/www.sock|fcgi://localhost/"
    </FilesMatch>
</VirtualHost>
```

重新啟動 Apache

```bash
sudo systemctl restart httpd
```
