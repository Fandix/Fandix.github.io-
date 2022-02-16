---
title: Day8. Templates and Text interpolation
date: 2021-09-08 10:12:25
tags:
- Angular
- Front-end

categories:
- 2021 鐵人賽
---

前幾天大概講完了 Angular 的 Component 的基本功能與介紹，在很多例子中可以看到在 component.html 中使用了滿多沒看過的語法，比如 `{{ value }}` 或 `(clicl)="onClick()"` 等等的，在前幾天可能會對他覺得非常陌生，別擔心接下來我們會對這些語法有詳細的說明。

在 Angular 中，Templates 是一段 HTML，可以在 Templates 中使用特殊的語法來構建 Angular app 的畫面，那就讓我們繼續往下看吧。

![https://ithelp.ithome.com.tw/upload/images/20210802/201247672iUKFGyC6V.png](https://ithelp.ithome.com.tw/upload/images/20210802/201247672iUKFGyC6V.png)

<!-- more -->


# Empower your HTML
在我們開發 Angular app 時，可以在 Templates 使用特殊的語法來擴展 HTML 的詞彙表，舉例來說可以在 Angular 通過內置模板函數、變量、event  和數據綁定，可以幫助你在開發過程中動態的獲得和設置 DOM 的值。

雖然說 Angular 的 Template 是一段 HTML，但是所有的 templates 都只是整個網頁的一部分，而不是整個頁面，所以不需要在每一個 templates 中添加 `<html>`、`<body>` 或 `<base>` 之類的元素，而出於安全考量 Angular 也不允許在 templates 中加入 `<script>` 。



# Text interpolation
介紹完什麼是 Templates 後接著要來介紹一些 templates 的語法，首先我們要介紹的是 Text interpolation，顧名思義就是將 component.ts 中的變量插入到 templates 中，`當這個變量在 component.ts 中發生變化時，也會同步在 template 中發生變化`，意味著畫面也會跟著變化。

在默認情況下，插值是使用雙花括號 `{{` 和 `}}` 將插值夾在其中，來舉個簡單的例子：
1. 首先先在 app.component.ts 中添加一個變量

    ```typescript
    import { Component } from '@angular/core';

    @Component({
      selector: 'app-root',
      templateUrl: './app.component.html'
    })
    export class AppComponent {
      currentCustomer = 'Fandix'
    }
    ```
2. 在 app.component.html 中將這個值插入到 template 中

    ```typescript
    <!-- app.component.html -->

    <h3>Current customer: {{ currentCustomer }}</h3>
    ```
![https://ithelp.ithome.com.tw/upload/images/20210802/20124767SfgZjpfW0a.png](https://ithelp.ithome.com.tw/upload/images/20210802/20124767SfgZjpfW0a.png)

在畫面可以看到確實有將 component.ts 當中的變量（currentCustomer）的內容呈現出來，那我們將變量的值更改一下

```typescript
currentCustomer = 'Tako'
```

![https://ithelp.ithome.com.tw/upload/images/20210802/20124767zWCwZTqGtJ.png](https://ithelp.ithome.com.tw/upload/images/20210802/20124767zWCwZTqGtJ.png)
當 component.ts 中的變量內容改變後會動態的顯示在畫面中，這就是 Text interpolation 的用途。


# Template expressions
可以在 template expressions 在雙花括號中`{{ }}`產生一個值，Angular 會自動解析表達式並將解析過後的內容分配給綁定目標的屬性，這個目標可以是 HTML Tag、Component selector 或 directive。

## Resolving expressions with interpolation

一般來說在雙花括號之間填入一個模板表達式的話，Angular 會先將他做計算，得到結果後將它轉成字串並顯示出來，舉個簡單的例子，透過差值將兩個數字相加並呈現出來：

```html
<!-- app.component.html -->

<h3>The sum of 1 + 1 is {{1 + 1}}</h3>
```

![https://ithelp.ithome.com.tw/upload/images/20210802/20124767CF8r7xaM47.png](https://ithelp.ithome.com.tw/upload/images/20210802/20124767CF8r7xaM47.png)

Angular 在 templates 的雙花括號中收到了一個表達式，他會自動將這個表達式做計算，像上面的例子中的 `1 + 1`，得到結果後將它轉成字串並顯示出來，所以畫面中會顯示 1 + 1 = 2。

而表達式也可以調用這個 component.ts 中的 method，比如說：

1. 在  app.component.ts 中新增一個 method

    ```typescript
    import { Component } from '@angular/core';

    @Component({
      selector: 'app-root',
      templateUrl: './app.component.html'
    })
    export class AppComponent {
      getValue() {
        return 'Fandix';
      }
    }
    ```
2. 在 app.component.html 中將調用 method 的表達式填入到雙花括號中

    ```html
    <!-- app.component.html -->

    <h3>Hello {{ getValue() }}</h3>
    ```

![https://ithelp.ithome.com.tw/upload/images/20210802/20124767dSHzApckq2.png](https://ithelp.ithome.com.tw/upload/images/20210802/20124767dSHzApckq2.png)

可以看到 Angular 在 template 調用了 component.ts 中的 method ，獲得到的值將他呈現在畫面中，而 Angular 在遇到差值時會執行以下的任務：
1. 計算雙花括號中的所有表達式
2. 將表達式的結果轉換為字串
3. 將結果鏈接到任何相鄰的文字字串
4. 將組合分配給element 或 directive 屬性

## Syntas

一樣的，要運用一個新的程式語言之前，需要先充分熟悉他的語法，不然會產生意料之外的錯誤喔，其實 templates expressions 的語法基本上都跟 Javascript expressions 一樣，但是有幾個特例：

- 你不能使用有會產生 side effects 的 javascript expressions
    - 賦值（=, +=, -=, ...）
    - 運算符（new, typeof, instanceof）
    - 用分號（;）鏈結 expressions
    - 遞增或遞減運算符（++,  --）
    - 還有一些 ES6 的運算符
- 與 javascript 語法有明顯差異的：
    - 不支援 bit 的運算符（|,  &）
    - 新的 template expressions（|,  ?,  !, ...）



# Preventing name collisions

介紹完如何將 component 中的變量插入 template 讓畫面隨著變量動態改變後，要注意的是在使用這個功能時要防止名稱衝突，因為當你使用了 Text interpolationt 插入變量後， Angular 對這個表達式的計算 context 是從 `template variable`、`directive` 和 `component member` 的聯合，所以當你插入了一個名稱到 template 但這個名稱在多個地方都有定義的話， Angular 會用以下的邏輯來確定 context：

1. 在 template 中的變量名稱 (ngFor 中的變量)
2. 在 directive 中的 context 名稱
3. Component 中的變量名稱

所以當你在使用 Text interpolationt  時盡量保持所有變量名稱唯一，不然可能會被 shadowing 掉你原本想要呈現內容，我們舉個例子：

這邊先提早介紹 *ngFor，他也是使用在 template 當中，用法與 javascript 的 for loop 一樣，當你的 component 中的變量是一個 arr 時，想要把這個 arr 中的數值都呈現在畫面上就可以使用這個方法，一樣舉個簡單的例子吧：

1. 首先先在 app.component.ts 中宣告一個 arr 變數並將裡面填上要呈現的內容。

    ```typescript
    import { Component } from '@angular/core';

    @Component({
      selector: 'app-root',
      templateUrl: './app.component.html'
    })
    export class AppComponent {
      displayArray = ['Hello', 'world', 'Angular', 'good'];
    }
    ```

2. 接著在 app.component.html 中使用 *ngFor 將他們全部呈現在畫面上

    ```html
    <!-- app.component.html -->

    <div *ngFor="let content of displayArray">
        {{content}}
    </div>
    ```

![https://ithelp.ithome.com.tw/upload/images/20210802/20124767Dasf05ADZV.png](https://ithelp.ithome.com.tw/upload/images/20210802/20124767Dasf05ADZV.png)

在畫面中可以看到透過 *ngFor 不斷的遞迴將 displayArray 中的內容都顯示出來，這邊暫時介紹到這邊，之後會更詳細的介紹他的使用方法。

大概了解的 *ngFor 的使用方法後，將畫面回到命名衝突，當你使用 *ngFor 命名迭代出來的每一個值得名稱時要注意這個名稱有沒有在其他地方被引用到，如果有則會將其他相同名稱的內容 shadowing 掉，舉個小例子吧。

1. 先在 app.component.ts 中定義兩個變量

    ```typescript
    import { Component } from '@angular/core';

    @Component({
      selector: 'app-root',
      templateUrl: './app.component.html'
    })
    export class AppComponent {
      customers = [ 'Ebony', 'Chiho'];
      customer = 'Padma';
    }
    ```

2. 在 app.component.html 中將這兩個變量呈現在畫面中

    ```html
    <!-- app.component.html -->

    <div>
      <ul>
        <li *ngFor="let customer of customers">
    			<h1>Hello, {{ customer }}</h1>
    			{{ customer }}
    		</li>
      </ul>
    </div>
    ```

![https://ithelp.ithome.com.tw/upload/images/20210802/20124767Gofwpc8OrA.png](https://ithelp.ithome.com.tw/upload/images/20210802/20124767Gofwpc8OrA.png)
在畫面中可以看到，原本我們希望在每次迭代 customers 中的值時，都將 customer = 'Padma' 添加在上面，但是由於 *ngFor 中的變量 shadowing 了 customer 所以導致 'Padma' 無法呈現出來，所以在使用 Text interpolationt 插入變量時要注意有沒有相同的名稱喔！



# 結論

本篇中介紹了如何在 template 中插入 component 的變量以及該注意的事情，最後介紹幾個在使用 Text interpolationt 時最好的設計規範以避免發生錯誤或是增加程式的可閱讀性：

- **使用簡短的表達**：盡可能使用 component 中 property 的名稱或 method 調用。將應用程序和業務邏輯保留在 component 中而 template 只負責調用就可以，這樣以便進行開發和測試。
- **快速執行**：Angular 會在在每個變更檢測週期後執行模板表達式，許多非同步的行為都會觸發變更檢測週期，比如 Promise、HTTP 的結果、timer events、按鍵與滑鼠的移動等等，expression 應該快速的完成不然會讓使用者體驗下降，所以當需要進行長時間計算的事情時，請考慮使用緩存值。
- **沒有明顯的 side effect**：模板表達式不應該更改除了目標屬性值之外的任何應用程序狀態，讀取 component 的值時不應更改到其他的顯示值。



# Reference

- [Angular.io - interpolation](https://angular.io/guide/interpolation)