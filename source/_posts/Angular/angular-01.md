---
title: Day1. 認識 Angular
date: 2021-09-01 10:21:02
tags:
- Angular
- Front-end
categories:
- 2021 鐵人賽
---

在2020年9月，我正式的從韌體工程師轉職成為一位前端工程師，在自學前端的時候我是選擇了 React 這個 Javascript 框架，做了幾個作品後投履歷面試最後找到目前任職的公司，一路上跌跌撞撞但也算順利，不過唯一不太順了就是目前任職的這間公司是使用 Angular ，當初剛進到這間公司時也是從頭開始 K Angular 官方文檔，看得真的是無煞煞，現在也進到這間公司快一年了，也使用過 Angular 做了幾個專案與處理了幾個問題， 想說再次挑戰自己從頭開始認真看 Angular 官方文檔並用自己的理解整理成筆記，希望能對新進的 Angular 開發者有些幫助。

![https://ithelp.ithome.com.tw/upload/images/20210901/20124767Bd7CvcH5JV.png](https://ithelp.ithome.com.tw/upload/images/20210901/20124767Bd7CvcH5JV.png)

<!--more-->

# What is Angular ?
開門見山的說 Angular 是一個 `Javascript 的框架`，他是基於 `TypeScript` 進行開發的，它包括了：

- 基於 Component 且用於構建 Web App 的框架
- 他集成了許多 Library 涵蓋了多種功能，包括 Router, forms, client-server…
- 一套開發人員工具，提供開發者開發、構建、測試與更新程式碼

記得當初主管跟我分享 Angular 的好處，因為 Angular 擁有非常完整的功能，可以透過它提供的功能建立一個完整且大型的 Web App，而且就如他有
非常完整的功能，當在面對多個開發者同時開發專案時，就會遵守共同的開發原則以減少多人開發的問題。



# Angular applications: The essentials
Angular 中有許多核心思想，了解這些核心思想對於之後的開發有非常好的幫助。

## Components
Component 是`構建一個 Web App 的最基本方塊`，每一個 Web App 都是由一個又一個大大小小的 Component 所組成的，在之前學習 React 
的時候也是這個概念，但是 Angular 的 Component 跟 React 非常不一樣，他是一個帶有 `@Component()` 裝飾器的 TypeScript Class、
HTML template, Style ，這邊你可能聽不懂，但沒關係現階段只要簡單記住他是一個有 TypeScript、HTML、CSS 的小方塊就好。
```typescript
import { Component } from '@angular/core';

@Component({
  selector: 'hello-world',
  template: `
        <h2>Hello World</h2>
        <p>This is my first component!</p>
    `
})
export class HelloWorldComponent {
  // The code in this class drives the component's behavior.
}
```

當你要使用這個 Component 時，只需要在 HTML 中將你要用的 selector 當作 tag 使用就好。

```html
<hello-world></hello-world>
```

## Templates
每一個 Component 都擁有自己的 HTML template，它用於在畫面中呈現你這個 Component，當你的 Component 發生改變時 Angular 會
自動更新並且重新渲染 DOM，他有許多非常好用的功能，比如說動態插入、事件綁定等等的，不過這個我們之後會專門講解，在這裡只要記得 Template 
是用來在 UI 上顯示 Component 內容的就可以了。

## Dependency injection
Dependency injection 在 Angular 中是一個非常重要的觀念，簡單來說當你在寫一般的 JavaScript 時，你可能寫了一個 Class 裡面存放了許
多邏輯，當你在使用的時候你可能會需要使用 new 將這個 Class 實例化成一個 Object，然後才能使用裡面的 method 或 property，這樣兩個 Class
就產生了`依賴關係`。

就「依賴」的文字意思來看，是指「一個東西需要另一個東西而存在；若沒有另外一個東西，則本身不能自立」。

物件導向程式(OOP)中，程式是透過許多類別(Class)的實例(instance)，也就是物件(object)，彼此的交互組合來實現各種功能。所以依賴是指「一個物件需要另一個物件才能作用」

而 DI 則是指`被依賴物件透過外部注入至依賴物件的程式中使用`，也就是被依賴物件並不是在依賴物件的程式中使用 `new` 產生而是由`外部注入`，這種注入可以透過 Class 中的
Constructor 或 Setter 實現。

詳細的內容會到後面再仔細講解什麼事 DI，目前只要記住 DI 是 Angular 的一個重要的概念，主要目的是用於將各個 Class 之間`解耦`。

## Angular CLI
Angular CLI 是開發 Angular 應用程序的最快、直接且推薦的方式，就像 React 的 create-react-app 一樣，可以透過指令快速建立出基本的 Angular app，除了建立模板之外還有非常多好用的功能，我會在下一章節中仔細講解 Angular CLI 是什麼。

## First-party libraries
就如上面所提到 Angular 擁有非常多非常完善的資源提供你創建你的 Web App，所以當你在開發過程中遇到需要添加功能時，就可以添加相對應的模組來達成你的目的。

這邊簡單介紹幾個 first party libraries

| Name   |      Description      |
|----------|:-------------:|
| Angular Router |  基於 Component 的客戶端導航與 Router，支援延遲載入、嵌套 Router、自定義路徑匹配等等 | 
| Angular Forms |    表單輸入與表單內容驗證   |
| Angular HttpClient | 可支援 client 端與 server 端的溝通 |
| Angular Animations | 驅動 app 動畫 |



# 結論
本篇章大概介紹了什麼是 Angular 與使用它的好處，有提到許多比較難的觀念，template 的操作與
Dependency injection 等等，現在可能看不太懂不過沒關係，我們後面會詳細講解這些是幹什麼的，當初我也是花了很多時間才大概了解他在幹嘛的 ( 苦笑。



# Reference
- [Angular.io](https://angular.io/guide/what-is-angular)