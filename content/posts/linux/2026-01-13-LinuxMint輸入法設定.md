---
title: "Linux Mint 輸入法設定"
date: 2026-01-13 11:25:00 +0800
categories: 
  - Linux
tags:
  - Linux
  - LMDE
---

Linux Mint(LMDE) 更新 Cinnamon 後，在 kate 無法正確切換新酷音輸入法。

## 原因

輸出 _IM_MODULE 相關環境變數設定

```bash
export | grep _IM_MODULE

# declare -x CLUTTER_IM_MODULE="fcitx"
# declare -x GTK_IM_MODULE="fcitx"
# declare -x QT_IM_MODULE="fcitx"
# declare -x QT_IM_MODULES="wayland;ibus"
# declare -x SDL_IM_MODULE="fcitx"
```

說明如下：

|名稱|作用|範例
|---|---|---
|QT_IM_MODULE|最關鍵的設定。告訴使用 Qt 框架開發的軟體如何連結輸入法。|Kate、VLC、WPS Office、VirtualBox
|QT_IM_MODULES|告訴 Qt 框架優先搜尋的輸入法插件清單。|同上 (通常用於解決 Wayland/X11 相容性)
|GTK_IM_MODULE|告訴使用 GTK 框架開發的軟體如何連結輸入法。|Firefox、Chrome、GIMP、檔案管理器
|CLUTTER_IM_MODULE|針對 Clutter 圖形庫（常與 GNOME 相關元件有關）的輸入法設定。|部分 Cinnamon 內建元件、特定影音軟體
|SDL_IM_MODULE|針對使用 SDL 庫開發的程式（主要是遊戲）。|許多 Linux 原生遊戲、Steam 遊戲、模擬器
|XMODIFIERS|最古老的標準，用來告訴 X11 視窗系統輸入法伺服器的名稱。|所有運行在 X 視窗下的舊式程式

其中看到 QT_IM_MODULES 設定為 `wayland;ibus`，導致 kate 無法正確使用 fcitx 切換輸入法。

## 解決方式

### 個別使用者

在使用者的 ~/.profile 加入

```conf
export QT_IM_MODULES="fcitx"
```

接著登出/登入，重新檢查一次 QT_IM_MODULES 是否調整為 fcitx，即可開啟 kate 正常切換輸入法。

### 所有使用者

若要直接全域設定，則可編輯 /etc/environment。

```conf
# 編輯 /etc/environment
# 不須加 export
QT_IM_MODULES="fcitx"
```

重新開機後生效。
