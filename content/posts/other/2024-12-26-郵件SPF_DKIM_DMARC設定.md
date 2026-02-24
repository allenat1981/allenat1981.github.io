---
title: "寄送郵件設定 SPF DKIM DMARC"
date: 2024-12-26 14:15:00 +0800
categories: 
  - Other
tags:
  - mail
---

為了通過收件者伺服器的檢驗，在寄件者的網域 DNS 必須設定 SPF、DKIM、DMARC。

假設寄件者為 `allen@be-yond.com.tw`

必須到 be-yond.com.tw 的 DNS 管理介面設定，加入以下記錄

## SPF

SPF 機制可以讓收件者的郵件伺服器，到寄件者域名的 DNS 伺服器查詢 SPF 記錄，用來確認寄件來源主機是該域名所允許的。

例如以下情境：

1. 在 smtp.example.com 撰寫一隻程式，用 PHPMailer 設定 From 或 Sender 為 `allen@be-yond.com.tw`，寄送一封郵件到 `allengogogo@gmail.com`。
2. `allengogogo@gmail.com` 郵件伺服器收到以後，會去找管理 be-yond.com.tw 這個域名的 DNS 主機，接著根據 DNS 主機的 SPF 記錄，查詢 be-yond.com.tw 這個域名是否允許從 example.com 這台主機寄送郵件(SPF 必須包含 include:smtp.exmaple.com)。
3. 根據 SPF 的檢查判斷郵件的來源是否合法。

DNS 的 SPF 記錄欄位如下：

```txt
名稱 be-yond.com.tw
型態 TXT
值 v=spf1 +mx +a +ip4:122.146.203.226 ~all
```

名稱為寄件者的域名，根據 DNS 管理介面的不同，決定是否需要加 .(大部分不需要)。

### 常見的 SPF 參數及其意義

- v=spf1： 指定 SPF 的版本，目前普遍使用 v=spf1。
- mx： 允許所有在 DNS 中列出的 MX 記錄所對應的郵件伺服器發送郵件。
- a： 允許所有在 DNS 中列出的 A 記錄所對應的 IP 位址發送郵件。
- ptr： 允許所有在 DNS 中列出的 PTR 記錄所對應的 IP 位址發送郵件。
- ip4： 指定特定的 IPv4 位址。
- ip6： 指定特定的 IPv6 位址。
- include： 包含另一個網域的 SPF 記錄。
- exists： 檢查是否存在指定的 DNS 記錄。
- all： 指定預設的處理方式，通常有以下幾種：
- +all： 允許所有未明確禁止的郵件。
- -all： 拒絕所有未明確允許的郵件。
- ~all： 軟性失敗，郵件可能會被標記為垃圾郵件。
- ?all： 無條件通過，不進行任何檢查。

SPF 紀錄範例

```txt
v=spf1 mx include:_spf.google.com ip4:203.0.113.1 -all
```

這個範例表示：

- 允許所有 MX 記錄所對應的郵件伺服器發送郵件。
- 包含 Google 的 SPF 記錄（通常用於 Google Workspace）。
- 允許 IP 位址 203.0.113.1 發送郵件。
- 如果郵件來自未經授權的伺服器，則拒絕接收。

#### 設定 SPF 記錄的注意事項

- 唯一性： 每個網域只能有一個 SPF 記錄。
- 順序性： SPF 紀錄中的參數順序會影響匹配結果，一般將較嚴格的限制放在前面。
- 複雜性： SPF 記錄的設定可能較為複雜，建議參考相關文件或諮詢專業人士。
- 測試： 設定完 SPF 記錄後，建議使用線上 SPF 測試工具進行驗證。

## DKIM

DKIM 通常設定在寄件者的 MTA 伺服器。

DKIM 機制是寄件者在發送郵件時，透過私鑰建立一個簽章，並隨著郵件的表頭送到收件者端。收件者的郵件伺服器收到該郵件後，根據 DKIM 簽章，去寄件者域名的 DNS 伺服器找到 DKIM 的公鑰來驗證簽章，確保郵件沒有被竄改。

### DKIM 工作流程簡述

1. 生成公私鑰對：在設定 DKIM 時，會生成一對公私鑰。私鑰存放在 MTA 伺服器上，公鑰則會放在 DNS 的 TXT 記錄中。
2. 郵件簽名：MTA 在發送郵件時，使用私鑰對郵件進行簽名，生成一個數字簽名。
3. 驗證簽名：收件人的郵件伺服器收到郵件後，會從 DNS 中查詢公鑰，然後使用公鑰對郵件中的數字簽名進行驗證。如果驗證通過，就表示郵件的完整性得到了保障，並且是由聲稱的發件人發送的。

#### 為什麼私鑰要放在 MTA 伺服器？

簽署郵件： 當 MTA 發送郵件時，會使用私鑰對郵件內容進行簽名，生成一個數字簽名。這個簽名會被添加到郵件的標頭中。

確保安全性： 私鑰是生成數字簽名的關鍵，必須妥善保管。將私鑰放在 MTA 伺服器上，方便 MTA 直接使用，同時也確保私鑰不會被輕易洩露。

#### 為什麼公鑰要放在 DNS？

公開可驗證： 公鑰是公開的，任何人都可以通過查詢 DNS 來獲取。
方便驗證： 收件人的郵件伺服器只需要查詢 DNS 就可以獲得公鑰，方便進行驗證。
總結

DKIM 的私鑰放置在 MTA 伺服器上，而公鑰則放在 DNS 中，這樣一來可以保證郵件的安全性，又方便進行驗證。

### DNS 的 DKIM 設定

DKIM 記錄的名稱格式如下

```txt
[selector]._domainkey.[domain]
```

- selector: 這是您為 DKIM 設定的一個唯一標識符，可以是任何字串，但通常會選擇一個有意義且易於記憶的名稱。
- _domainkey: 這是 IANA (Internet Assigned Numbers Authority) 規定的子域名，用於標示 DKIM 記錄。
- domain: 這是您要驗證的域名，也就是發送郵件的域名(寄件者郵件帳號的域名)。

selector 可以自訂，但是必須一併設定在 MTA 的設定檔內。

考慮以下情境

1. MTA 主機為 mymail.example.com。
2. 寄件者帳號為 `allen@be-yond.com.tw`。
3. 透過 mymail.example.com 主機代表 be-yond.com.tw 發送郵件。例如在 mymail.example.com 主機以 `allen@be-yond.com.tw` 作為寄件者發送郵件。

建立一組 DKIM 金鑰，並替 selector 命名為 mymail.example.com。在負責 be-yond.com.tw 的 DNS 主機中設定以下 DNS 記錄。

```txt
mymail.example.com._domainkey.be-yond.com.tw IN TXT "v=DKIM1; k=rsa; p=[public_key]"
```

設定完上例 DNS 後，還需要在 MTA 主機(此例為 mymail.example.com)設定 selector 和私鑰。

由於 DKIM 設定較為複雜，可再查詢關於 DKIM 設定或 OpenDKIM 相關資料。

## DMARC

>DMARC 由網域所有者設置，告訴收件伺服器如何處理 SPF 或 DKIM 驗證失敗的郵件，並透過接收報告掌握網域郵件的驗證狀況。\
>協議中包含\
>身份驗證：DMARC 會告訴收件伺服器驗證機制，可以單純依據 SPF 或 DKIM，或是需要兩者均驗證通過。如果沒有 DMARC 紀錄，則收件伺服器將會依照預設的規則驗證郵件。\
>處理方式：DMARC 中，透過不同的設定值，可以讓收件伺服器知道，如果收到未通過驗證的郵件時，對郵件進行哪些處置。其中包含回報網域所有者，但不對郵件進行特殊處置、將郵件隔離或標記為垃圾信件，或是直接退回，拒絕未通過驗證的郵件。\
>發送報告：透過 DMARC ，可以獲得使用寄件者網域名義寄出的電子郵件報告，包含郵件的詳細資訊，是否通過驗證或被標記為垃圾郵件的原因，有效幫助寄件者監控郵件狀態與發現潛在風險。\
>資料來源: [電子豹 電子郵件驗證：SPF、DKIM 與 DMARC 的原理與重要性](https://blog.newsleopard.com/spf-dkim-dmarc-email-security/)

### DMARC 設定

搭配 SPF 和 DKIM，在 DNS 記錄中加入

```txt
# 簡易版
_dmarc.[domain] IN TXT "v=DMARC1; p=none;"

# 詳細版
_dmarc.[domain] IN TXT "v=DMARC1; p=none; rua=mailto:[email-address]; ruf=mailto:[email address];"

```

說明如下：

- _dmarc.[domain]：DMARC的記錄名稱，必須以_dmarc開頭，後面是你的域名。
- v=DMARC1：DMARC版本，必須設為DMARC1。
- p=none：策略（policy），有三種可選值：
  - none：不採取任何行動，只是監控。
  - quarantine：將不合規的郵件隔離，標記為垃圾郵件。
  - reject：拒絕不合規的郵件。
rua=mailto:[email address]：Aggregate reports（彙總報告）的接收地址。
ruf=mailto:[email address]：Forensic reports（詳細報告）的接收地址。

## 補充：線上驗證 SPF 或 DKIM 的 web 工具

- [DKIM, SPF, SpamAssassin Email Validator](https://dkimvalidator.com/)
- [SPF Record Check - Lookup SPF Records](https://mxtoolbox.com/spf.aspx)
- [SPF Record Checker - Check SPF Record - SPF Record Lookup](https://dmarcly.com/tools/spf-record-checker)
