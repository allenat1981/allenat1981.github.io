---
title: "PHP 使用 openssl 進行加解密"
date: 2024-12-31 17:03:00 +0800
categories:
  - php
tags:
  - PHP
  - OpenSSL
---

## 參考資料

[OpenSSL Functions](https://www.php.net/manual/en/ref.openssl.php)

## openssl_ 函式

PHP 提供一系列的 **openssl_** 前綴的加解密方法。

使用前需確認有安裝並開啟 OPENSSL extension。

### openssl_encrypt() 加密函式

參考資料：[openssl_encrypt](https://www.php.net/manual/en/function.openssl-encrypt.php)

使用 `openssl_encrypt()` 可對明文(plaintext)進行加密，加密後的資料通常稱為密文(ciphertext OR cyphertext)。

```php
openssl_encrypt(
    string $data,
    string $cipher_algo,
    string $passphrase,
    int $options = 0,
    string $iv = "",
    string &$tag = null,
    string $aad = "",
    int $tag_length = 16
): string|false
// Encrypts given data with given method and key, returns a raw or base64 encoded string
// 根據 $options 設定，會回傳 raw data(OPENSSL_RAW_DATA) 或 base64 encoded(default) 字串
```

參數說明如下：

- $data 要加密的明文資料
- $cipher_algo 加密演算法
- $passphrase 加密演算法所使用的密碼(密鑰)
- $options 加解密標記
  - 詳細說明請參考 [Other Constants](https://www.php.net/manual/en/openssl.constants.other.php)，以下略為註記
  - `OPENSSL_RAW_DATA` 若在 openssl_encrypt() 和 openssl_decrypt() 設置，則將會回傳處理完成後的原始資料(raw data)，若沒有設定（預設為 0），則將會回傳經過 Base64 編碼的資料(Base64 Encoded)。
  - `OPENSSL_ZERO_PADDING` 默認情況下，加密操作使用標準區塊填充進行填充，並且在解密時會檢查並刪除填充。如果在 openssl_encrypt() 或 openssl_decrypt() 選項中設置了 OPENSSL_ZERO_PADDING 則不執行填充，則加密或解密的數據總量必須是區塊大小的倍數，否則將發生錯誤。
    - 也就是說若使用 OPENSSL_ZERO_PADDING，則必須自行處理加解密原始內容的區塊長度。
- $iv 初始化向量 (initialization vector)，為了避免加密的內容被破解而在加密時加入的亂數值。
  - 視加密演算法有不同的 iv 內容要求。例如 DES 就不須使用 iv。
  - 可以使用 `openssl_cipher_iv_length($cipher_algo)` 取得加密演算法所需的 iv 長度。接著使用`openssl_random_pseudo_bytes($iv_length)` 產生相應長度的 iv 內容。
- $tag, $aad, $tag_length 尚未用到，等熟悉之後再補上說明。

### openssl_decrypt() 解密函式

參考資料：[openssl_decryp](https://www.php.net/manual/en/function.openssl-decrypt.php)

使用 `openssl_encrypt()` 可對密文進行解密，解密後的資料通常即為明文。

```php
openssl_decrypt(
    string $data,
    string $cipher_algo,
    string $passphrase,
    int $options = 0,
    string $iv = "",
    ?string $tag = null,
    string $aad = ""
): string|false
// Takes a raw or base64 encoded string and decrypts it using a given method and key.
```

參數說明如下：

- $data 要解密的密文資料
- $cipher_algo 加密演算法。加密時用什麼演算法，解密就要用同樣的演算法。
- $passphrase 解密所使用的密碼(密鑰)
  - 對稱式加解密：加解密使用同樣的密鑰。
  - 非對稱式加解密：加解密使用對應的公鑰/私鑰。
- $options 加解密標記。請參考 openssl_encrypt 的說明。
- $iv 初始化向量。需輸入加密時使用的 iv。
- $tag, $aad, $tag_length 尚未用到，等熟悉之後再補上說明。

## AES-256-CBC

參考資料：

- [Day 7 - 使用 AES-CBC 機制對 Message 內文進行加密](https://ithelp.ithome.com.tw/articles/10268975)
- [Day 20. 對稱式加密演算法 - 大家都愛用的 AES](https://ithelp.ithome.com.tw/articles/10249488)

AES-256 簡單說明：

- 使用 256 bits 的密鑰(32 bytes)。
- 將明文拆解成每 256 bits 為一個區塊(block)
  - 若拆解的區塊不足 256 bits，則視各語言的實作進行可能的填充，將最後的區塊補滿 256 bits。
- 用密鑰針對每一個 block 進行加密。
- 另外 AES 還有 128 和 192。可簡單想成密鑰長度和 block 區塊長度不同。

### AES-256-CBC 加密範例

假設明文檔案為 plain.txt。

```php
$algo = 'AES-256-CBC'; // 指令加密演算法
$plain = file_get_contents("plain.txt"); // 讀入明文
$passphrase = "ijsiBi9dvaa13jijda"; // 設定密鑰。
$iv_length = openssl_cipher_iv_length($algo); // 取得演算法所需的 iv 長度。
$iv = openssl_random_pseudo_bytes($iv_lengthv); // 根據傳入的長度產生隨機字串作為 iv。請將此保存好。
$ciphertext = openssl_encrypt($plain, $algo, $passphrase, OPENSSL_RAW_DATA, $iv); //進行加密
```

注意事項：

- 密鑰的長度根據演算法而有所不同。AES-256 需要 256 bits(32 bytes)長度的密鑰。當使用 `openssl_encrypt()` 進行加密，密鑰長度不足時，會自動補上 NUL；當密鑰長度過長，則會被自動截斷。
- `openssl_chipher_iv_length()` 可以根據傳入的演算法傳回該演算法所需的 iv 長度($iv_length)。
- 取得 iv 長度後，可以使用 `openssl_random_pseudo_bytes()` 產生相應長度的字串。
- 密鑰和 iv 需要保存好，應避免外流或者遺失。

### AES-256-CBC 解密範例

假設已經產生密文，並寫入檔案 cipher.data。

```php
$algo = 'AES-256-CBC'; // 指定加解密演算法
$ciphertext = file_get_contents("cipher.data"); // 讀入密文
$passphrase = "ijsiBi9dvaa13jijda"; // 加解密的密鑰。
$iv = "someiv"; // 加密的 iv。
$plaintext = openssl_decrpyt($ciphertext, $algo, $passphrase, OPENSSL_RAW_DATA, $iv); //進行解密
```
