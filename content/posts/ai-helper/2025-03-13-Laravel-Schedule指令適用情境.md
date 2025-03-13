---
title: "Laravel Schedule 指令適用情境"
date: 2025-03-13 10:30:00 +0800
categories: 
  - chat-gpt-helper
tags:
  - laravel
---

在 Laravel 專案中，`schedule:work` 和 `schedule:run` 這兩個指令都與排程（Scheduler）相關，但它們的運作方式有所不同。

## schedule:run

`php artisan schedule:run`

- 這個指令**只會執行一次**，它會檢查 `app/Console/Kernel.php` 中定義的排程任務（透過 `schedule` 方法註冊的任務），並執行當下符合條件的任務。
- 通常用於**搭配系統的排程工具（如 Linux 的 crontab）**，例如：

```bash
* * * * * php /path-to-your-project/artisan schedule:run >> /dev/null 2>&1
```

這行 crontab 設定代表「每分鐘執行一次 `schedule:run`」，Laravel 會自行檢查是否有任務需要執行。

## schedule:work

php artisan schedule:work

- 這個指令**會持續運行**，並在背景監聽排程的任務，類似於一個守護程式（daemon）。
- **與 `schedule:run` 最大的不同是：它不需要依賴系統的 crontab，而是自行管理排程的執行。**
- 執行後，Laravel 會每分鐘自行觸發 `schedule:run`，無需 crontab 來觸發：

```bash
php artisan schedule:work
```

- 這個方式適合需要長時間運行的伺服器環境，**但如果伺服器重啟或進程意外終止，則需要手動重新啟動**，因此建議搭配 `supervisor` 來管理。

### 適用情境

| 指令 | 適用情境 |
|------|---------|
| `schedule:run` | 適合搭配 `crontab`，讓系統管理排程 |
| `schedule:work` | 適合獨立運行的環境，例如 Docker、長時間執行的伺服器 |

如果你的專案部署在 Linux 伺服器上，通常建議 **使用 `schedule:run` 搭配 crontab**，這樣可以讓系統確保排程的穩定運行。如果你的專案在 **不支援 crontab 的環境，或希望 Laravel 自行管理排程**，則可以考慮使用 `schedule:work`。
