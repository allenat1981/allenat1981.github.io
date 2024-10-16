---
layout: single
title: "python 開發環境設定 in WSL"
date: 2024-10-16 16:00:00 +0800
category: python
tags:
  - python
---

# python 開發環境設定(WSL)

## 說明

1. 使用 WSL 安裝 Rocky Linux
2. 安裝 python 版本控制套件 pyenv
3. 使用 venv 建立 python 虛擬環境

## WSL 的 Rocky Linux 環境

### 安裝 Rocky Linux 虛擬機

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

## 安裝 pyenv

pyenv 可用來版控安裝的 python 版本。參考 [github:pyenv](https://github.com/pyenv/pyenv) 進行安裝。

```bash
# 將 pyenv 安裝在 ~/.pyenv
mkdir .pyenv

git clone https://github.com/pyenv/pyenv.git ~/.pyenv

# 嘗試編譯動態 Bash 擴充功能以加速 Pyenv。如果失敗也不用擔心； Pyenv 仍將正常運作
cd ~/.pyenv && src/configure && make -C src

# 將 pyenv 路徑加入環境變數 PATH
# 可設定到以下檔案(擇一)
# ~/.bashrc
# ~/.profile
# ~/.bash_profile
# ~/.bash_login
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bashrc
echo 'command -v pyenv >/dev/null || export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bashrc
echo 'eval "$(pyenv init -)"' >> ~/.bashrc

# 重新讀取 .bashsr
source ~/.bashrc

# 測試
pyenv versions
# 若成功會出現目前python版本
```

### 安裝 python

使用 pyenv 安裝 python 前，請參考安裝依賴套件

[pyenv suggested-build-environment](https://github.com/pyenv/pyenv/wiki#suggested-build-environment)

```bash
# Rocky Linux9 環境參考以下安裝項目
sudo dnf install make gcc patch zlib-devel bzip2 bzip2-devel readline-devel sqlite sqlite-devel openssl-devel tk-devel libffi-devel xz-devel libuuid-devel gdbm-libs libnsl2
```

```bash
# 列出可以安裝的 python 版本
pyenv install --list

# 安裝 python 3.12.6
pyenv install 3.12.6

# 將全域環境改為 python 3.12.6
pyenv global 3.12.6

# 檢查是否切換成功
pyenv versions

# 輸出如下
# system
# * 3.12.6(...)
```

## 建立 venv 虛擬環境

在 ~/py/django/.venv 建立虛擬環境

```bash
# 建立目錄
mkdir -p ~/py/django/.venv

# 建立虛擬環境
# --prompt <Prompt-name> 替虛擬環境設定一個名稱，提供辨識
python -m venv --prompt django ~/py/django/.venv 

# 啟動虛擬環境
source ~/py/django/.venv/bin/activate

# 離開虛擬環境
deactivate
```

若要刪除虛擬環境，可直接將虛擬環境目錄刪除即可。本範例中即為 `~/py/django/.venv`。

### 補充：建立腳本來啟用虛擬環境

在 ~/py/django 目錄下建立 venv-activate.sh 腳本檔，內容如下：

```bash
#!/bin/bash
source ~/py/django/.venv/bin/activate
```

執行腳本

```bash
source ./venv-activate.sh
```
