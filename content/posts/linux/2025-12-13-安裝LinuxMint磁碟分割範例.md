---
title: "安裝 Linux Mint 磁碟分割範例"
date: 2025-12-13 09:15:00 +0800
categories: 
  - Linux
tags:
  - Linux
---

安裝 Linux Mint 分割磁碟時的範例設定。

## 基本分割區

以 2TB 硬碟為例，作為個人電腦使用的基本分割配置如下

|目錄|建議空間|磁碟格式|標記|
|---|------|-------|---|
|/boot/efi|512MB ~ 1024MB|fat32(vfat)|esp or boot|
|/|300GB ~ 500GB|ext4||
|/home|剩餘空間|ext4||

{{< alert type="info" >}}
使用 Linux Mint 安裝光碟時，可使用 `GParted` 工具進行分割。  
/boot/efi 可用 GParted 設定 flag，可複選 `esp` 和 `boot`。
{{< /alert >}}

### /boot 分割區

有些教學會要切 /boot 目錄，但是其實可以只切 / ， /boot 會自動掛載在 / 底下，可省去更新時可能發生 /boot 目錄空間不足的問題。

### swap 分割區

若有 swap 需求，可以改用 swap file 的方式替代傳統的 swap 分割區，較具彈性。

通常 swap 可以設定和實體記憶體(RAM)一樣大。

swap file 可參考

[Linux Swap 置換空間設定](https://www.fooish.com/linux/swap.html)
