---
title: "從 Windows 檔案總管存取 WSL 的目錄與檔案"
date: 2025-01-16 16:00:00 +0800
categories: 
  - linux
tags:
  - WSL
---

## 從 windows 存取 WSL 的目錄與檔案

若要從 windows 存取 WSL 的目錄，必須先啟動 WSL，並知道 WSL 的名稱，例如使用 `wsl --list --verbose` 指令

```bash
# 在 windows 的 powershell 或 cmd
wsl --list --verbose

# 可以看見 NAME STATE VERSION 例如
# AlmaLinux-8 Running 2
# ...
```

此時在 windows 檔案總管輸入以下位置

`\\wsl$\AlmaLinux-8` 即可進入到根目錄(/)。

也可以指定更深一層的目錄，例如

`\\wsl$\AlmaLinux-8\home`

即可進入到 `/home` 目錄。
