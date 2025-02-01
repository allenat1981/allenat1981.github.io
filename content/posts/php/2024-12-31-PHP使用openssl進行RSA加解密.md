---
title: "PHP 使用 openssl 進行 RSA 加解密"
date: 2024-12-31 17:10:00 +0800
categories:
  - php
tags:
  - PHP
  - OpenSSL
---

## 參考資料

透過對 ChatGPT 提問取得的解答進行節錄。

## 原由

1976 與紅陽進行 BNML 整合，必須透過 RSA 的 PublicKey 對文本內容進行加密與傳送。

## 目標

- 使用 openssl 在 console 建立一組 RSA PublicKey/PrivateKey組合。
- 使用建立好的 PublicKey/PrivateKey 對文本進行加解密。

## 操作流程

以下主要記錄與 ChatGPT 問答所取得的資訊，並根據實作調整內容。

並且保留提問內容以便之後可回溯當初的提問。

### 使用 openssl 建立 PublicKey/PrivateKey

> 如何用 openssl 生成公鑰檔與私鑰檔?

使用 OpenSSL 生成公鑰檔與私鑰檔的步驟如下：

打開終端機或命令提示字元視窗，輸入以下命令生成私鑰：

```bash
openssl genrsa -out private.key 2048
```

這會生成一個長度為 2048 位元的 RSA 私鑰，並將其存儲在 private.key 檔案中。

輸入以下命令生成公鑰：

```bash
openssl rsa -in private.key -pubout -out public.key
```

這會將 private.key 檔案中的私鑰轉換為公鑰，並將其存儲在 public.key 檔案中。

驗證公鑰與私鑰是否匹配：

```bash
openssl rsa -in private.key -check
openssl rsa -in public.key -pubin -check
```

這會分別驗證 private.key 和 public.key 中的私鑰和公鑰是否匹配。

完成上述步驟後，您就可以使用生成的公鑰和私鑰進行加密和解密等操作。

### 使用 PublicKey 對文本進行加密

>PHP 如何使用 RSA 公鑰加密?

```php
// 載入 RSA 公鑰
$publicKey = openssl_pkey_get_public(file_get_contents('public.key'));

// 要加密的明文
$data = 'Hello World!';

// 使用 RSA 公鑰加密
openssl_public_encrypt($data, $encrypted, $publicKey);

// 輸出加密後的結果
echo $encryptedBase64;
```

### RSA 公鑰加密的限制

使用 RSA 公鑰加密時的注意事項

>RSA公鑰加密的限制為何?

1. 加密明文的長度不能超過公鑰的長度減去 11 bytes，否則需要分塊加密。
2. 加密效率相對較低，不適合加密大量數據。
3. 加密的安全性依賴於使用的密鑰長度，密鑰長度越短越容易被破解。
4. 加密和解密的過程耗費時間較長，不適合對即時性要求較高的應用場景。

若明文長度大於公鑰長度 - 11 bytes，則需要進行分塊加密，以下為範例程式：

```php
public function encryptPlainText(string $pubKeyFile, string $plainText): string
{
    $pubKeyContent = file_get_contents($pubKeyFile);
    if ($pubKeyContent === false) {
        throw new Exception("cannot read public key: {$pubKeyFile}");
    }

    $publicKey = openssl_pkey_get_public($pubKeyContent);        
    if ($publicKey === false) {
        throw new Exception('openssl_pkey_get_public cannot get public key');
    }
    //使用 openssl_pkey_get_details 函式取得公鑰的資訊，該函式將會回傳一組陣列，使用 bits 陣列索引可取得公鑰長度(bits)。
    //將取得的公鑰長度轉為 bytes 後扣掉 11 即為每個加密區塊的大小($blockSize)。
    $publicKeyDetail = openssl_pkey_get_details($publicKey);
    $blockSize = $publicKeyDetail['bits'] / 8 - 11;
    //使用 str_split 將明文依照 $blockSize 切割成陣列。再將每個陣列中的每個區塊逐一加密後串接成新的加密文本。
    $plainBlocks = str_split($plainText, $blockSize);
    $cipher = '';
    foreach ($plainBlocks as $plainBlock) {
        $success = openssl_public_encrypt($plainBlock, $encryptedBlock, $publicKey);
        if (!$success) {
            throw new Exception("cannot encrypt UrlencodeFormString");
        }
        $cipher .= $encryptedBlock;
    }
    return $cipher;
}
```

#### 檢視 RSA 公鑰的長度

> 如何知道 RSA 公鑰的長度?

```bash
openssl rsa -in public_key.pem -pubin -text -noout
```

其中，public_key.pem 是公鑰的檔案路徑。在命令執行後，將輸出公鑰的詳細資訊，包括模數和指數。模數的位元長度即為 RSA 公鑰的長度。
