---
title: Docker - Dive Into Docker!
date: 2022-07-05 14:53:12
tags:
- Back-end
- Docker
- CI/CI

categories:
- Docker
---

本章會介紹什麼是 Docker 與為什麼需要使用 Docker，現在的專案中大多都會使用非常多額外的套件，而不同專案不同套件在不同的機器上常常會因為沒有安裝某些東西而導致無法使用，而 Docker 的思想就是 `Build and Ship any Application Anywhere`，可以讓開發者不考慮機器與環境的問題就可以建置與運行某個應用程式，可以快速的建立與發佈我們所撰寫的應用程式，以開發的角度來說，可以專注於應用程式的開發而不需要花心思與精力在環境建置。
<img src="https://d1.awsstatic.com/acs/characters/Logos/Docker-Logo_Horizontel_279x131.b8a5c41e56b77706656d61080f6a0217a3ba356d.png" alt="drawing" width="900"/>

<!-- more -->

# Why Use Docker?
當要安裝某個軟體在本機使用時，可能會遇到滿多問題，比如說要安裝 `redis` 這個 open source database 的話，如果跟官網的操作進行下載可能會遇到一些問題

```bash
$ wget https://download.redis.io/releases/redis-6.2.6.tar.gz
$ tar xzf redis-6.2.6.tar.gz
$ cd redis-6.2.6
$ make
```

上面是 redis 官方提供的 Installation 方法，如果今天是一台新的電腦那麼用他的方式安裝就會發生問題，比如說
<img src="https://ithelp.ithome.com.tw/upload/images/20220705/20124767pHtkLpsCEp.png" alt="drawing" width="860"/>

因為我的電腦先前並沒有安裝 wget 這個套件，所以無法理用官方提供的方式安裝，這時我們可能會 google 看看要如何解決這個問題，當解決完一個問題後有可能會在出現另一個問題，當所以有問題解決完後才能完成安裝，而 Docker 就是為了解決這種問題，他主要的目的就是
> Why we use Docket - Docker makes it really easy to install and run software without worrying about setup or dependencies

---

# What is Container?
要介紹什麼是 Container 之前需要先大概了解一下作業系統是如何運作的
<img src="https://ithelp.ithome.com.tw/upload/images/20220705/20124767ayj04lMWKo.png" alt="drawing" width="860"/>

在作業系統中有個重要的部分是用來管理應用軟體所發出的資料 I/O 他稱為 `Kernel`，他會將這些 I/O 要求轉譯為指令並交由相對應的部分進行處理，比如說 Spotify 軟體會要求讀取音樂檔案的要求，所以 `Kernel` 就會將這個要求轉譯為指令送給 `Memory` 進行音樂檔案的讀取，在比如說如果使用 `NodeJS` 的 `writeFile` 功能的話，就會將這個請求送給 `Kernel` 接著將這個請求轉換為指令發送到 `Hard Disk` 進行寫檔的操作。

那麼比如說你的電腦只安裝的 `Python v2` ，但是今天有兩個應用軟體個別使用的是 `Python v2` 和 `Python v3` 那麼由於電腦只知道 `Python v2` 所以使用 `Python v3` 的軟體就無法順利執行，就跟最上面要安裝 `redis` 一樣，因為我的電腦中並沒有安裝 `wget` 所以我的電腦不知道這是什麼所以無法做出相對的動作。

那麼要怎麼解決這個問題？其實可以使用作業系統提供的 `name spacing` 的功能
<img src="https://ithelp.ithome.com.tw/upload/images/20220705/20124767EET6vxek28.png" alt="drawing" width="860"/>

簡單來說 `name spacing` 就是在電腦中隔出一個小區間，所以如果當 Chrome 需要操作到 Hard Disk 的話， `Kernel` 就會知道他是需要使用到 `Python v2` 於是會將這個請求發給帶有 `Python v2` 的 Hard Disk name spacing，反之如果是 `NodeJS` 需要操作 Hard Disk 那麼他就會將這個請求發給帶有 `Python v3` 的 Hard Disk name spacing，這樣就可以讓不同的應用程式在同一個機器上執行。

上面介紹了如何 name spacing 在同一台機器中運行不同的應用軟體，那麼回到正題， `Container` 是什麼？可以把 `Container` 想像成是一個小型的 `name spacing`
<img src="https://ithelp.ithome.com.tw/upload/images/20220705/20124767fn7GaiurEm.png" alt="drawing" width="860"/>

一個 Conatiner 是一個 Process 它具有一組特定的資源以供給應用程式使用
<img src="https://ithelp.ithome.com.tw/upload/images/20220705/20124767J99asQMtUi.png" alt="drawing" width="860"/>

以上面的例子來說，今天有一個 `Container` 他是一個 Process 並提供了 `Chrome` 這個應用程式需要使用到的所有資源 ( Hard Drive, Network, RAM, CUP... )。

了解了什麼是 `Container` 之後可能還會有個疑問，那麼 `Image` 和 `Container` 是什麼關係？來看一張圖
<img src="https://ithelp.ithome.com.tw/upload/images/20220705/20124767LJpV6I2yHy.png" alt="drawing" width="860"/>

如果今天有一個 Image 裡面包含了 Chrome 和 Python 兩種執行 >Run Chrome 的軟體，但如果是一般的電腦因為缺乏 Chrome 和 Python 所以無法執行 >Run Chrome ，所以當利用這個 Image 創建出一個 Container 之後，這個 Container 就會包含 Chrome 和 Python
<img src="https://ithelp.ithome.com.tw/upload/images/20220705/20124767sdjJEKrEs1.jpg" alt="drawing" width="860"/>

---

# Reference
- [Docker and Kubernetes: The Complete Guide](https://www.udemy.com/course/docker-and-kubernetes-the-complete-guide/)

