---
title: "自架本地端 Vaultwarden 密碼管理伺服器"
date: 2026-02-06 17:15:00 +0800
categories: 
  - Other
tags:
  - Vaultwarden
  - Bitwarden
  - Tools
---

使用 Vaultwarden 作為密碼管理伺服器，並透過 Bitwarden 作為 Client 端，進行個人密碼管理。

## 使用 docker 安裝 Vaultwarden

在專案目錄 `~/Projects/Vaultwarden` 進行以下操作

建立 docker-compose.yml 內容如下

```yml
services:
    vaultwarden:
        image: vaultwarden/server:latest
        container_name: vaultwarden
        restart: always
        environment:
            - DOMAIN=https://vaultwarden.localhost:8099
            - SIGNUPS_ALLOWED=true # 允許註冊使用者
        volumes:
            - ./vw-data:/data

    caddy:
        image: caddy:latest
        container_name: caddy
        restart: always
        ports:
            - 8099:443
        volumes:
            - ./Caddyfile:/etc/caddy/Caddyfile
            - ./vaultwarden.localhost.pem:/etc/caddy/certs/cert.pem
            - ./vaultwarden.localhost-key.pem:/etc/caddy/certs/key.pem
            - ./caddy_data:/data

```

- vaultwarden 為密碼管理伺服器。
- caddy 則是用來做 proxy 服務，將 Host 的 port 代理到 vaultwarden 服務。

接著使用 mkcert 建立本機信任憑證

```bash
mkcert vaultwarden.localhost
# 產生以下檔案 
# vaultwarden.localhost.pem
# vaultwarden.localhost-key.pem
```

接著建立 Caddyfile 進行 proxy 設定

```text
vaultwarden.localhost:443 {
    tls /etc/caddy/certs/cert.pem /etc/caddy/certs/key.pem
    reverse_proxy vaultwarden:80
}
```

reserve_proxy 代理到 vaultwarden 的 Port 80。

{{< alert type="info">}}
reverse_proxy 的對象名稱指定為 docker-compose.yml 內的 services 的服務名稱。
{{< /alert >}}

接著啟動 docker compose

```bash
docker compose up -d
```

即可進入 `https://vaultwarden:localhost:8099` 進行新增使用者帳號。

新增完帳號後，可將 docker-compose.yml 的 `SIGNUPS_ALLOWED` 設為 false，並重啟docker compose。

## 備份與復原

主要備份 vaultwarden 容器的 /data 目錄，在 volumes 中設定為 ./vw-data，可撰寫腳本進行備份下來，備份前須先暫停 docker compose。

```bash
BACKUP_DIR="/home/user/backup"
FILENAME="Vaultwarden_${DATE}.tar.gz"
SOURCE_BASE_PATH="/home/user/Projects/Vaultwarden"
SOURCE_FILE="./vw-data"

docker compose stop > /dev/null 2>&1

tar -czv -f "${BACKUP_DIR}/${FILENAME}" -C ${SOURCE_BASE_PATH} ${SOURCE_FILE} > /dev/null 2>&1

docker compose start > /dev/null 2>&1
```

### 復原

若要復原，直接把備份的 `vw-data` 目錄解開後，讓 vaultwarden 容器的 /data 目錄在 volumes 中指過去就可以了。
