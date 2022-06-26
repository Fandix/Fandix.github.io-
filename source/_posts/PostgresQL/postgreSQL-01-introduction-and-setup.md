---
title: PostgreSQL - Introduction and Setup
date: 2022-06-26 13:54:20
tags:
- Back-end
- PostgreSQL

categories:
- PostgreSQL
---

由於在公司中的專案開發是使用 `PostgreSQL` 進行數據的存儲，所以希望能夠對 PostgresQL 有更近一步的了解，從基本的操作到比較複雜的效能或安全問題等等，藉由 Udemy 和 PostgreSQL 官網進行學習的紀錄。

<img src="https://www.ovhcloud.com/sites/default/files/styles/text_media_horizontal/public/2021-09/ECX-1909_Hero_PostgreSQL_600x400%402x.png" alt="drawing" width="900"/>

<!-- more -->

# What is PostgreSQL ?
PostgreSQL 是一個企業級的`開源的關連式數據庫`，所謂的關聯式數據庫簡單來說所有的資料是由 `資料表 (Table)`, `紀錄 (Record)`, `欄位 (Field)` 和 `資料 (Data)` 所構成的，之所以會是關聯式那是因為儲存的資料是具有互相關聯的關係，不同 Table 的資料可以透過特定的欄位將資料串接起來，比如說可以利用互相關聯的方式將 `使用者資料表` 與 `購物車資料表` 之中的數據串連起來，就可以知道某一個使用者的購物車中有存放哪一些數據，這就是關聯式的目的。

---

# Setup PostgreSQL In MacBooks
要在 MacOS 的環境中安裝 PostgreSQL，首先需要先在 [PostgresQL 官網](https://www.postgresql.org/download/) 中下載，下載安裝完後可以在將其點開便會看到 PostgreSQL 的畫面。

<img src="https://ithelp.ithome.com.tw/upload/images/20220626/20124767xshryFER4p.png" alt="drawing" width="900"/>

接著可以點擊任何一個 DataBase 便會自動開啟 `Terminal` 並連接上該 DataBase
<img src="https://ithelp.ithome.com.tw/upload/images/20220626/20124767MWRNuCDXtJ.png" alt="drawing" width="900"/>

而如果要從 Terminal 中直接使用 PostgreSQL 的 CLI 的話，需要將安裝 PostgreSQL 的路徑添加到 `.zshrc` file 中。
```bash
psql
## zsh: command not found: psql

vim .zshrc

## Add PATH
export PATH=$PATH:/Applications/Postgres.app/Contents/Versions/14/bin

## Reload
source .zshrc
```

設定完成後就可以直接在 Terminal 中輸入 `psql` 便可以使用 CLI 了
<img src="https://ithelp.ithome.com.tw/upload/images/20220626/20124767iyM6583DzJ.png" alt="drawing" width="900"/>

---

# Install PostgreSQL's GUI -> PGAdmin 
除了使用 Terminal 輸入 PostgreSQL 的 CLI 對 DB 進行操作之外，還可以使用 PostgreSQL 提供的 GUI `pgAdmin` 來達到一樣的目的，首先一樣先在 [pgAdmin 的官網](https://www.pgadmin.org/download/) 中下載並安裝，

當安裝完後便可以打開 pgAdmin，之後會看到要輸入密碼，輸入完設定的密碼後便可以看到目前本機中有哪些 DataBase。
<img src="https://ithelp.ithome.com.tw/upload/images/20220626/20124767rm2KssoxGu.png" alt="drawing" width="900"/>

<img src="https://ithelp.ithome.com.tw/upload/images/20220626/20124767TgmSLHULWc.png" alt="drawing" width="900"/>

## 新增 Localhost Database
要新增 localhost database 的話，需要在 server 上點擊右鍵選擇 `Create` -> `Server...`
<img src="https://ithelp.ithome.com.tw/upload/images/20220626/20124767rzY4pFF5L0.png" alt="drawing" width="900"/>

接著在 Name 中輸入 `localhost` 並在 `Connection` 中的 hostname 也輸入 `localhost`，最後在 `Username` 中輸入你電腦中的使用這名稱，如果不知道使用者名稱可以在 Terminal 中輸入 `whoami` 就可以看到了
```bash
$ whoami
fandixhuang
```
- 輸入 Name 為 localhost
<img src="https://ithelp.ithome.com.tw/upload/images/20220626/20124767Seus1bhqgL.png" alt="drawing" width="760"/>

- 在 Connection 中輸入 hostname 和 username
<img src="https://ithelp.ithome.com.tw/upload/images/20220626/20124767iRt1Pwx90D.png" alt="drawing" width="760"/>

由於預設 Port 就是 5432，所以可以不用更改，不過如果有更改過預設的 Port 的話，記得要在 Connection 中設定更改過的 Port。

