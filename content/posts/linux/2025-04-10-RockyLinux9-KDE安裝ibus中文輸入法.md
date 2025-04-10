---
title: "Rocky Linux 9 KDE 安裝 ibus 中文輸入法"
date: 2025-04-10 10:50:00 +0800
categories: 
  - Linux
tags:
  - Linux
---

Rocky Linux 安裝 KDE 桌面後，必須自行安裝中文輸入法。

## 安裝 ibus

```bash
# 安裝全部
sudo dnf install ibus-*

# 或只安裝需要的項目
sudo dnf install ibus ibus-gtk ibus-gtk3 ibus-qt ibus-chewing
```

## 將 ibus 加入環境變數

加入到 /etc/environment

```conf
GTK_IM_MODULE=ibus
QT_IM_MODULE=ibus
XMODIFIERS=@im=ibus
SDL_IM_MODULE=ibus
GLFW_IM_MODULE=ibus
```

或加入到 ~/.bashrc

```conf
export GTK_IM_MODULE=ibus
export QT_IM_MODULE=ibus
export XMODIFIERS=@im=ibus
````

首次啟用 ibus

```bash
ibus-setup
```

## 設定進入 KDE 自動啟動 ibus-daemon

若要登入 KDE 自動啟動 ibus，則可加入以下項目

```bash
mkdir -p ~/.config/autostart
nano ~/.config/autostart/ibus-daemon.desktop
```

```conf
[Desktop Entry]
Type=Application
Exec=ibus-daemon -drx
Hidden=false
NoDisplay=false
X-GNOME-Autostart-enabled=true
Name=IBus Daemon
Comment=Start IBus input method daemon
```

完成後重啟系統。
