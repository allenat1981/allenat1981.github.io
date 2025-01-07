---
layout: single
title: "在 PHPMailer 使用 DKIM 簽章"
date: 2025-01-07 16:00:00 +0800
category: php
tags:
    - php
    - mail
---

# 在 PHPMailer 使用 DKIM 簽章

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
# select-myhost.private 私鑰
# select-myhost.txt 公鑰 DNS 記錄
```

常用參數說明：

- `-b`：指定金鑰長度。建議 2048。
- `-D`：指定金鑰儲存的目錄。
- `-d`：設定你的網域名稱（例如：myhost.com.tw）。
- `-s`：設定 DKIM 選用的 selector（可自訂，預設為 default）

### 設定公鑰 DNS

到 DNS 管理介面將將公鑰 DNS 加入。(預設公鑰會以斷行的方式呈現，可自行將斷行取消後加入)

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
