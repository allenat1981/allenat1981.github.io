---
title: "Rocky Linux 9 KDE 安裝 ibus 中文輸入法"
date: 2025-04-10 10:50:00 +0800
categories: 
  - Linux
tags:
  - Linux
---

Rocky Linux 安裝 KDE 桌面後，必須自行安裝中文輸入法。可以安裝 iBus 和 fcitx5(建議)。

{{< alert type="notice" >}}
若要使用 fcitx5，仍必須先安裝 ibus。
{{< /alert >}}

## 安裝 iBus(必要)

```bash
# 安裝全部
sudo dnf install ibus-*

# 或只安裝需要的項目
sudo dnf install ibus ibus-gtk ibus-gtk3 ibus-qt ibus-chewing
```

將 iBus 加入環境變數

編輯: `/etc/environment`(若檔案不存在則建立新檔)

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
```

## 啟用 iBus

**若要使用 fcitx5 則可略過此步驟。**

首次啟用 ibus

```bash
ibus-setup
```

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

## 安裝 fcitx5

須先安裝 flatpak，請參考 [Rocky Linux Quick Setup](https://flatpak.org/setup/Rocky%20Linux)。

```bash
flatpak install org.fcitx.Fcitx5
flatpak install org.fcitx.Fcitx5.Addon.Chewing
```

安裝完後編輯: `~/.config/autostart/fcitx5.desktop`

```conf
[Desktop Entry]
Type=Application
Exec=flatpak run org.fcitx.Fcitx5
Hidden=false
NoDisplay=false
Comment=Start Fcitx5
X-GNOME-Autostart-enabled=true
X-KDE-autostart-phase=1
```

完成後重啟系統。

參考資料：[透過Flatpak跑Fcitx5，安裝Linux的中文輸入法](https://ivonblog.com/posts/fcitx5-flatpak/)
