---
title: "Rocky Linux 9 設定 clamAV"
date: 2025-04-11 10:50:00 +0800
categories: 
  - Linux
tags:
  - Linux
---

clamAV 為 Linux 系統常用的防毒套件。

## 安裝 clamAV

參考資料：

- [clamav](https://reintech.io/blog/installing-using-clamav-antivirus-rocky-linux-9)
- [rockylinux 9安裝防軟體clamAV](https://blog.yslifes.com/archives/3135)

```bash
sudo dnf install clamav clamd clamav-update
```

## 更新病毒碼

```bash
sudo freshclam

# 自動更新病毒碼
sudo systemctl enable clamav-freshclam
```

## 設定 clamAV 自動防護（on-access scanning）

**On-access：你打開某個文件，防毒會馬上檢查這個文件有沒有毒。**

編輯主設定檔

```bash
sudo vi /etc/clamd.d/scan.conf
```

取消以下註解

```conf
LocalSocket /run/clamd.scan/clamd.sock
FixStaleSocket true
```

說明：

- `LocalSocket` 設定 clamd 的本地 socket 路徑，讓其他工具能與它溝通。
- `FixStaleSocket` 啟動時自動清除舊 socket，避免錯誤發生。

啟動並啟用 clamAV 守護進程

```bash
sudo systemctl enable --now clamd@scan
```
