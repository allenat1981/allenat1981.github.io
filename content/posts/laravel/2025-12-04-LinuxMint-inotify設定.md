---
title: "Linux Mint 開發 Laravel 出現 Error: ENOSPC: System limit for number of file watchers..."
date: 2025-12-04 13:30:00 +0800
categories: 
    - laravel
tags:
    - laravel
    - Linux
---

在 Linux Mint LMDE 版本使用 `composer run dev` 時，出現 `Error: ENOSPC: System limit for number of file watchers reached, ...` 錯誤。

代表 Vite（或 Node）想監控的檔案太多，超過系統可用的 inotify watchers 數量上限。這在 Linux（包括 Debian / Ubuntu）非常常見，特別是 Laravel + Vite 專案檔案多時。

Linux 用 inotify 監控檔案變更（Vite dev server 需要監控 JS、CSS、Blade 等檔案）。
預設限制通常只有 8192 或 16384 watchers。
但 Vite、Webpack 或大型專案可能需要幾萬甚至十幾萬個 watcher。

當數量不夠，就會出現：

ENOSPC: System limit for number of file watchers reached...

### 檢查目前系統的 watcher 數量

```bash
cat /proc/sys/fs/inotify/max_user_watches
cat /proc/sys/fs/inotify/max_user_instances
# 預設 max_user_watches 為 65535
```

若要解決此問題需要自訂 max_user_watches 上限。

基本的設定檔可能在 `etc/sysctl.conf`，但不同的 Linux 發行版可能不同，因此可以先查詢

```bash
ls | grep sysctl
```

以 Linux Mint 為例，原始設定檔位於 `/etc/sysctl.d/lmde.conf`，故可在同一目錄下建立 x-custom.conf 檔案來覆蓋原本設定。

```conf
# 建立 /etc/sysctl.d/x-custom.conf

fs.inotify.max_user_watches=524288
fs.inotify.max_user_instances=1024
```

以上將 max_user_watches 設為 524288；max_user_instances 設為 1024。

接著重新套用設定

```bash
sudo sysctl --system
```

重新檢視設定是否成功

```bash
cat /proc/sys/fs/inotify/max_user_watches
cat /proc/sys/fs/inotify/max_user_instances

# 若成功則輸出 x-custom.conf 的設定。
```

設定成功後可再用 `composer run dev` 運行開發環境。
