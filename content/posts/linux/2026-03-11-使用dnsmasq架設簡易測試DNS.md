---
title: "使用 dnsmasq 架設建議測試 DNS"
date: 2026-03-11 10:05:00 +0800
categories: 
  - Linux
tags:
  - Linux
  - DNS
---

使用 dnsmasq 架設簡單的測試 DNS 服務，讓區網內的裝置可以透過設定 DNS 連線到主機開發環境中的網站。

測試環境如下

- 開發主機(192.168.0.100) 已在 apache 設定 vhost `www.frog.test`。

- iPhone(192.168.0.105) 用來作為用戶端，連線到 `www.frog.test` 進行測試。

## 安裝與設定 dnsmasq

在主機安裝 dnsmasq

```bash
sudo apt install dnsmasq
```

加入設定檔 /etc/dnsmasq/dev-host.conf，內容如下

```conf
# 監聽本機的 127.0.0.1 和 192.168.0.100。
# 監聽主機區網 IP 192.168.0.100 可以限制只服務同網段的 IP，在開發環境中增加安全性。
listen-address=127.0.0.1, 192.168.0.100

# 實體綁訂網卡介面，確保不會流向虛擬網段。增加安全性，且避免與虛擬網段的介面衝突。
bind-interfaces

# 自訂 DNS 指向(可增加多筆)
# 將 www.frog.test 指向開發主機的 IP
address=/www.frog.test/192.168.0.100
```

完成後重啟 dnsmasq 服務

```bash
sudo systemctl restart dnsmasq
```

接著進入網路設定，將 DNS 改為手動，並加入主機 IP `192.168.0.100`，關掉網路再重開。

使用指令 `ping www.frog.test` 若得到指定的 IP 回應，則設定完成。

## 在 iPhone 設定 DNS

iPhone 加入區網 wifi 後，開啟 wifi 設定，將 DNS 改為手動並加入 `192.168.0.100`，關閉 wifi 再重開。即可測試是否能夠用瀏覽器連上 `www.frog.test` 頁面。

{{< alert type="notice" >}}
若 iPhone 仍然無法連線到測試主機，可試著把所有 DNS 主機移除，只保留 `192.168.0.100`。
{{< /alert >}}
