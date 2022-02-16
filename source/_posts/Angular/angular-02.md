---
title: Day2. Angular CLI
date: 2021-09-02 09:26:52
tags:
- Angular
- Front-end

categories:
- 2021 鐵人賽
---

大家都說"工欲善其事必先利其器"，在我們發開 Angular 時，必需了解一個非常好用的工具， "Angular CLI" ，而這篇文章將會介紹什麼是 Angular CLI、如何安裝與使用 Angular CLI以及一些其他該注意的事項以及常見的命令操作。

![https://ithelp.ithome.com.tw/upload/images/20210821/20124767FXInjTdVpO.png](https://ithelp.ithome.com.tw/upload/images/20210821/20124767FXInjTdVpO.png)

<!--more-->

# What is the Angular CLI ?
在 Angular 的官方文檔中提到， Angular CLI 是 Angular 團隊創建的工具，用於管理、構建、維護和測試你的 Angular 專案。

可能有些剛接觸前端的新手會覺得又要記一些指令覺得麻煩，實際上有一個 [Viisual Studio Code](https://marketplace.visualstudio.com/items?itemName=sasxa-net.angular-gui) 的擴展可以使用，它提供了一個 Angular CLI GUI 介面，可以讓你取代在終端機中輸入命令。

![img](https://i.imgur.com/o8zEERJ.gif)

雖然有 GUI 介面可以使用，但是本篇章還是主要會介紹如何透過在終端機中下達命令來使用 Angular CLI，別放棄學習啊！！


# Do I have to use the Angular CLI ?
現在我們知道了什麼是 Angular CLI，可能很多人都會問，既然他是自動幫開發著處理一些問題的，那我一定要用嗎？我可以不要這麼自動（？

答案是當然可以，你可以再開發 Angular 的時候，無論創建、維護、測試等等的都用手動的方式建立或操作，但是你可能會變成.... foolish Angular developer (別篇文章說的下面會放連結，罵他不要罵我）。

Angular CLI 存在的目的是為了讓開發者專心進行專案開發，它負責將麻煩與耗時的動作自動化，使用指令可以自動化生成一個基礎並帶有 .gitgnore( 用來告訴 git 應該忽略哪些 file ) 的 Angular app，其中包括 Angular 的核心部分比如 component, module 等等，除了創造 app 之外他還可以自動化進行 unit test、building和其他基本卻複雜的操作。

所以說了這麼多，有沒有更想使用 Angular CLI 啦（不要說沒有！


# How to install the Angular CLI
終於要進入到安裝的環節了，在安裝 Angular CLI 之前比需要先安裝  [Node.js](https://node.js.org/) ，如果不想使用終端機命令安裝 Node.js 可以在 [Node.js 官網下載](https://nodejs.org/en/download/) 並進行安裝。

當安裝完 Node.js 後，可以在終端中輸入指令正式安裝 Angular CLI
```bash
npm install -g @angular/cli
```

> 使用 -g option 代表要在系統範圍內（全域）安裝特定的 npm module，如果沒有使用 -g option 則會安裝在當前目錄的 node_modules 中。

如果你需要更新 Angular CLI 到最新版本則可以使用
```bash
npm update -g @angular/cli
```

這樣就完成在全域中安裝 Angular CLI 的方式，這意味著你可以在你機器的任何地方使用它，有趣的是當創建一個 Angular app 時，CLI 也將安裝在本地，這意味著他也會安裝在你 Angular app 的 node_module 中，讓你可以在你的專案中使用 Angular CLI 的功能。



# How to create an Angular application with the CLI
當要創建一個新的 Angular app 時，可以使用 Angular CLI 他可以自動且快速的建立所有需要的文件（如果手動創建可能要花一天的時間，那麼我們就來使用指令吧！

```bash
ng new MyApplicationName
```
當按下 enter 後你就可以悠閒的喝個咖啡聊個天等待他自動完成

當 Angular CLI 在創建 App 的時候會問一些問題:
1. 是否需要設置 Router (y / N) → 選擇 N 之後也可以手動加入 Router
2. 選擇想要的樣式格式 → 選擇要使用 CSS, SASS, SCSS 等不同的 style format
   ![https://ithelp.ithome.com.tw/upload/images/20210727/20124767MZ8BUaS7ww.png](https://ithelp.ithome.com.tw/upload/images/20210727/20124767MZ8BUaS7ww.png)

當都選擇好後就可以等他完成啦～
![https://ithelp.ithome.com.tw/upload/images/20210727/20124767WZkErowhaO.png](https://ithelp.ithome.com.tw/upload/images/20210727/20124767WZkErowhaO.png)



# CLI command-language syntax
在使用每一個程式語言的時候，都必須要注意他的語法，不然很容易造成出乎自己意料的結果，CLI syntax如下：
```
ng commandNameOrAlias requiredArg [optionalArg] [options]
```
- 大部分的 command 和一些 option 擁有別名，可以使用簡單的方式達到一樣的目的。
- 使用雙破折號(—)放在 option 前面當作前綴，使用單破折號(-)放在 option 別名前面當作前綴，而參數沒有前綴。
- 通常生成文件的名稱可以將它當作參數加在後面，也可以使用 —name 來指定文件名稱。
- 參數和 option 名稱可以使用駝峰命名或破折號 → —myOpitonName = —my-option-name



# How to use Angular CLI
上面介紹了這麼多的 Angular CLI 來源、用途、語法等等，接下來就要進到最重要的環節，來介紹一些常用的 CLI 指令。

## Create Component
下面是創建一個新 Component 的 Angular CLI 指令。
```
ng generate component MyComponentName -> 完整
ng g c MyCompoenntName -> 簡寫
```
如果你要將這個創建出來的 Component 歸屬於特定的 module 則可以使用
```
ng generate component MyComponentName --module MyModuleName
```
## Create Module
在 Angular 中 Module 的觀念非常重要，他可以有效地模塊化你的程式，詳細的 Module 我們後面再慢慢介紹，這邊就先了解如何快速創建一個 Module 吧。
```
ng generate module MyModuleName
```
## Create pipes
pipes 也是 Angular 一個重要的觀念，他可以將你的資料自動地進行轉換，一樣詳細的內容我們放到後面介紹，這邊一樣介紹如何快速建立 pipes。
```
ng generate pipe MyPipeName
```
## Create Services
Services 在 Angular 中也是非常重要的，他可以有效地將畫面與計算邏輯分割開來，一樣詳細的內容我們放到後面介紹（瘋狂挖坑...
```
ng generate service MyServiceName
```
## Run project
當我們撰寫好了我們的 Angular 程式後，最重要的就是讓他跑起來，哪們就可以使用這個指令
```
ng serve <project> [options]
ng s <project> [options]
```
使用這個指令會自動 build 和 serve 你的 App，並且在你`更改內容儲存後重新構建`。
## Building project
當我們撰寫好程式後，我們可以將我們寫的程式 build 好，以便做更多的處理，這時就可以使用這個指令
```
ng build <project> [options]
ng b <project> [options]
```
## Extract-i18n
Angular 支持多語言的設計，後面會詳細講解 i18n 的使用方法，這邊一樣先介紹如何編譯 i18n
```
ng extract-i18n <project> [options]
ng i18n-extract <project> [options]
ng xi18n <project> [options]
```


# 結論
本篇章介紹了許多 Angular CLI 的概念，我在剛接觸 Angular 的時候對 CLI 也是模模糊糊，雖然要真正了解 CLI 內部是如何操作的十分困難，但對於一般開發者而言只要懂的使用就可以了，上面也介紹了許多 CLI 的指令，但這只是冰山一角我只介紹了幾個我自己比較常用到的，有興趣或是有使用到的話也可以到 Angular 官網中查詢需要使用到的指令，希望今天的講解有讓你稍微了解一點什麼是 Angular CLI 也算使用 Angular 的第一步吧。


# Reference
- [How to install and use the Angular CLI](https://itnext.io/how-to-install-and-use-the-angular-cli-ac8b5aae1d05)
- [Angular.io](https://angular.io/cli)