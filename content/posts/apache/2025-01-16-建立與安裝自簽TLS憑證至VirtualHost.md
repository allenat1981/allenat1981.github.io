---
title: "建立與安裝自簽 TLS 憑證至 VirtualHost"
date: 2025-01-16 14:20:00 +0800
categories: 
  - apache
tags:
  - apache
  - TLS
  - OpenSSL
---

## 說明

若要在本地開發/測試使用 TLS 連線(https)，可以建立自簽憑證並安裝至 apache 的 VirtualHost。

## 建立自簽憑證

建立一個產生自簽憑證的設定檔，例如建立一個替域名 myproject.localhost 的憑證設定 `~/local-cert/myproject.conf`。

```conf
[req]
prompt = no
default_md = sha256
default_bits = 2048
distinguished_name = dn
x509_extensions = v3_req

# 設定憑證相關說明，基本上隨便填
[dn]
C = TW
ST = Taiwan
L = Taipei
O = my company # 公司
OU = IT # 部門
emailAddress = admin@example.com # 管理者信箱
CN = myproject.localhost # 主體名稱

[v3_req]
subjectAltName = @alt_names

# 設定憑證的域名與IP
[alt_names]
DNS.1 = myproject.localhost
DNS.2 = *.myproject.localhost
IP.1 = 127.0.0.1
# ... 可自行新增
```

建立完後使用 openssl 產生私密金鑰和憑證檔案

```bash
# 切換到 ~/local-cert 目錄
openssl req -x509 -new -nodes -sha256 -utf8 \
-days 3650 -newkey rsa:2048 \
-keyout myproject.private -out myproject.crt -config myproject.conf
```

指令說明：

- `req` OpenSSL 的一個子命令，用於處理證書簽名請求（CSR）以及生成自簽名證書。
- `-x509` 指定生成一個 X.509 格式的證書，通常用於自簽名證書。
- `-new` 生成一個新的證書或 CSR。
- `-nodes` 跳過加密私鑰，生成的私鑰將不會有密碼保護。這在某些需要自動化的場景中很常見。
- `-sha256` 指定使用 SHA-256 作為證書的雜湊演算法。SHA-256 是目前常用的安全雜湊演算法。
- `-utf8` 將 UTF-8 編碼用於主體欄位（subject fields），這允許支持非 ASCII 字符。
- `-days 3650` 指定證書的有效期，單位為天，此處為 10 年（3650 天）。
- `-newkey rsa:2048` 生成一個新的 RSA 私鑰，密鑰長度為 2048 位元。此選項同時要求生成私鑰和 CSR。
- `-keyout myproject.private` 指定生成的私鑰檔案名稱，這裡是 myproject.private。
- `-out myproject.crt` 指定輸出的證書檔案名稱，這裡是 myproject.crt。
- `-config myproject.conf` 指定使用的配置檔案，這裡是 myproject.conf。該檔案中包含一些證書生成的詳細設置（如主體資訊、擴展欄位等）。

## 啟用 mod_ssl

根據不同的版本，必須先啟動 apache 的 mod_ssl。

以 AlmaLinux 為例，必須安裝 mod_ssl

```bash
# 安裝 mod_ssl
sudo dnf install mod_ssl
# 重啟 httpd
sudo systemctl restart httpd
```

## 設定 TLS/SSL 憑證

開啟 VirtualHost 的設定檔，加入以下設定

```conf
<VirtualHost *:443>
    # ServerName ...
    # DocumentRoot ...

    SSLEngine on # 開啟 SSLEngine
    SSLCertificateFile "/home/allen/local-cert/myproject.crt" # 設定憑證檔路徑
    SSLCertificateKeyFile "/home/allen/local-cert/myproject.private" # 設定私鑰檔

    # others ...
</virtualhost>
```

設定完成後重啟 httpd

```bash
sudo systemctl restart httpd
```

接著即可用瀏覽器開啟 https://myproject.localhost 進行測試。

## 將自簽憑證加入受信任的憑證庫

瀏覽器檢視自簽憑證的網站，通常會出現不信任/不安全的訊息，此時可以將自簽憑證加入到主機內「受信任的根憑證授權單位」。

加入後等待一段時間讓瀏覽器更新後重試 https 連線即可。
