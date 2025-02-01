---
title: "WSL安裝 Rocky Linux 9"
date: 2025-01-08 14:00:00 +0800
categories: 
  - linux
tags:
  - WSL
  - Rocky Linux
---

## 安裝 Rocky Linux 虛擬機

[下載 RockyLinux tar 檔](https://docs.rockylinux.org/guides/interoperability/import_rocky_to_wsl/)

```bash
# 進入 powershell 輸入以下指令
# wsl --import <machine-name> <path-to-vm-dir> <path-to/rocky-9-image.tar.xz>
# <machine-name> 為虛擬機的名稱，可自訂
# <path-to-vm-dir> 為虛擬機的儲存目錄，會在該目錄建立虛擬機的主檔(.vhdx)
# <path-to/rocky-9-image.tar.xz> 下載的 Rocky Linux tarball
# 此處將 Rocky Linux9 匯入為 Rocky9

wsl --import Rocky9 ./VM-Rocky9 ./RockyLinux9/Rocky-9-Container-Base.latest.x86_64.tar.xz

# 匯入成功後將會建立一個 MyRocky9 的虛擬機

wsl -l -v

# 輸出
# ...
# Rocky9
# ...

# 若要移除WSL 虛擬機，則可用
# wsl --unregister <machine-name>
# e.g.
# wsl --unregister Rocky9

```

### 登入虛擬機

```bash
# 登入指定的虛擬機
# wsl -d <machine-name>
wsl -d Rocky9

# 若要將 Rocky9 設為預設的虛擬機
# wsl --set-default <machine-name>
wsl --set-default Rocky9

# 若要停止WSL
# wsl --shutdown

# 若要停止指定的虛擬機
# wsl -t <machine-name>
```

### 補充：安裝常用或必須套件

```bash
# 安裝 systemd
dnf install systemd

# 安裝 sudo
dnf install sudo

# 安裝 git
dnf install git

# 安裝 gcc
dnf install gcc
```

### 建立使用者與設定 WSL

```bash
# 使用 root 進行以下操作
# 建立使用者
useradd allen

# 變更密碼
passwd allen

# 將使用者加入 sudo
usermod -aG wheel allen
```

### 補充：設定wsl.conf

```conf
# 為方便登入 WSL
# 可在 /etc/wsl.conf 設定如下

# 啟用systemd
# 必須有安裝 systemd 否則會無法啟動 WSL 虛擬機，必須重新安裝。
# dnf install systemd
[boot]
systemd=true

# 設定預設的登入使用者
[user]
default=allen
```
