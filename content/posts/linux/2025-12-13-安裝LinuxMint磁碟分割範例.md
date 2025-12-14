---
title: "安裝 Linux Mint 磁碟分割範例"
date: 2025-12-13 09:15:00 +0800
categories: 
  - Linux
tags:
  - Linux
---

安裝 Linux Mint 分割磁碟時的範例設定。

以 2TB 硬碟為例，作為個人電腦使用的基本分割配置如下

|目錄|建議空間|磁碟格式|標記|
|---|------|-------|---|
|/boot/efi|512MB ~ 1024MB|fat32(vfat)|esp or boot|
|/|300GB ~ 500GB|ext4| |
|swap|和實體記憶體一樣|swap| |
|/home|剩餘空間|ext4| |

{{< alert type="info" >}}
使用 Linux Mint 安裝光碟時，可使用 `GParted` 工具進行分割。  
/boot/efi 可用 GParted 設定 flag，可複選 `esp` 和 `boot`。
{{< /alert >}}
