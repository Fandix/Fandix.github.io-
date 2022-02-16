---
title: Day9. Transforming Data Using Pipes
date: 2021-09-09 10:17:12
tags:
- Angular
- Front-end

categories:
- 2021 鐵人賽
---

在上一章中介紹了如何在 template 中插入 component 的變量，而本章節要介紹如何使用 angular 的 pipes 來轉換插入的值（字串、貨幣金額、日期或其他數據）， pipes 是在模板表達式中的簡單函數，用於接受輸入值並返迴轉換後的值，而 Anngular 提供了幾個預設的 pipes 提供使用：

- **DatePipe**：根據區域設置規則格式化日期值
- **UpperCasePipe**：將內容都變更為大寫
- **LowerCasePipe**：將內容都變更為小寫
- **CurrencyPipe**：將數字轉換為貨幣字串，根據區域設置規則進行格式化。
- **DecimalPipe**：將數字轉換為帶小數點的字串，並根據區域設置規則進行格式化。
- **PercentPipe**：將數字轉換為百分比字串，根據區域設置規則進行格式化。

![https://ithelp.ithome.com.tw/upload/images/20210821/20124767iCR0nyUNiR.png](https://ithelp.ithome.com.tw/upload/images/20210821/20124767iCR0nyUNiR.png)

<!-- more -->

# Using a pipe in a template
要在 template 中使用 pipes 功能請使用管道運算符 ( | )，一樣舉個例子吧：

1. 在 app.component.ts 中新增一個變數，賦予它 javescript 的 Date 型別資料

    ```typescript
    import { Component } from '@angular/core';

    @Component({
      selector: 'app-root',
      templateUrl: './app.component.html'
    })
    export class AppComponent {
      birthday = new Date(1995, 9, 25);
    }
    ```

2. 在 app.component.html 中將 birthday 插入並將其中一個使用 pipes 改變他的型態

    ```html
    <!-- app.component.html -->

    <p>My birthday is {{birthday}}</p>
    <p>My birthday is {{birthday | date }}</p>
    ```
    
![https://ithelp.ithome.com.tw/upload/images/20210803/20124767GlSkfWn7Na.png](https://ithelp.ithome.com.tw/upload/images/20210803/20124767GlSkfWn7Na.png)

在畫面中可以看到，沒有使用 pipes 轉換過的數值就是 javascript Date 型態的內容，而使用了 pipes 轉換過的數值看起來就好看多了，而這就是 pipes 的用法。



# Transforming data with parameters and chained pipes

在使用 pipes 改變呈現內容時，可以輸入可選的參數來微調 pipes 輸出的結果，比如說可以將國家單位（EUR）當作參數傳遞給 CurrencyPipe，將轉換過的貨幣單位以歐元顯示 `{{ amount | currency:'EUR' }}`，如果要對一個 pipes 使用多個參數時請使用冒號分隔這些參數，比如 `{{ amount | currency:'EUR':'Euros '}}`，也可以使用任何一個有效的模板表達式作為參數。

有些 pipes 需要至少一個參數才可以使用，比如說 SlicePipe ，`{{ slice:1:5 }}` 會創建一個新陣列或字串，其中包含從  element-1 開始到 element-5 結束的元素子集。

```typescript
import { Component } from '@angular/core';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html'
})
export class AppComponent {
  sayHello = 'Hello world'
}
```

```html
<!-- app.component.html -->

<h2>{{ sayHello | slice:1:5 }}</h2>
```

![https://ithelp.ithome.com.tw/upload/images/20210803/20124767cAsa3R4sC4.png](https://ithelp.ithome.com.tw/upload/images/20210803/20124767cAsa3R4sC4.png)

畫面中原本要呈現的 `Hello world`，變成了 ello ，是因為透過 slice 選擇了 index[1] ~ index[5] 的資料。



# Creating pipes for custom data transformations

在開頭時提到了 Angular 有提供幾個預設的 pipes 可以使用，但這些不夠應付我們可能面面臨到的問題，這時候客製化 pipes 就很重要了，客製化的 pipes 也和預設的一樣，接收一個輸入將他轉換過後輸出，那麼就來看看該如何使用客製化 pipes 吧。

## Marking a class as a pipe

要建立一個客製化的 pipes，首先需要向建立 component 一樣先建立一個 typescript 的 class，但是不同的是，當我們創建 Component Class 時使用的裝飾器是 `@Component`，代表這個 Class 是屬於 Component 的，但是要建立 pipis class 則需要使用 `@Pipe` 這個裝飾器，而對這個 Class 的命名請使用 `駝峰命名法`，不要在名稱中間使用連字符號( - )，舉個例子吧，可以建立一個 pipes 接收一個數值作為輸入與一個參數，將輸入的數值做參數的次方，比如輸入 = 2 參數 = 10 那麼就會等於 2^10 = 1024。

1. 使用 Angular CLI 建立一個 pipes class

    ```bash
    ng generate pipe exponential-strength
    ```

2. 在 exponential-strength.pipe.ts 中添加轉換 method

    ```typescript
    import { Pipe, PipeTransform } from '@angular/core';

    @Pipe({name: 'exponentialStrength'})
    export class ExponentialStrengthPipe implements PipeTransform {
      transform(value: number, exponent: number = 1): number {
        return Math.pow(value, exponent);
      }
    }
    ```

3. 如果你是使用 Angular CLI 他會自動將這個 pipes class 放到 app.module.ts 的 declarations 中，如果你是手動建立的話要記得將它放到 app.module.ts 的 declarations 裡面喔！

    ```typescript
    import { NgModule } from '@angular/core';
    import { AppComponent } from './app.component';
    import { ExponentialStrengthPipe } from './exponential-strengh.pipe';

    @NgModule({
      declarations: [
        AppComponent,
        ExponentialStrengthPipe,
      ]
    })
    export class AppModule { }
    ```
4. 在 app.component.html 中使用客製化的 pipe 轉換資料

    ```html
    <!-- app.component.html -->

    <h2>Power boost: {{ 2 | exponentialStrength:10 }}</h2>
    ```
    
![https://ithelp.ithome.com.tw/upload/images/20210803/20124767RueRFUir7N.png](https://ithelp.ithome.com.tw/upload/images/20210803/20124767RueRFUir7N.png)



# Detecting changes with data binding in pipes

還記得昨天提到的 Text interpolation 的特性嗎？他可以隨著 class 的 property 變化而動態的顯示，而這個動態變化也可以套用到 pipe 中，所以當客製化的 pipe 輸入是 `string` 或 `number` 時且發生改變時，會動態的作為輸入進到 pipe 中進行轉換，但如果是 `Date` 或 `Array` 類型時，Angular 會檢測到 reference 發生改變時才會觸發執行 pipe 的轉換，舉個例子吧

1. 在 app.component.ts 中定義兩個 property，一個代表要被轉換的值（輸入）另一個代表輸入要做幾次方

    ```typescript
    import { Component } from '@angular/core';

    @Component({
      selector: 'app-root',
      templateUrl: './app.component.html',
      styles: ['input {margin: .5rem 0;}']
    })
    export class AppComponent {
      power = 2;
      factor = 1;
    }
    ```

2. 在 app.component.module.ts 的 imports 中加入 `FormsModule` ，這個是因為要在這個例子中使用到 form，所以要加入這個（之後會詳細的介紹 form）

    ```typescript
    import { NgModule } from '@angular/core';
    import { BrowserModule } from '@angular/platform-browser';

    import { AppRoutingModule } from './app-routing.module';
    import { AppComponent } from './app.component';
    import { FormsModule } from '@angular/forms';
    import { ExponentialStrengthPipe } from './exponential-strengh.pipe';

    @NgModule({
      declarations: [
        AppComponent,
        ExponentialStrengthPipe,
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

3. 在 app.component.html 中使用 pipe

    ```html
    <!-- app.component.html -->

    <h2>Power Boost Calculator</h2>
    <label for="power-input">Normal power: </label>
    <input id="power-input" type="text" [(ngModel)]="power" />
    <label for="boost-input">Boost factor: </label>
    <input id="boost-input" type="text" [(ngModel)]="factor" />
    <p>Super Hero Power: {{ power | exponentialStrength: factor }}</p>
    ```

    這邊可能會有疑問，`[(ngModel)]` 這個是什麼？這邊大概介紹一下，如果將 component 中的 property 使用 `[(ngModel)]` 綁定，代表當 user 在畫面中的 `<input>` 中改變數值時，他會同步改變到 Component 中的 property，當然如果你改 Component 中的 property 的數值一樣會同步更改到畫面 `<input>` 中的值，這稱為 `雙向綁定`，之後會詳細講解，這邊先有大概的概念就好。
    
![img](https://i.imgur.com/Y92boUn.gif)

在畫面中可以看到，當我們每次更改 factor 的值時，pipe 都會自動重新計算結果。



# Detecting pure changes to primitives and object references

在上面提到 `如果是 `Date` 或 `Array` 類型時，Angular 會檢測到 reference 發生改變時才會觸發執行 pipe 的轉換` 這邊要來詳細的說明一下。

在默認情況下 pipe 要被定義成 pure 的，以便 Angular 只有在檢測到輸入值變化時才執行 pipe ，所以必須要是沒有 side effect 的 pure function，而如果是將複合對象當作輸入送進 pipe （現有的 arr 添加新元素）時，因為檢查他的 reference 比深入到 arr 中遞迴的檢查每個元素快得多，所以 Angular 會通過檢查他的 reference 來判定是否發生改變，所以當你將一個 arr 作為輸入送進一個 pipe 時，可能會發生意料之外的錯誤，下面來句個例子：

對了！如果是新手對 javascript 的 object reference 不熟悉的話，建議先去了解一下為什麼對 arr 使用 push 時不算對 reference 的改變，這也是為什麼使用 const 宣告 arr 卻可以對他新增內容的原因

，我在 [ES6 學習筆記_01(let & const)](https://ithelp.ithome.com.tw/articles/10231666)  中提到 const 的本質是什麼，有興趣可以去看一下。

1. 首先一樣先創建一個 pipe 

    ```typescript
    import { Pipe, PipeTransform } from '@angular/core';

    @Pipe({name: 'flyingHeros'})
    export class FlyingHerosPipe implements PipeTransform {
      transform(allHeroes: any) {
        return allHeroes.filter((hero: any) => hero.caFly);
      }
    }
    ```

2. 在 app.component.ts 中定義 Hero list

    ```typescript
    import { Component } from '@angular/core';

    @Component({
      selector: 'app-root',
      templateUrl: './app.component.html',
    })
    export class AppComponent {
      Hero = [
        { id: 11, name: 'Dr Nice', canFly: true },
        { id: 12, name: 'Narco', canFly: false },
        { id: 13, name: 'Bombasto',  canFly: false },
        { id: 14, name: 'Celeritas',  canFly: true },
        { id: 15, name: 'Magneta',  canFly: false },
        { id: 16, name: 'RubberMan',  canFly: true },
        { id: 17, name: 'Dynama',  canFly: true },
        { id: 18, name: 'Dr IQ',  canFly: true },
        { id: 19, name: 'Magma',  canFly: false },
        { id: 20, name: 'Tornado',  canFly: true },
      ];
    }
    ```

3. 在 app.component.html 中使用 pipe 將可以飛的英雄顯示出來

    ```html
    <!-- app.component.html -->

    <div *ngFor="let hero of ( heros | flyingHeros )">
        {{hero.name}}
    </div>
    ```

![https://ithelp.ithome.com.tw/upload/images/20210803/201247677rVVugO8sm.png](https://ithelp.ithome.com.tw/upload/images/20210803/201247677rVVugO8sm.png)

在畫面中可以看到只有會飛的英雄顯示出來，這時我們在 heros 中使用 push 新增英雄。

```typescript
onAddHero() {
  this.heros.push({ id: 1, name: 'Fandix', canFly: true });
}
```

當我將新的英雄 push 近 heros 中，卻發現畫面沒有更改，是壞掉了嗎？ 讓我們將 heros console 出來確認是否真的有將他 push 進去。

```typescript
onAddHero() {
  this.heros.push({ id: 1, name: 'Fandix', canFly: true });
	console.log(this.heros);
}
```

![https://ithelp.ithome.com.tw/upload/images/20210803/20124767DNrdyLkdv3.png](https://ithelp.ithome.com.tw/upload/images/20210803/20124767DNrdyLkdv3.png)

在 console 中可以看到我們確實有將新英雄 push 進 heros 中，這就是剛剛提到的 Angular 在面對 arr 時，只有在他的 reference 發生改變時才會觸發 pipe，有興趣的可以自己嘗試一下，把 `onAddHero()` 變成

```typescript
onAddHero() {
  this.heros = [
      { id: 11, name: 'Dr Nice', canFly: true },
      { id: 12, name: 'Narco', canFly: false },
      { id: 13, name: 'Bombasto', canFly: false },
      { id: 14, name: 'Celeritas', canFly: true },
      { id: 15, name: 'Magneta', canFly: false },
      { id: 16, name: 'RubberMan', canFly: true },
      { id: 17, name: 'Dynama', canFly: true },
      { id: 18, name: 'Dr IQ', canFly: true },
      { id: 19, name: 'Magma', canFly: false },
      { id: 20, name: 'Tornado', canFly: true },
      { id: 1, name: 'Fandix', canFly: true },
    ];
}
```

會發生什麼事。

所以在使用者種複合型的資料時，需要特別注意`只有當輸入的 Reference 發生改變時，才會觸發 pipe `。



# Detecting impure changes within composite objects

那你可能想說，我不要啊我天生反骨我就是想要使用 push 就可以觸發 pipe 可不可以？

當然可以，Angular 提供了當複合數據內發生改變時也可以觸發 pipe 的方法，首先你`需要將這個 pipe 變得 impure`，這樣才可以檢測到 impure 的變化，所以 Angular 只要檢測到每次按鍵或滑鼠的變化時都會觸發一個 impure 的 pipe，要做的很簡單，只要在 pipe.ts 中增加一個屬性就可以了

```typescript
@Pipe({name: 'flyingHeros', pure: false})
```

一樣有興趣的可以拿上面的例子直接將他變成 impure 試試看，這邊就不再做一次了。



# Pipes and precedence

介紹了這麼多的 pipe 用法，可能有人會問：既然 pipe 是透過 ｜ 加在數值後面的，那如果這個數值在進行其他的 Javascript expressions 怎麼辦？

其實在 Angular 中 pipe 運算符的優先級是高於三元運算子（: ?），這意味著如果你有一個 Text interpolation 長這樣 `{{ a ? b : c | x }}` ，那麼他會被解析成 `{{ a ? b : (c | x) }}` 而這個結果可能不是你所希望的，如果你希望達到 `{{ a ? b : c | x }}` 這個結果，請使用括號將前面的三元運算子括起來 `{{ (a ? b : c) | x }}`



# 結論

本篇章中介紹了什麼是 pipe、該怎麼使用它以及客製化自己的 pipe，了解 pipe 對於開發專案是有幫助的，也要特別注意在預設情況下 pipe 是 pure 的，當傳入的輸入是複合型資料時，只有在他的 reference 發生改變時才會觸發，如果想要避免這個問題可以將 pipe 更改為 impure 就可以了。

在官方文檔中還有介紹 `observable` 與 `HTTP` 的 pipe，但是因為牽扯到太多其他的技巧不太符合我們新手入門的領域，所以不在這邊介紹，不過了解了pipe 的基本原理之後，當遇到類似問題再去看就會比較容易看懂，所以就允許我偷懶一下吧

下一篇會介紹 template 的 property binding，它的作用是可以讓你設置 HTML Tag 或 directive 的屬性質，詳細的介紹就期待明天吧。



# Reference

- [Angular.io - pipes](https://angular.io/guide/pipes)
