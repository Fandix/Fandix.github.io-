---
title: Day15. Attribute directives
date: 2021-09-15 10:08:08
tags:
- Angular
- Front-end

categories:
- 2021 鐵人賽
---

在前幾天中有介紹了 Angular 中內建的一些 attribute directive，但是在實際開發專案時可能會遇到內建 attribute directive 無法處理的問題，這時候就需要建立客製化 attribute directive，本章就會介紹如何建立屬於自己的 attribute directive。

![https://ithelp.ithome.com.tw/upload/images/20210821/201247672vA3mJw784.jpg](https://ithelp.ithome.com.tw/upload/images/20210821/201247672vA3mJw784.jpg)

<!-- more -->

# Building an attribute directive

來跟著 Angular 的官方文檔做一個練習，將會建立一個客製化的 attribute directive，將一個宿主 element 的背景顏色設置為黃色的 directive。

1. 首先先使用 Angular CLI 建立一個 directive

    ```bash
    ng generate directive highlight
    ng g d highlight
    ```

    使用這個 CLI 指令可以自動建立一個名為 highlight.directive.ts 和 highlight.directive.spec.ts 的檔案，並且會在被加入到 app.module.ts 的 `declares` ，所以如果不是用 CLI 建立 directive file 時，要記得手動將它加入到 declares 裡面喔！

2. 在 highlight.directive.ts 中從 `@angular/core` 中導入 `ElementRef`

    ```typescript
    import { Directive, ElementRef } from '@angular/core';

    @Directive({
      selector: '[appHighlight]'
    })
    export class HighlightDirective {
        constructor(el: ElementRef) {                           // (1)
           el.nativeElement.style.backgroundColor = 'yellow';   // (2)
        }
    }
    ```

    - (1): 在 constructor 中將 ElementRef inject 到 HighlightDirective 中
    - (2): 透過使用 ElementRef 的 `nativeElement` property 授與對宿主 DOM element 的直接訪問權限，並將他的 style 改變為黃色
3. 在 app.component.html 中添加一個 element 並將剛剛客製化的 directive 加在上面

    ```html
    <!-- app.component.html -->

    <p appHighlight>Highlight me!</p>
    ```
    
![https://ithelp.ithome.com.tw/upload/images/20210809/20124767f71HrHwdCO.png](https://ithelp.ithome.com.tw/upload/images/20210809/20124767f71HrHwdCO.png)

Angular 會將 HighlightDirective 給實例化並將對 `<p>` element 的 reference injects 到  highlight.directive.ts 的 constructor 中，讓他可以對這個元素的 style 進行更改。



# Handling user events

除了改變 element 之外也可以在客製化的 directive 中檢測用戶在畫面中的事件（鼠標移入或移出）並對事件進行不同的響應。

舉個例子，保持上面的例子只是將畫面上的 `<p>` element 綁定一個 hover 事件，當鼠標移到上面時將他背景顏色改為黃色

1. 從 `@angular/core`中導入 HostListener

    ```typescript
    import { Directive, ElementRef, HostListener } from '@angular/core';
    ```

2. 更改 highlight.directive.ts 中的 method 用於響應事件

    ```typescript
    import { Directive, ElementRef, HostListener } from '@angular/core';

    @Directive({
      selector: '[appHighlight]'
    })
    export class HighlightDirective {
      constructor(private el: ElementRef) {}

      @HostListener('mouseenter') onMouseEnter() {               // (1)
        this.highlight('yellow');
      }
      
      @HostListener('mouseleave') onMouseLeave() {               // (2)
        this.highlight('');
      }
      
      private highlight(color: string) {                         // (3)
        this.el.nativeElement.style.backgroundColor = color;
      }
    }
    ```

    - (1): 當鼠標移動到元素上時觸發此 method 對行為作出響應
    - (2): 當鼠標從元素上離開時觸發此 method 對行為作出響應
    - (3): 對綁定元素的 style 進行更改

![img](https://i.imgur.com/l47DGum.gif)

在畫面中可以看，當我們的鼠標移動到 `<p>`  element 上面時會觸發 highlight.directive.ts 中的 method 將他的背景顏色改為黃色。



# Passing values into an attribute directive

除了將你要更改的內容寫死在 directive 之外（上面例子是寫死 hover 時背景變黃色），你也可以透過傳遞參數的方式，動態的傳遞你所期望改變的內容，一樣拿上面的例子來延伸吧

1. 首先先改變 highlight.directive.ts 的內容

    ```typescript
    import { Directive, ElementRef, HostListener, Input } from '@angular/core';  // (1)

    @Directive({
      selector: '[appHighlight]'
    })
    export class HighlightDirective {
      @Input() appHighlight = '';                              // (2)
      constructor(private el: ElementRef) {}

      @HostListener('mouseenter') onMouseEnter() {
        this.highlight(this.appHighlight);                     // (3)
      }
      
      @HostListener('mouseleave') onMouseLeave() {
        this.highlight('');
      }
      
      private highlight(color: string) {
        this.el.nativeElement.style.backgroundColor = color;
      }
    }
    ```

    - (1): 從 `@angular/core` 中引入 Input
    - (2): 新增一個 property 並使用 @Input() 將他裝飾為從父層傳遞的數據
    - (3): 當鼠標移動到 element 時更改的顏色從寫死的狀態變更為綁定 @Input() 的內容
2. 更改 app.component.html 的內容，將顏色傳遞給 HighlightDirective

    ```html
    <!-- app.component.html -->

    <p [appHighlight]="'red'">Highlight me!</p>
    ```
    
![img](https://i.imgur.com/Fh0oaOI.gif)

可以看到畫面中，當鼠標移動到 element 時從黃色變為我們傳遞給 directive 的紅色了。



# Binding to a second property

除了可以傳遞參數給 directive 之外還可以對他設定一個預設值，以上面例子來說，我們可以設定一個預設值，直到使用者改變顏色後才變為指定的顏色

1. 更改 highlight.directive.ts 中的內容

    ```typescript
    import { Directive, ElementRef, HostListener, Input } from '@angular/core';

    @Directive({
      selector: '[appHighlight]'
    })
    export class HighlightDirective {
      @Input() appHighlight = '';
      @Input() defaultColor = '';                              // (1)
      constructor(private el: ElementRef) {}

      @HostListener('mouseenter') onMouseEnter() {
        this.highlight(this.appHighlight || this.defaultColor || 'red');
      }
      
      @HostListener('mouseleave') onMouseLeave() {
        this.highlight('');
      }
      
      private highlight(color: string) {
        this.el.nativeElement.style.backgroundColor = color;
      }
    }
    ```

2. 在 app.component.ts 中添加一個 property 與 method

    ```typescript
    import { Component } from '@angular/core';

    @Component({
      selector: 'app-root',
      templateUrl: './app.component.html',
      styleUrls: ['./app.component.css']
    })
    export class AppComponent {
      color = '';                            // (1) 
      
      onClick (color: string) {              // (2)
        this.color = color;
      }
    }
    ```

    - (1): 新增一個 property 用於傳給 directive
    - (2): 新增一個 method 當畫面中的按鈕被點擊時觸發
3. 在 app.component.html 中新增三個 `<button>` 並將 directive 加上預設值

    ```html
    <!-- app.component.html -->

    <button (click)="onClick('green')">Green</button>
    <button (click)="onClick('yellow')">Yellow</button>
    <button (click)="onClick('cyan')">Cyan</button>

    <p [appHighlight]="color" defaultColor="violet">Highlight me too!</p>
    ```
    
![img](https://i.imgur.com/b0ZYw8X.gif)

在畫面中可以看到，當我們沒有點擊畫面中的 `<button>` 時我們的鼠標移動到 element 時顯示的顏色是預設的顏色，而當點擊了其中一個按鈕後就換更新為指定的顏色。



# 結論

本章介紹了如何建立與使用客製化的 directive，可以透過使用 ElementRef 的 `nativeElement` property 授與對宿主 DOM element 的直接訪問權限，可以使用 HostListener 用來處理畫面中使用者的行為，也可以使用之前提到的 @Input() 用於傳遞參數進到 directive 中，這個客製化的 directive 對於要處理特別的情況時非常好用。

下一章將介紹如何創建客製化的 structural directive並介紹 directive 使如何工作的、 Angular 如解釋速記以及如何添加 template 的保護 property 用於捕獲 template 的錯誤，那我們就明天見囉！



# Reference

- [Angular.io - attribute directive](https://angular.io/guide/attribute-directives)