---
title: "在 PHP-FPM 服務設定 umask"
date: 2025-01-16 12:00:00 +0800
categories: 
  - php
tags:
  - PHP
  - Linux
---

## 說明

以下是在 WSL 的 AlmaLinux8 所進行的操作。其他版本的 Linux 可能略有不同。

## 參考資料

- [Proper Way of Setting Umask For php-fpm on Debian/Ubuntu](https://serverfault.com/questions/694396/proper-way-of-setting-umask-for-php-fpm-on-debian-ubuntu)
- [Setting the umask of the Apache user](https://stackoverflow.com/questions/428416/setting-the-umask-of-the-apache-user)

## 設定 PHP-FPM 建立檔案與目錄的 umask

使用 PHP-FPM 在網站後端建立檔案或目錄時，預設的擁有者是 apache，並且 umask 會是系統的預設 022。若要方便群組權限，則需要調整 PHP-FPM 建立檔案或目錄的 umask 為 0002。

### 推薦作法：修改 PHP-FPM 服務參數

讓 umask 0002 成為 PHP-FPM 處理時的預設值，方法如下：

在 PHP-FPM 服務設定加入 umask.conf ，例如在 AlmaLinux 預設安裝的 PHP-FPM，其路徑為 `/etc/systemd/system/php-fpm.service.d`，故進行以下設定

```conf
# sudo vi /etc/systemd/system/php-fpm.service.d/umask.conf
# 加入以下(大小寫需吻合)
[Service]
UMask=0222
```

編輯完成後，重新啟動相關服務

```bash
sudo systemctl daemon-reload
sudo systemctl restart php-fpm
# 若沒有生效，則重啟 httpd
sudo systemctl restart httpd
```

### 不推薦的作法

另一種作法則是在 PHP 程式呼叫 umask() 來改變預設的 umask 設定，但是每次都要呼叫此方法，故不建議。

```php
umask(0002);
$fp = fopen('filename', 'w');
fwrite($fp, 'blahblah');
fclose($fp)
```
