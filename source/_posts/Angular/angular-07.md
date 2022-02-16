---
title: Day7. Content projection
date: 2021-09-07 10:17:12
tags:
- Angular
- Front-end

categories:
- 2021 鐵人賽
---

本章節將要介紹如何使用 ng-content 將一個 Component 的內容投影到另一個 Component 中，創建靈活且可被重複使用的 Component，來滿足程式設計中 DRY (Don't repeat yourself) 的觀念。

在 Angular 中 Content 是一種呈現的模式，可以在其中插入或投影另一個 Component 的內容，簡單來說，當我們在寫 HTML 時常常會有這樣的結構：
```html
<div>
	<p> Hello World!</p>
</div>
```
在 Angular 的概念中可以把它想像成，有一個 `<p>` tag 被投影到 `<div>` 之中，雖然不是原理可能不這樣子，但是類似的模式。

在 Angular 中有幾種常見的 ng-content 例子：

- **Signle-slot content projection**: 投影來自單一 Source 的 Component 內容
- **Multi-slot content projection**: 一個 Component 接收來自多個 Source 的 Component 內容

![https://ithelp.ithome.com.tw/upload/images/20210821/20124767Bs01NpTA05.jpg](https://ithelp.ithome.com.tw/upload/images/20210821/20124767Bs01NpTA05.jpg)

<!--more-->

# Single-slot content projection
在 Angular 的 ng-content 中最基本的就是 single-slot content projection，他是指`將一個 Component 的內容投影在另一個 Component 之中`，我們做一個簡單的小例子來解釋這個行為。

1. 首先我們先使用 Angular CLI 建立一個 Component

    ```bash
    ng generate component zippy-basic
    ng g c zippy-basic
    ```
2. 在 zippy-basic.component.html 中添加 ng-content 到你希望投影內容出現的位置

    ```html
    <!-- zippy basic.component.html -->
    <h2>Single-slot content projection</h2>
    <div>
        ng-content content: 
        <ng-content></ng-content>
    </div>
    ```

3. 接著我們在 app.component.html 中使用 zippy-basic 的 selector

    ```html
    <!-- app.component.html -->
    <app-zippy-basic></app-zippy-basic>
    ```
![https://ithelp.ithome.com.tw/upload/images/20210801/20124767DnHr1yIlhY.png](https://ithelp.ithome.com.tw/upload/images/20210801/20124767DnHr1yIlhY.png)

在畫面中我們看到，我們使用 ng-content 的位置什麼都沒有，是出了什麼 bug 嗎？其實不是!，是因為我們還沒決定該把什麼內容投影在 <ng-content> 的位置上，我們來更改一下 app.component.html 的內容

```html
<!-- app.component.html -->
<app-zippy-basic>
    <p>From app.component.html projection content to zippy-basic component</p>
</app-zippy-basic>
```

![https://ithelp.ithome.com.tw/upload/images/20210801/20124767EQg91a8KNp.png](https://ithelp.ithome.com.tw/upload/images/20210801/20124767EQg91a8KNp.png)

當更改玩 app.component.html 的內容後，可以在畫面中看到，Angular 把我們夾在 `<app-zippy-basic>` 的內容放到 `<ng-content>` 的位置了，
![https://ithelp.ithome.com.tw/upload/images/20210801/20124767bUsgQrsaND.png](https://ithelp.ithome.com.tw/upload/images/20210801/20124767bUsgQrsaND.png)

當然可能有人會問了，那...我直接把 `<p>` 放在 zippy-basic.component.html 不就好了嗎？

```html
<!-- zippy basic.component.html -->
<h2>Single-slot content projection</h2>
<div>
    ng-content content: 
    <p>From app.component.html projection content to zippy-basic component</p>
</div>
```

沒有錯，但是其實 ng-content 通常不是讓你放 html tag 的，我們來舉個例子，想像一下一個場景，當你在開發專案時，有沒有遇到在一個地方需要放置三種不同類型的 button ? 如果是這樣的話你是不是會這麼做

```html
<div>
	<!-- style1 button -->
	<label>This is style 1 button</label>
	<p>Hello world</p>
	<button>button</button>
<div>
<div>
	<!-- style2 button -->
	<label> This is style 2 button </label>
	<p>Hello world</p>
	<button>button</button>
<div>
<div>
	<!-- style3 button -->
	<label> This is style 3 button </label>
	<p>Hello world</p>
	<button>button</button>
<div>
```

有沒有發現當使用上面這種 HTML 結構雖然可以達到我們要的目的，但是 `<label>`、`<p>`、`<button>` 一直在重複出現但內容卻是一樣的，就違背了 DRY 的原則了，這時候我們就可以使用 ng-content 先將模板做好，再把不同 style 的 `<button>` 投影進去就好，來舉個例子吧：

1. 首先我們先建立一個模板 Component

    ```bash
    ng generate component content-template
    ng g c content-template
    ```
2. 接著我們在 content-template.component.html 中把我們的模板寫好

    ```html
    <!-- content-template.component.html -->

    <h1>Content projection number {{styleCount}}</h1>
    <label>This is style {{styleCount}} button</label>
    <p>Hello world</p>
    <ng-content></ng-content>
    ```

    我們在原本放置不同 style 的地方使用 <ng-content> 變成將別的內容投影到這個位置上，對了還記得昨天提到的 @Input( ) 嗎？ 這邊我們來複習一下，`使用 ng-content 一樣可以由父層傳遞數據到子層喔`。

    ```typescript
    import { Component, Input } from '@angular/core';

    @Component({
      selector: 'app-content-template',
      templateUrl: './content-template.component.html',
    })
    export class ContentTemplateComponent {
      @Input() styleCount: string = '';
      constructor() { }
    }
    ```
3. 接著我們來更改一下 app.component.html 的內容，讓不同 style 的 button 投影到 content-template 吧

    ```html
    <!-- app.component.html -->
    <app-content-template [styleCount]="'1'">
        <button class="style1">Click Me</button>
    </app-content-template>

    <app-content-template [styleCount]="'2'">
        <button class="style2">Click Me</button>
    </app-content-template>

    <app-content-template [styleCount]="'3'">
        <button class="style3">Click Me</button>
    </app-content-template>
    ```
![https://ithelp.ithome.com.tw/upload/images/20210801/20124767W7aNxciNY5.png](https://ithelp.ithome.com.tw/upload/images/20210801/20124767W7aNxciNY5.png)

這樣就完成了我們的目的，簡單的幾行就可以達到相同的目的，透過`先建立模板再將不同樣式或不同的 Component 投影進這個模板 Component 之中`，就可以讓這個模板 Component 在各個地方都被使用並且達到減少重複程式碼的目的。


# Multi-slot content projection
第二種其實與第一種非常相似，只不過變成了一個 Component 中投影了多個不同的內容，不過值得注意的是，由於是多個不同的內容投影在一個 Component 之中，所以就會有順序位置問題，於是 Angular 提供了 `select` 屬性讓你加在 ng-content 上，讓你可以`將某一個內容放在指定的位置上`，一樣舉個例子吧。

1. 首先一樣創建一個新的 Component

    ```bash
    ng generate component zippy-basic
    ng g c zippy-basic
    ```
2. 接著我們在 zippy-basic.component.html 中添加 ng-content 到想要的位置

    ```html
    <!-- zippy-basic.component.html -->

    <h2>Multi-slot content projection</h2>
    Default:
    <ng-content></ng-content>
    Question:
    <ng-content></ng-content>
    ```
3. 接著我們把想要的位置添加一個 select 屬性，讓投影的位置固定在我們想要的地方

    ```html
    <!-- zippy-basic.component.html -->

    <h2>Multi-slot content projection</h2>
    Default:
    <ng-content></ng-content>
    Question:
    <ng-content select="[question]"></ng-content>
    ```
4. 最後我們在 app.component.html  中將我們像要投影的內容放進去，記得！指定位置的內容需要加上 select 的內容喔

    ```html
    <!-- app.component.html -->

    <app-zippy-basic>
      <p question>Is content projection cool?</p>
      <p>Let's learn about content projection!</p>
    </app-zippy-basic>
    ```
![https://ithelp.ithome.com.tw/upload/images/20210801/20124767IFrm7OvyYX.png](https://ithelp.ithome.com.tw/upload/images/20210801/20124767IFrm7OvyYX.png)

可以看到我們在 app.component.html 中 <p> 的順序與 UI 呈現的順序是顛倒的，就因為我們指定了有 question 這個 select  放置的地方。

與 Single-slot content projection 一樣，Multi-slot content projection 的目的不在於放入原生 HTML 的 Tag，目的與上一個一樣，都是可以先建立一個模板之後將不同的內容投影到這個模板中，只不過多了`可以透過 select 讓你選擇放置的位置`與讓你`可以放多個投影`，這邊就不在做一次練習了，有興趣的話可以自己挑戰一下利用 Multi-slot content projection 完成與上面的例子一樣的內容。



# 結論

在本章中介紹了什麼是 content projection 以及該如何使用它，在官方文檔中其實除了 Single-slot content projection 與 Multi-slot content projection 之外其實還有第三個常見的投影 `Conditional content projection`，但是因為他牽扯的概念跟本篇章使用 ng-content 比較不一樣，他是透過使用 `ng-template` 來做到這個功能，所以就先不將它納入本章的範圍，這個方法會在之後介紹到。

下一篇將會分享 template 是什麼，他在 Angular 中是 HTML 的角色，而可以在 template 中使用許多語法來達到靈活建立畫面的功能，那我們就下一篇再見吧。


# Referece
- [Angular.io - content-projection](https://angular.io/guide/content-projection)
