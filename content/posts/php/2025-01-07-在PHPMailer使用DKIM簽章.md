---
title: "在 PHPMailer 使用 DKIM 簽章"
date: 2025-01-07 16:00:00 +0800
categories:
    - php
tags:
    - PHP
    - mail
---

當系統使用 PHPMailer 寄送郵件時，若主機環境無法設定 DKIM，則可以考慮自己產生 DKIM 金鑰，並在 PHPMailer 發送郵件時設定簽章。

## 前置作業

安裝 OpenDKIM

```bash
sudo apt install opendkim opendkim-tools
```

## 建立 DKIM 金鑰

假設以下環境

- 寄件者域名: myhost.com.tw
- 自訂 selector: select-myhost
- 金鑰目錄 ~/dkim-keys(請先建立目錄)

```bash
sudo opendkim-genkey -b 2048 -D ~/dkim-keys -d myhost.com.tw -s select-myhost

# 完成後會在指定的目錄產生以下檔案
# select-myhost.private 私鑰(pem 格式)
# select-myhost.txt 公鑰 DNS 記錄

# 可使用 openssl 對金鑰進行驗證

openssl rsa -in select-myhost.private -check
# 驗證成功出現
# RSA key ok ...
```

常用參數說明：

- `-b`：指定金鑰長度。建議 2048。
- `-D`：指定金鑰儲存的目錄。
- `-d`：設定你的網域名稱（例如：myhost.com.tw）。
- `-s`：設定 DKIM 選用的 selector（可自訂，預設為 default）

### 設定公鑰 DNS

到 DNS 管理介面將將公鑰 DNS 加入，並等待生效。(預設公鑰會以斷行的方式呈現，可自行將斷行取消後加入)

### PHPMailer 設定 DKIM 簽章

可將 PHPMailer 包在一個靜態工廠類別，參考以下範例

```php
use PHPMailer\PHPMailer\PHPMailer;

class PHPMailerFactory
{
    public const FROM_DOMAIN = "myhost.com.tw";

    public static function CreateMailer($exception = null): PHPMailer
    {
        $mailer = new PHPMailer($exception);
        $mailer->setFrom('account@myhost.com.tw', 'Somebody');
        $mailer->DKIM_domain = 'myhost.com.tw'; //DKIM 域名
        $mailer->DKIM_private = '/path/to/dkimkeys/select-myhost.private' //DKIM 私鑰檔
        $mailer->DKIM_passphrase = ''; //如果私鑰有密碼，填入密碼（通常不需要）
        $mailer->DKIM_selector = 'select-apply'; //DKIM selector
        $mailer->DKIM_identity = $mailer->From; //郵件身份（通常與寄件人一致）
        return $mailer;
    }
}

$mailer = PHPMailerFactory::CreateMailer();
//... 其他設定
$mailer->send();
```

設定完成即可寄送測試郵件。

## 補充：用 openssl 測試 OpenDKIM 簽章

可自行使用 openssl 對 OpenDKIM 產生的金鑰進行測試。

建立一個測試的文字檔案

```bash
echo "Test message for DKIM key verification" > message.txt
```

使用 DKIM 私鑰對檔案進行簽章

```bash
openssl dgst -sha256 -sign select-myhost.private -out signature.bin message.txt
```

- `-sha256`：指定使用 SHA-256 雜湊演算法（DKIM 通常使用）。
- `-sign` select-myhost.private：使用私鑰簽署訊息。
- `-out` signature.bin：生成二進位簽名檔案。

接著使用公鑰驗證簽章。預設 OpenDKIM 產生的公鑰會在公鑰檔內的 `p=` 欄位

```txt
select-myhost._domainkey IN TXT ( "v=DKIM1; k=rsa; "
  "p=MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQE..." )  ; ----- DKIM key select-apply for taipeiitf.org.tw
```

可將 p 的內容複製出後，移除換行與字串符號(")，存成以下公鑰檔案格式，例如另存為 select-myhost.public

```txt
-----BEGIN PUBLIC KEY-----
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQE...
-----END PUBLIC KEY-----
```

接著使用公鑰檔針對簽章檔進行驗證

```bash
openssl dgst -sha256 -verify select-myhost.public -signature signature.bin message.txt

# 若驗證成功，輸出以下訊息
# Verified OK

```

- `-verify` public_key.pem：使用公鑰進行驗證。
- `-signature` signature.bin：簽章檔案。
- message.txt：原始檔案。
