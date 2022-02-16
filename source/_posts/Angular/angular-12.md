---
title: Day12. Template variables
date: 2021-09-12 14:07:10
tags:
- Angular
- Front-end

categories:
- 2021 鐵人賽
---

Template variable 可以讓你在 template 的任意一處使用被標記過的 HTML 元件的數據，例如響應使用者的輸入或微調應用程序的表單，簡單來說當你在畫面中有一個 `<input>`，除了透過 Form 獲得使用者在這個 `<input>` 所以輸入的數據之外，也可以透過將這個 `<input>` 設定為 template variable，這樣就可以讓在別的地方的 `<button>` 中的 event binding 獲得這個 `<input>` 的數據，詳細的內容讓我們繼續看下去吧。

![https://ithelp.ithome.com.tw/upload/images/20210821/20124767lMmC7T1IVB.png](https://ithelp.ithome.com.tw/upload/images/20210821/20124767lMmC7T1IVB.png)

<!-- more -->

# Syntax

要將 template 中的元件聲明為 template variable，需要在這個元件上加上`哈希符號 #`，舉個例子

```html
<input #phone placeholder="phone number" />

<!-- lots of other elements -->

<!-- phone refers to the input element; pass its `value` to an event handler -->
<button (click)="callPhone(phone.value)">Call</button>
```

雖然在 template 中 `<button>` 離 `<input>` 很遠，但因為`<input>` 透過 # 被標記為 template variable，所以遠處的 `<button>` 可以透過呼叫 phone.value 獲取這個 `<input>` 的值。


# How Angular assigns values to template variables

了解了 template variable 後，接著要來介紹 Angular 是如何根據你聲明變量的位置為 template variable 分配一個值：

- 如果在 component 的 selector 上聲明變量，則是指這個 component 的 instance
- 如果在標準 HTML element 上聲明變量，則是指這個 HTML 的 element
- 如果在 `<ng-template>` 上聲明變量，則該變量引用一個 `TemplateRef` 的 instance，它代表著這個 template，這個之後會詳細介紹
- 如果在變量右邊指定了一個名稱，例如 `#var="ngModel"，則該變量引用匹配 ngModel 名稱 (exportAs name) 的元素上的 directive 或 component

## Using NgForm with template variables

在大多數情況下，Angular 將 template varibale 的值設置為他出現的元素，比如上面的例子，phone 是指 `<input>`，而點擊了按鈕則會將 `<input>` 的內容傳遞給 component 中的 callPhone() method。

不過也可以透過在變量右邊指定一個名稱達到其他的效果，比如說可以使用 NgForm directive 來達到 Form 的效果，他通過引用 directive 的 exportAs name 來獲取對不同值得引用，來舉個例子吧

1. 在 app.component.ts 中加入 property 與 method

    ```typescript
    import { Component } from '@angular/core';

    @Component({
      selector: 'app-root',
      templateUrl: './app.component.html',
    })
    export class AppComponent {
      submitMessage: string = '';                 // (1)

      onSubmit($event: any) {                     // (2)
        this.submitMessage = $event.form.value.name;
      }
    }
    ```

    - (1): 新增 property 用於在畫面中顯示 `<input>` 的值
    - (2): 新增一個 method 用於當使用者點擊按鈕時觸發
2. 在 app.component.html 中添加 `<form>`

    ```html
    <!-- app.component.html -->

    <form #itemForm="ngForm" (ngSubmit)="onSubmit(itemForm)">
      <label for="name"
        >Name <input class="form-control" name="name" ngModel required />
      </label>
      <button type="submit">Submit</button>
    </form>

    <div [hidden]="!itemForm.form.valid">
      <p>{{ submitMessage }}</p>
    </div>
    ```
    
![img](https://i.imgur.com/ZhvCzuM.gif)

如果沒有在 #itemForm 右邊使用 `ngForm` 則 #itemForm 的引用值將是 `HTMLFormElement` 的 `<form>` 元件，但這邊是使用了 `ngForm` 所以 #itemForm 是對 NgForm directive 的引用，他可以跟蹤表單中的每一個可控制元件的值和有效性，這與原生的 `<form>` 元素不同，NgForm directive 中有一個 property，他用於檢測整個表單的有效性，所以當 itemForm.form.valid 無效時，則整個 NgForm 會將 submit 按鈕 disable。


# Template variable scope

既然提到 variable 就不免俗的要提到 scope，至於什麼是 scope？簡單來說 `scope 就是一個變數的生存範圍，一但超出了這個範圍就無法存取到這個變數` 至於詳細的內容可以看我這一篇 [[JS] You Don't Know JavaScript [Scope & Closures] - What is Scope?](https://ithelp.ithome.com.tw/articles/10249231) 文章中有詳細的介紹什麼是 scope。

而 template variable 則可以在 template 中的任何一個位置中調用到，但是 `Structural directive` （*ngIf, *ngFor, `<ng-template>` ...） 他會充當 template 的邊界，所以你會無法訪問到 Structural directive 內部的 template variable。

## Accessing in a nested template

就如同 Javascript 的 scope 一樣，template 中內部的 template 可以訪問外部 template 的變量，但相反的話不行，舉個在同層級的例子

```html
<input #ref1 type="text" [(ngModel)]="firstExample" />
<span *ngIf="true">Value: {{ ref1.value }}</span> 
```

 當更改了 `<input>` 的內容時會立即更改 `<span>` 中的內容，因為 Angular 會立即通過 template variable ref1 來更新內容，接著再舉一個由內部訪問到外部變量的例子

```html
<input #ref1 type="text" [(ngModel)]="firstExample" />

<!-- New template -->
<ng-template [ngIf]="true">
  <!-- Because the context is inherited, the value is available to the new template -->
  <span>Value: {{ ref1.value }}</span>
</ng-template>
```

像上面提到的，`<ng-template>` 會創造一個新的 template 範圍，但是因為在這個新的 template 範圍中的 `<span>` 是從內部訪問外部的變量 ref1，所以是可以正常訪問到的，就跟 Javascript  一樣

```javascript
const ref1 = 'input value';

function spanValue() {
	console.log(ref1); // input value
}
```

雖然 ref1 與 function spanValue 是不同的 scope，但因為內部可以訪問外部變量，所以可以將 ref1 給 console 出來，接著來看父層如果訪問子層變量會發生什麼事

```html
<ng-template [ngIf]="true">
  <!-- The reference is defined within a template -->
  <input #ref2 type="text" [(ngModel)]="secondExample" />
</ng-template>
<!-- ref2 accessed from outside that template doesn't work -->
<span>Value: {{ ref2?.value }}</span>
```

如果是上面例子的情況，`<span>` 會無法獲得 ref2 的內容，因為對於 ref2 來說他是存在於子層的變量，所以無法透過父層呼叫到，就跟 Javascript 一樣

```javascript
function inputScope() {
    let ref2 = 'in child scope';
}

console.log(ref2);  // Uncaught ReferenceError: ref2 is not defined
```

外部無法呼叫到內部 scope 的變量，所以在使用 template variable 時要記得存取變量的規則， `外部 scope 無法存取到內部 scope 的變量`。


# 結論

本章中介紹了什麼是 template variable 與他的使用方法，簡單來說就是可以利用他獲得其他 element 的內容，不過因為他也是屬於變量所以也會有 scope 的問題，要記住外部 scope 是無法訪問到內部 scope 的變量的，而使用了 `Structural directive` 則會創造一個獨立的 template 讓整個 template 出現父子層的現象，就跟在 javescript 中使用 function 建立 function scope  一樣，所以要特別注意。

而本篇也是講解 template 的最後一篇，明天開始將會進入到 directive，這個觀念在前面多多少少都有提到一點，但沒關係之後會詳細地對他進行講解，那我們就明天見吧。

---

# Reference

- [Angular.io](https://angular.io/guide/template-reference-variables)