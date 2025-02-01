---
title: "VSCode 設定 WSL 的 umask"
date: 2025-01-16 11:00:00 +0800
categories: 
  - linux
tags:
  - VSCode
  - WSL
---

## 如何設定 vscode 在 wsl 系統的 umask 權限

即使在 WSL 內編輯使用者的 .bashrc 檔，加入 umask(例如 0002)，也只會對登入後的 bash 生效。

在 VSCode 透過遠端總管連線到 WSL 系統後，新建檔案或目錄時，仍然會參照 VSCode 預設的 umask，因此需要進行以下設定。

登入 WSL 系統，新增 `~/.vscode-server/server-env-setup`，並在其中設定 umask 0002。

```bash
# 登入 WSL

echo "umask 0002" > ~/.vscode-server/server-env-setup"
```

之後重新用 VSCode 連進 WSL 即可。

## 參考資料

[Incorrect permissions for new files / folders #2009](https://github.com/microsoft/vscode-remote-release/issues/2009)
