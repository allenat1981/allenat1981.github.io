---
title: "使用 ssh 連線到 github.com"
date: 2024-12-27 12:07:00 +0800
categories: 
  - git
tags:
  - github
---

## 說明

1. 在本機端建立 ssh 使用的金鑰。
2. 將公鑰設定在 github。
3. 使用 ssh 連線到 github。

## 參考資料

[【Git】使用 SSH 金鑰與 GitHub 連線](https://cynthiachuang.github.io/Generating-a-Ssh-Key-and-Adding-It-to-the-Github/)

## 在本機建立使用者的 ssh 金鑰

首先在本機建立 ssh 金鑰

```bash!
# 建立使用者密鑰
ssh-keygen -t rsa -b 4096 -C "<email>"
# 預設將密鑰檔輸出至 ~/home/.ssh 目錄
# id_rsa 私鑰檔
# id_rsa.pub 公鑰檔
```

1. 登入到 github，點擊使用者名稱的 settings。
2. 連入 settings 頁面，點擊 Access 下的 SSH and GPG keys
3. 在 SSH keys 區塊加入 New SSH key
    - Title 設定金鑰的名稱(輸入容易辨識的名稱 e.g. Allen PC)。
    - Key type 選 Authentication Key。
    - Key 輸入 id_rsa.pub 的內容。
4. 在本機輸入 `ssh -T git@github.com` 測試是否連線成功。

### 補充：將私鑰加入 ssh-agent

若不想每次 ssh 連到 github 都需要輸入密碼，則可以使用 ssh-agent

```bash!
# 將 ssh-agent 加入 shell 環境變數
eval "$(ssh-agent -s)"

# 將密鑰 id_rsa 加入 ssh-agent
ssh-add ~/.ssh/id_rsa

# 檢視加入 ssh-agent 的密鑰
ssh-add -l
```

### 補充：變更金鑰名稱

建立金鑰可加上 `-f <key-file-name>` 用來變更產出金鑰的名稱。可以建立容易辨識此金鑰用途的名稱，例如：`-f for_github`。將會產出：

- `for_github` 私鑰
- `for_github.pub` 公鑰

使用 ssh 連線到 github 時，預設會使用前綴為 id_ 的檔名，所以必須在 .ssh 目錄下建立 config 檔案，指定連線使用的金鑰檔。

建立/編輯 ~/.ssh/config

```conf!
# 目前測試 User 可以任意輸入或不輸入，測試登入仍然會是當初在 github 註冊的帳號
# 建議 User 也可以直接設定 github 帳號

Host github.com
  HostName github.com
  User <user-name>
  IdentityFile ~/.ssh/<key-file-name>
```

建立完成後使用 `ssh -T git@github.com` 測試即可。

## 結論

設定成功後，即可以透過 ssh 進行 repo 的 push/pull。
