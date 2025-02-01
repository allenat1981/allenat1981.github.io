---
title: "檢視 source 指令匯入資料時的 Warnings 訊息"
date: 2025-01-09 10:30:00 +0800
categories: 
  - mysql
tags:
  - MySQL
---

MySQL CLI 介面可以使用 source filename.sql 讀入SQL檔，但若發生 Warnings 時會無法即時檢視，可採用以下方法

## 使用 MySQL 的 tee 指令

登入 mysql 時加上 --show-warnings

```bash
mysql --show-warnings -u username -p
```

進入 mysql CLI 後，使用 tee 指令設定輸出到檔案

```bash
tee /path/to/output.log

# 用 source 讀入 .sql 檔
source /path/to/database.sql
# 所有的輸出都會被記錄到 output.log

# 若要關閉 tee
notee;
```
