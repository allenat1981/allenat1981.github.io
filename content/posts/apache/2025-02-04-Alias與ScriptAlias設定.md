---
title: "Alias 與 ScriptAlias 設定"
date: 2025-02-04 16:30:00 +0800
categories: 
  - apache
tags:
  - apache
---

## 模組簡述

模組名稱：alias_module

`Alias` 和 `ScriptAlias` 是用來設定 URL 路徑和系統檔案路徑的對應(映射)。此設定允許不在 DocumentRoot 目錄下的內容做為 Web 檔案目錄樹狀結構的一部分。ScriptAlias 指令具有將目標目錄標記為僅包含 CGI 腳本的附加效果。

`Redirect` 指令用於指示客戶端使用不同的 URL 發出新請求。 它們通常在資源移動到新位置時使用。

當在 `<Location>` 或 `<LocationMatch>` 部分中使用 Alias、ScriptAlias 和 Redirect 指令時，可以使用表達式語法來操作目標路徑或 URL。

mod_alias 旨在處理簡單的 URL 操作任務。 對於操作查詢字符串等更複雜的任務，請使用 mod_rewrite 提供的工具。

### 補充

mod_alias 模組包含 Redirect 相關指定，請參照官方文件說明。

換句話說，VirtualHost 原本設定外部可直接存取的目錄為 DocumentRoot 所設定的目錄。透過 Alias 的設定則可以將目錄掛載到 DocumentRoot 目錄結構底下。ScriptAlias 則是用來掛載僅包含 CGI 腳本的目錄。

## 運作順序

根據標準合併規則，在不同上下文中發生的別名和重定向與其他指令一樣被處理。但是，當多個別名或重定向出現在同一上下文中時（例如，在同一 `<VirtualHost>` 部分中），它們將按特定順序進行處理。

1. 首先，在處理別名之前處理所有重定向(Redirect)，因此匹配重定向或重定向匹配的請求將永遠不會應用別名。
   1. 換句話說，若同時為 Redirect 和 Alias 設定同樣的規則，則只會執行 Redirect 的設定。
2. 其次，別名和重定向按照它們在配置文件中出現的順序進行處理，第一個匹配項優先。
   1. 若同一個別名或重定向規則被設定，則用第一個抓到的規則優先執行。

最好的方式就是不要出現重複設定，也可以避免混淆。

因此，當這些指令中的兩個或多個應用於同一子路徑時，您必須首先列出最具體的路徑，以使所有指令生效。例如，以下配置將按預期工作：

```conf
# 相當於在 DocumentRoot 底下掛載 /foo/bar 目錄，並對應到主機的 /baz 資料夾
Alias "/foo/bar" "/baz"
# 相當於在 DocumentRoot 底下掛載 /foo 目錄，並對應到主機的 /gaq 資料夾
Alias "/foo" "/gaq"
```

但是如果上面兩個指令的順序顛倒過來，別名 /foo 總是會在 /foo/bar 之前匹配，所以後面的指令會被忽略。

換句話說：

```conf
# 相當於在 DocumentRoot 底下掛載 /foo 目錄，並對應到主機的 /gaq 資料夾
Alias "/foo" "/gaq"

# 當匹配到 /foo 時，就會使用以上的規則，因此會嘗試存取的路徑實際上為 /gaq/bar。以下規則將永遠不會生效。
Alias "/foo/bar" "/baz"
```

## VirtualHost 共用路徑的原理

若別名只對特定的 VirtualHost 生效，則可設定在 VirtualHost 區塊內。例如：

```conf
# 若為 SSL 則可設定 443
<VirtualHost *:80>
...
DocumentRoot "/web/miamia.com.tw/public_html"
ServerName miamia.be-yond.allen
# 在此 VirtualHost 設定 /foo 會映射到 /web/test 資料夾
Alias "/foo" "/web/test"
...
</VirtualHost>
```

以上設定成功後，http://miamia.be-yond.allen/foo 實際上會對應到 /web/test 資料夾，而非 DocumentRoot 所設定的 /web/miamia.com.tw/public_html/foo。

在此概念底下，若將 Alias 設定在外部，則代表所有的 VirtualHost 都會共用同樣的路徑別名。例如：

```conf
# 將 foo 設定在外部
Alias "/foo" "/web/test"

# miamia 的虛擬主機
<VirtualHost>
...
ServerName miamia.be-yond.allen
...
</VirtualHost>

# 1976 的虛擬主機
<VirtualHost>
...
ServerName 1976.be-yond
...
</VirtualHost>
```

以上設定完成後，miamia.be-yond.allen/foo 和 1976.be-yond/foo 都同樣對應到 /web/test 目錄。

### phpmyadmin 的範例

在 Debina11 安裝完 phpmyadmin 後，所有 VirtualHost 的網址接上 /phpmyadmin 都可以進入到 phpmyadmin 的登入首頁，其原理就是使用 Alias。

在安裝完 phpmyadmin 後，可以在 /etc/apache2/conf-avaialable 目錄內找到 phpmyadmin.conf，使用 vi 編輯器開啟可以看到以下資訊：

```conf
# phpMyAdmin default Apache Configuration
Alias /phpmyadmin /usr/share/phpmyadmin 

<Directory /usr/share/phpmyadmin>
...
</Directory>
...
```

上面可以看到 phpmyadmin.conf 會在外部設定一個 /phpmyadmin 的別名對應到 /usr/share/phpmyadmin 目錄，因此所有的 VirtaulHost 只要在網址後面加上 /phpmyadmin，實際上都是對應到 /usr/share/phpmyadmin 這個目錄，而該目錄就是存放 phpmyadmin 的主要檔案(php)。

基於安全性的考量，若不想讓所有人都可以透過 /phpmyadmin 進入 phpmyadmin 的首頁，可以考慮的方式：

方法一：鎖定 /usr/share/phpmyadmin 的 IP 存取權限

```conf
<Directory /usr/share/phpmyadmin>
...
# 先封鎖所有的存取權限
Require all denied
# 僅開放特定IP，若有多組 IP 可用空白分隔。
# 也可用 netmask 方式設定網段，例如 192.168.0.0/24
Require ip 1.34.133.73
...
</Directory>
```

以上可參考 [VM虛擬主機(beyond_miamia)](../Azure/VM虛擬主機(beyond_miamia).md) 內的 phpmyadmin 安裝與設定部分。

方法二：將 Alias 限制在特定的 VirtualHost(尚未實作過)

開啟 phpmyadmin.conf，並將 Alias 註解或取消

```conf
# phpMyAdmin default Apache Configuration
# 註解掉外部(全域)別名
# Alias /phpmyadmin /usr/share/phpmyadmin 

<Directory /usr/share/phpmyadmin>
...
</Directory>
...
```

之後可能可以採用的做法：

1. 在特定的 VirtualHost 設定 Alias /phpmyadmin，僅讓特定的 VirtualHost 可對 /phpmyadmin 發出要求。
2. 或者直接開一個 VirtualHost 讓 DocumentRoot 指向 /usr/share/phpmyadmin。用此方法應該可以另外再指定 VirtualHost 的連接埠(port)，達成像某些虛擬主機將 phpmyadmin 設為不同連接埠的效果。
3. 若情況允許，最好還是搭配鎖定 IP 權限。

## ScriptAlias 簡述

ScriptAlias 指令與 Alias 指令基本上具有相同的行為。除此之外，它將目標目錄標記為包含將由 mod_cgi 的 cgi-script 處理程序處理的 CGI 腳本。

ScriptAlias 的設定與 Alias 相似，但是它會限制對其發出的要求必須是可執行的 CGI 腳本，否則會產生類似以下的 errorlog：

```txt
[Tue Mar 29 11:05:47.107805 2022] [win32:error] [pid 11252:tid 1204] [client 192.168.0.144:54598] AH02102: C:/php/php56/php.gif is not executable; ensure interpreted scripts have "#!" or "'!" first line
[Tue Mar 29 11:05:47.107805 2022] [cgi:error] [pid 11252:tid 1204] (9)Bad file descriptor: [client 192.168.0.144:54598] AH01222: don't know how to spawn child process: C:/php/php56/php.gif
```

從錯誤訊息看到大概是指檔案必須是要是 `#!` 或 `'!` 開頭的可執行的腳本檔案。也就是[模組簡述](#模組簡述) 提到 ScriptAlias 會將目錄標記為必須為可執行的腳本檔案。

## 小結

Alias 的使用情境，目前可以想到的如下：

1. 多個 VirtualHost 要共用相同的資源。
2. 某些資源並沒有直接放置在 VirtualHost 的 DocumentRoot 底下，可以使用 Alias 像掛載的方式一樣掛載到 DoucmentRoot 目錄下。
