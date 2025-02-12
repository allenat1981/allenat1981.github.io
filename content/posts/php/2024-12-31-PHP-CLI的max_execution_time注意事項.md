---
title: "PHP CLI 的 max_execution_time 注意事項"
date: 2024-12-31 12:03:00 +0800
categories: 
  - php
tags:
  - PHP
---

## PHP max_execution_time

參考資料：[max-execution-time](https://www.php.net/manual/en/info.configuration.php#ini.max-execution-time)

php.ini 預設的 max_execution_time=30，因此 Web Server 運行 php script 時，若超過 30 秒，php 會發出逾時錯誤。

{{< alert type="notice" >}}
Web Server 可能也會有自訂的逾時設定。因此檢查逾時錯誤時，有可能是 php 或 Web Server 發出，要特別注意。
{{< /alert >}}

## PHP CLI 必須自訂逾時時間

在 PHP CLI 執行 php script ，預設的 max_execution_time 則為 0。

可參考 [Differences to other SAPIs](https://www.php.net/manual/en/features.commandline.differences.php)  
>PHP in a shell environment tends to be used for a much more diverse range of purposes than typical Web-based scripts, and as these can be very long-running, the maximum execution time is set to unlimited.

因此若是消耗大量資源的 php script 若持續運行可能會導致系統崩潰

{{< alert type="notice" >}}
排程(cron)執行 php script 時，也是以 PHP CLI 方式運作，所以也要注意。
{{< /alert >}}

因此若要確保 PHP CLI 執行的 php script 會逾時，則必須在檔案內使用 `set_time_limit()` 或 `ini_set('max_execution_time', N)` 來明確設定逾時條件。

```php
// 設定執行超過 10 秒就產生逾時錯誤
set_time_limit(10);
//或使用 ini_set('max_execution_time', 10);

while(true) {
    echo "infinite loop!\n"
}
```

明確設定逾時條件，使用 PHP CLI 時，就同樣會在腳本執行逾時會發出錯誤。
