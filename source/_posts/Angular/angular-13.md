---
title: Day13. Built-in directives - attribute
date: 2021-09-13 10:54:04
tags:
- Angular
- Front-end

categories:
- 2021 鐵人賽
---

在介紹完 component 與 template 後，接著要來介紹什麼是 dierctive，可能在前面的章節中多多少少都有提到，但是可能都沒有詳細的講解過，也有遇過實際上已經有在使用了但卻不知道這個其實就是 directive，不過沒關係接下來會詳細的介紹什麼是 directive。

directive 是 Angular 中為了應用程序元素額外添加行為的 class，可以使用 Angular 內建的 directive 來管理 `form` 、`lists`、 `style` 和 `使用者看到的內容`。

而 directive 有不同類性的用法，以下分為三種：

1. Component: 帶有 template 的 directive，這種是最常見的 directive
2. Attribute directive: 改變 element, component 或其他 directive 的外觀或行為的 directive
3. Structureal directive: 通過添加或刪除 DOM element 來改變 DOM 的佈局


![https://ithelp.ithome.com.tw/upload/images/20210807/20124767SxBTFn4QZA.png](https://ithelp.ithome.com.tw/upload/images/20210807/20124767SxBTFn4QZA.png)

<!-- more -->

# Built-in attribute directives

attribute directive 是用來監聽和修改 HTML 的 element、attribute、property 和 component 的行為，比如 RouterModule 或 FormsModule 這類型的 NgModule，他都有定義自己的 attribute directive，而常見的的 attribute directive 有：

- NgClass: 添加或刪除一組 css class
- NgStyle: 添加或刪除一組 HTML style
- NgModule: 像 HTML 的 `<form>` element 添加雙向數據綁定


# Adding and removing classes with NgClass

可以使用 ngClass 同時添加或刪除一個或多個 CSS class，而如果要添加或刪除一個 CSS class 則不要使用 ngClass，請使用 class binding。

## Using NgClass with an expression

要在設置樣式的 element 上添加 [ngClass] 並將一個表達式賦予給他，來舉個子吧
```html
<div [ngClass]="isSpecial ? 'special' : ''">This div is special</div>
```

在上面的例子中可以看到將一個`三元運算子`作為表達式賦予給 [ngClass]，所以當 `isSpecial` 為 true 時，就會在這個 `<div>` element 中套用名為 ` special` 的 CSS class 反之則不套用任何 class。

## Using NgClass with a method

除了將表達式賦予給 [ngClass] 之外，也可以與 component 中的 method 一起使用，舉個例子吧，希望透過畫面中三個 check box 來更換畫面中的字體或顏色。

1. 在 app.component.html 中建立三個 chech box 與一個按鈕
    ```html
    <!-- app.component.html -->

    <div [ngClass]="currentClasses">
      This div element CSS class is initial color, size and bgcColor
    </div>

    <ul>
      <li>
        <label for="colorChange">Change color</label>
        <input type="checkbox" id="colorChange" (change)="onChange('color')" />
      </li>
      <li>
        <label for="fontSizeChange">Font size change</label>
        <input type="checkbox" id="fontSizeChange" (change)="onChange('size')" />
      </li>
      <li>
        <label for="bgcChange">Background color change</label>
        <input type="checkbox" id="bgcChange" (change)="onChange('bgcColor')" />
      </li>
    </ul>

    <button (click)="onSetCurrentClasses()">Change div CSS class</button>
    ```

    - (1): 在最上面添加一個 `<div>` 並將他使用 [ngClass]="currentClasses"，透過 currentClasses 的內容決定要將幾個 CSS class 綁定在上面
    - (2): 新增三個 check box 並將他們綁定一個 method，當使用者點擊時改變他們各自的狀態
    - (3): 當使用者選擇好 check box 後，點擊按鈕觸發重新設置 currentClasses
2. 在 app.component.ts 中新增 property 與 method
    ```typescript
    import { Component } from '@angular/core';

    @Component({
      selector: 'app-root',
      templateUrl: './app.component.html',
    })
    export class AppComponent {
      color = false;                                      // (1)
      size = false;
      bgcColor = false;
      currentClasses = { color: false, size: false, bgcColor: false };

      onChange(checkboxId: string) {                      // (2)
        switch (checkboxId) {
          case 'color':
            this.color = !this.color;
            break;
          case 'size':
            this.size = !this.size;
            break;
          case 'bgcColor':
            this.bgcColor = !this.bgcColor;
            break;
          default:
            break;
        }
      }

      onSetCurrentClasses() {                             // (3)
        this.currentClasses = {
          color: this.color,
          size: this.size, 
          bgcColor: this.bgcColor
        }
      }
    }
    ```

    - (1): 新增每個 check box 所控制的 property，並新增 currentClasses 用於改變整體樣式
    - (2): 新增當 check box 被點擊時觸發的 method
    - (3): 新增當畫面中的 button 被點擊時觸發的 method
3. 在 app.component.css 中新增三個 CSS class

    ```css
    .color {
        color: lightskyblue;
    }

    .size {
        font-size: 36px;
    }

    .bgcColor {
        background-color: lightpink;
    }
    ```
    
![img](https://i.imgur.com/6gn4aw4.gif)

在畫面中可以看到，當我們點選一個或多個 check box 時，會依照對應的名稱對 `<div>` 添加或刪除一個或多個 CSS class，當點擊其中一個 check box 先將他的對應的 property 變更狀態，等全部都變更好後，當使用者點擊了 change 的按鈕，會將 [ngClass] 所綁定的 object 進行更新從而更新 `<div>` 中的 CSS class。

還記得為什麼綁定物件時可以更改 CSS class 的綁定嗎？在前幾篇的 class binding 中有提到，可以使用物件綁定，他的要求是 `key 為 class 名稱， value 為布林值（用於控制是否呈現）` 的這種型別的 object，忘記的可以回去複習一下喔！

## Setting inline styles with NgStyle

講完了 NgClass 後接著來介紹 NgStyle 的用法，他與 NgClass 差不多，唯一不同的是他是可以添加或刪除一個或多個 style，既然他們的使用方法差不多，那我們就以上面的例子，將上面的例子從 NgClass 更改為 NgStyle 吧。

1. 首先先將 app.component.html 中顯示的 `<div>` 從 [ngClass] 更改為 [ngStyle] 並將綁定的名稱更改為 currentStyle，其他我們保持不變

    ```html
    <!-- app.component.html -->

    <!-- Change [ngClass] to [ngStyle] -->
    <div [ngStyle]="currentStyle">
      This div element style is initial color, size and bgcColor
    </div>

    <ul>
      <li>
        <label for="colorChange">Change color</label>
        <input type="checkbox" id="colorChange" (change)="onChange('color')" />
      </li>
      <li>
        <label for="fontSizeChange">Font size change</label>
        <input type="checkbox" id="fontSizeChange" (change)="onChange('size')" />
      </li>
      <li>
        <label for="bgcChange">Background color change</label>
        <input type="checkbox" id="bgcChange" (change)="onChange('bgcColor')" />
      </li>
    </ul>

    <!-- Change method name -->
    <button (click)="onSetCurrentStyle()">Change div Style</button>
    ```

2. 接著我們改寫 app.component.ts 中的 property name 與 method

    ```typescript
    import { Component } from '@angular/core';

    @Component({
      selector: 'app-root',
      templateUrl: './app.component.html',
    })
    export class AppComponent {
      color = false;
      size = false;
      bgcColor = false;
      currentStyle: Record<string, string> = {};    // (1)

      onChange(checkboxId: string) {
        switch (checkboxId) {
          case 'color':
            this.color = !this.color;
            break;
          case 'size':
            this.size = !this.size;
            break;
          case 'bgcColor':
            this.bgcColor = !this.bgcColor;
            break;
          default:
            break;
        }
      }

      onSetCurrentStyle() {                        // (2)
        this.currentStyle = {
          'color':  this.color ? 'lightskyblue' : '',
          'font-size': this.size ? '36px' : '',
          'background-color': this.bgcColor? 'lightpink': ''
        }
      }
    }
    ```

    - (1): 將原本的 currentClasses 改名為 currentStyle （因為已經將 `<div>` 重新使用 [ngStyle] 綁定過了）
    - (2): 更改使用者點擊按鈕時處發的 method

![img](https://i.imgur.com/JNULNN0.gif)

在畫面中可以看到雖然將 `<div>` 重新使用 [ngStyle] 綁定過但是結果與使用 [ngClass] 一樣，唯一不同的是在 onSetCurrentStyle 中的操作改了，如果是要綁定 style 的話需要使用`以樣式名稱為 key，以樣式值為 value 的物件` 一樣在前幾天的 style binding 裡面有提到喔！


# Displaying and updating properties with ngModel

介紹完如何透過 [ngClass] 與 [ngStyle] 改變 HTML element 的樣式後，接著介紹如何使用 ngModel

ngModel directive 是用在顯示數據 property 與當用戶對被綁定的 HTML element 進行更改時同步更新 property， 一樣舉個例子吧

1. 首先要使用 ngModel 必須要在 app.module.ts 中引用 `FormsModule` 並將他放在 `import` 當中，可能很多新手會覺得為什麼要放在這邊，沒關係之後再講解 Module 時會專門為大家講解

    ```typescript
    import { NgModule } from '@angular/core';
    import { BrowserModule } from '@angular/platform-browser';

    import { AppRoutingModule } from './app-routing.module';
    import { AppComponent } from './app.component';
    import { FormsModule } from '@angular/forms';

    @NgModule({
      declarations: [
        AppComponent,
      ],
      imports: [
        BrowserModule,
        AppRoutingModule,
        FormsModule
      ],
      providers: [],
      bootstrap: [AppComponent]
    })
    export class AppModule { }
    ```

2. 在 app.component.ts 中新增一個 property 用於綁定畫面中的 `<input>`

    ```typescript
    import { Component } from '@angular/core';

    @Component({
      selector: 'app-root',
      templateUrl: './app.component.html',
      styleUrls: ['./app.component.css']
    })
    export class AppComponent {
      currentItem = '';
    }
    ```

3. 在 app.component.html 中新增一個 `<input>` 並將他綁定

    ```html
    <!-- app.component.html -->

    <label for="example-ngModel">[(ngModel)]: </label>
    <input [(ngModel)]="currentItem" id="example-ngModel">

    <div>{{currentItem}}</div>
    ```
    
![img](https://i.imgur.com/hIek2rn.gif)

在畫面中可以看到當我們在 `<input>` 中輸入數值時，會同步的將 component 中的 property 更改。

可能有人會問：阿這個跟 Two-way binding 有什麼不一樣？這個答案我可以很明確地告訴你，他們是一樣的，所以你也可以把它理解為在同一層中的 Two-way binidng，於是你也可以利用 property binding 與 event binding 達到相同的結果

```html
<input [currentItem]="currentItem" (change)="onChnge($event)" id="example-ngModel">

<div>{{currentItem}}</div>
```

有興趣的可以嘗試看看把他從 ngModel 改寫為 property binding 與 event binding 喔！


# 結論

本章中介紹了什麼是 attribute directive 與該怎使用，可以利用 ngClass 對 HTML 的 element 進行一個或多個的 CSS class 綁定，也可以使用 ngStyle 對 HTML 的 element 進行一個或多個的 Style 綁定，最後介紹的是 ngModel 的綁定，在使用他之前需要在 app.module.ts 的 imports 中引入 `FormsModule` 才可以使用，他的作用可以把它想像成是在同一層中的 Two-way binding，對於表單的控制非常好用，本章中使用了比較多例子，如果在嘗試的時候遇到問題也歡迎在底下留言。

雖然本章的開頭提到 Angular 的預設有 attribute 與 structural directives 兩種，但礙於篇章長度的問題將它分為本篇的 attribute directive 與下一篇的 structural directive。

structural directive 的概念其實我們在前面的例子中多多少少都有用到一點，比如 `*ngFor` 或 `*ngIf` 等等的，主要的目的是用來改變 DOM 的結構與佈局，詳細的內容會在明天介紹，那我們就明天見吧！



# Reference

- [Angular.io - build in directives](https://angular.io/guide/built-in-directives)