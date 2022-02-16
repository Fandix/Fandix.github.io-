---
title: Day24. Template-driven forms
date: 2021-09-24 08:52:28
tags:
- Angular
- Front-end

categories:
- 2021 鐵人賽
---

在上一篇中提到了 Angular 中的兩種不同的 Form，介紹了他們在使用上以及細節上的不同，接著在本篇中將會著重介紹 Template-driven forms，那就繼續看下去吧。

![https://ithelp.ithome.com.tw/upload/images/20210823/20124767lIolI8gw9s.png](https://ithelp.ithome.com.tw/upload/images/20210823/20124767lIolI8gw9s.png)

<!-- more -->

# What is Template-driven forms?

顧名思義 Template-driven forms 就是一個透過 template 驅動的表單，或者可以說他是基於原生 HTML 所產生出來的表單，在 template 中使用 directive 和 attribute 來為指定的輸入元件進行綁定與驗證，所有的動作都會在 template 中完成，所以 component 只需要很少的設定，這點是和 reactive forms 最大的不同，而 Template-driven forms 具有以下的設置：

- Form 是使用 `ngForm` directive 所設置的
- 使用 `ngModel` directive 設置控制元件
- ngModel 提供了雙向綁定，將 template 的輸入元素與 component 的 property 做綁定
- 在 template 中利用 directive 驗證輸入內容

所以對於 template-driven forms 的優點在於：

- 在 component 中有較少的設置
- 相較於 reactive forms 來說設置更簡單

但他的缺點是：

- 難以動態添加表單控制元件
- 單元測試較為困難


# Building a template-driven form

在介紹完 template-driven form 後，接著直接使用一個例子來講解該如何使用 template form 吧，我們的目的再於創建一個 template-driven form，其 template 中的輸入元素綁定到 component 的數據 property，並建立輸入驗證以維護數據的完整性，在這次的例子中我會添加一點樣式讓畫面不會太醜 (![/images/emoticon/emoticon07.gif](/images/emoticon/emoticon07.gif)

# Import FormsModule

首先要做的就是在 app.module.ts 中從 `@angular/form` 中將 `FormsModule` 引入到 app.module 中 metadata 的 `imports` 中。

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
    FormsModule,
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { } 
```


# Build the basic form

接著來建立一個基本的 form 吧，在建立 form 之前先把要 form 的數據模型給定義出來，所以新增一個 class 用於制定數據模型

```typescript
// hero.ts

export class Hero {
  id!: number;
  name!: string;
  power!: string;
  alterEgo?: string;
  constructor(id: number, name: string, power: string, alterEgo?: string) {
    this.id = id;
    this.name = name;
    this.power = power;
    this.alterEgo = alterEgo;
  }
}
```

建立了名為 `Hero` 的 class，這樣就可以利用 `new` 將參數傳入後產生一個固定資料結構的 object，接著使用 Angular CLI 建立一個 component 用於處理 form 的邏輯與顯示

```bash
ng generate component hero-form
```

建立完 component 後，接著在 hero-form.component.ts 中定義一些 form 的細節

```typescript
import { Component, OnInit } from '@angular/core';
import { Hero } from '../hero';                                                // (1)

@Component({
  selector: 'app-hero-form',
  templateUrl: './hero-form.component.html',
  styleUrls: ['./hero-form.component.css'],
})
export class HeroFormComponent implements OnInit {
  powers = ['Really Smart', 'Super Flexible', 'Super Hot', 'Weather Changer']; // (2)
  model = new Hero(18, 'Dr IQ', this.powers[0], 'Chuck Overstreet');           // (3)
  submitted = false;

  constructor() {}

  ngOnInit(): void {}

  onSubmit() {                                                                 // (4)
    this.submitted = true;
  }
}
```

- (1): 引入剛剛寫的 Hero class
- (2): 建立一個 property 將它指定為 array 並將所以有超能力填入其中
- (3): 使用 new 將剛剛寫的 Hero 實例化為一個 object 並賦予給 property model
- (4): 新增一個 method 用於當使用者點擊 submit 按鈕時觸發

定義完 app-hero.component.ts 後，接著將他的 selector 加入到 app.component.html 中

```html
<!-- app.component.html -->

<app-hero-form></app-hero-form>
```

現在在畫面生應該是空空如也才對，不過這非常正常因為還沒撰寫 hero-form.component 的 view，現在就來把他補上吧，希望在畫面中呈現兩個帶有 `<label>` 的 `<input>` 還要有一個 `<select>` 最後要有一個 `<button>` 用來點擊 submit。

```html
<!-- hero-form.component.html -->

<div class="form-group">
  <label for="name">Name</label>
  <input type="text" class="form-control" id="name" required/>

  <label for="alterEgo">Alter Ego</label>
  <input type="text" class="form-control" id="alterEgo" required/>

  <label for="power">Hero Power</label>
  <select class="form-control" id="power" required>
    <option *ngFor="let pow of powers" [value]="pow">{{ pow }}</option>
  </select>

  <button type="submit" class="btn btn-success">Submit</button>
</div>
```

![https://ithelp.ithome.com.tw/upload/images/20210823/20124767MZcnMB02me.png](https://ithelp.ithome.com.tw/upload/images/20210823/20124767MZcnMB02me.png)

在畫面中會看到這樣的畫面，因為我有加一些 CSS 的樣式，所以這邊可以發揮你的 CSS 功力看是要做的跟我一樣還是做一個屬於你自己的，不過主要還是要介紹 template form 所以就不多做介紹。


# Bind input controls to data properties

在完成基本的 form 畫面後，下一步要使用 `雙向綁定 ( Two-way binding )` 將 template 中的輸入元素綁定到 component 中對應的 property，以便將使用者輸入的值更新綁定的 property，也讓使用程式更改的 property 的值可以呈現在畫面中。

FormsModule 中聲明的 `ngModel` directive 可以讓 template-driven form 中的控制元件綁定到數據模型中的 property，當使用 Two-way binding 綁定 `[(ngModel)]` 後， Angualr 就可以跟蹤控制元件的值和使用者交互，這可以讓畫面與表單模型保持同步。

1. 首先先更改 hero-form.component.html 中的內容

    ```html
    <!-- hero-form.component.html -->

    <div class="form-group">
      <label for="name">Name</label>
      <input
        type="text"
        class="form-control"
        id="name"
        [(ngModel)]="model.name"
        name="name"
    		required
    		#name="ngModel"
      />

      <label for="alterEgo">Alter Ego</label>
      <input
        type="text"
        class="form-control"
        id="alterEgo"
        [(ngModel)]="model.alterEgo"
        name="alterEgo"
    		required
    		#alterEgo="ngModel"
      />

      <label for="power">Hero Power</label>
      <select class="form-control" id="power" [(ngModel)]="model.power" name="power">
        <option *ngFor="let pow of powers" [value]="pow">{{ pow }}</option>
      </select>

      <button type="submit" class="btn btn-success">Submit</button>
    </div>
    ```

    在 Name 和 Alter Ego 兩個 `<label>` 下面的 `<input>` 利用 `[(ngModel)]` 綁定 component 中的 property，這邊要注意的是當你使用了 `[(ngModel)]` 綁定 property 後，需要定義他的 `name` 不然會報錯喔！

2. 綁定完每一個輸入元素後接著來對整個 Form 進行綁定，在導入 `FormsModule` 時 Angular 會自動創建一個 `NgForm` directive 並將其附加到 template 中的 `<form>` 上面（因為 NgForm 具有匹配 `<form>` 的 select ），所以要訪問 NgForm 和整個表單狀態，需要聲明一個 template 引用變量。

    ```html
    <!-- hero-form.component.html -->

    {{ model | json }}

    <form #heroForm="ngForm">
      <div class="form-group">
        <label for="name">Name</label>
        <input
          type="text"
          class="form-control"
          id="name"
          [(ngModel)]="model.name"
          name="name"
    			required
    			#name="ngModel"
        />

        <label for="alterEgo">Alter Ego</label>
        <input
          type="text"
          class="form-control"
          id="alterEgo"
          [(ngModel)]="model.alterEgo"
          name="alterEgo"
    			required
    			name="alterEgo"
        />

        <label for="power">Hero Power</label>
        <select
          class="form-control"
          id="power"
          [(ngModel)]="model.power"
          name="power"
        >
          <option *ngFor="let pow of powers" [value]="pow">{{ pow }}</option>
        </select>

        <button type="submit" class="btn btn-success">Submit</button>
      </div>
    </form>
    ```

    使用 `<form>` 將之前寫的表單包起來並使用 template variable 設定 `#heroForm`，而 heroForm 這個變量現在是對控制整個表單的 `NgForm` directive 實例的 reference。

    而在整個 form 上面添加了 `{{ model | json }}` 用於觀看 component property 的變化，可以在畫面中的輸入框更改名稱或選擇其他的 power 來看看 component 中的 property 會不會跟著改變。
    

# Track control states

接著要來介紹 NgModel directive 的跟蹤控制元件的狀態，他會告訴你使用者是否觸碰了控制元件、值是否被更改了或是輸入的值是否無效，Angular 在控制元件上設置了特殊的 CSS Class 來反映他的狀況，如下表所示

| State | Class is true | Class if false |
|----------|:-------------:|:-------------:|
| 控制元件是否被訪問 | ng-touched | ng-untouched
| 控制元件的值是否被更改 | ng-dirty | ng-pristine
| 控制元件的值是否有效 | ng-valid | ng-invalid

此外 Angular 再提交時將 `ng-submitted` CSS Class 應該要用於 `<form>`，所以不放在上面一起介紹。

## Observe control states

要查看 Angular 如何添加和刪除 CSS Class，可以打開瀏覽器的開發人員工具並檢查英雄姓名的 `<input>`。

1. 在 Name 的輸入框中填入新的值，可以看到 `<input>` 綁定的 CSS Class 發生更改
2. 在 `<input>` 中執行以下操作會更改成不同的 CSS Class
    1. 完全不去點擊和更改 `<input>` 的話，代表他是`未受影響的`、`原始`且`有效的`。
    2. 點擊 `<input>` 後在點擊外部（不更改內容），現在已經訪問了控制元件，所以 CSS Class 從 `ng-untouched` 變為 `ng-touched`。
    3. 在 `<input>` 的內容加入一個斜槓（ \ ），他會變成 `ng-touched` 和 `ng-dirty`。
    4. 完全移除 `<input>` 的內容這會使這個控制元件的值變為無效，因此會從 `ng-valid` 變為 `ng-invalid`。

## Create visual feedback for states

可以利用 `ng-valid` 和 `ng-invalid` 來處理當使用者填入非有效內容時會發生什麼事，當輸入無效時可以在輸入框下方顯示警告的畫面，也可以在警告的畫面中填入提醒或範例，可以在 Name 的後面加上一個 `<div>` 並利用 `[hidded]` 來控制是否顯示僅告訊息

```html
<label for="name">Name</label>
<input
  type="text"
  class="form-control"
  id="name"
  [(ngModel)]="model.name"
  name="name"
  required
  #name="ngModel"
/>
<div [hidden]="name.valid || name.pristine" class="alert alert-danger">
  Name is required
</div>
```

當 name 的值是 `valid` 和 `pristine` 的時候會將這個警告區域隱藏，而當輸入值為 `invalid` 實則會顯示

![img](https://i.imgur.com/NNEaREt.gif)


# Submit the form with ngSubmit

在使用者填寫完表單後應該要有一個功能是提交使用者所寫的內容，以上面的例子來說就是下方的 `submit` 按鈕，但是目前還沒對他進行任何處理所以點了也沒反應，接著要來對這個按鈕進行更改

1. 首先在 `<form>` 中添加一個 event binding，將 `(ngSubmit)` 綁定上去

    ```html
    <form (ngSubmit)="onSubmit()" #heroForm="ngForm">
    ```

2. 接著使用 template variable `#heroForm` 來當作 submit 按鈕是否可以被點擊（是否所有內容都 valid），並將他的 type 改為 `submit`

    ```html
    <button type="submit" class="btn btn-success" [disabled]="!heroForm.form.valid">Submit</button>
    ```

3. 在 hero-form.component.ts 中更改 onSubmit method

    ```typescript
    import { Component, ViewChild } from '@angular/core';                // (1) 
    import { NgForm } from '@angular/forms'; 
    import { Hero } from '../hero';

    @Component({
      selector: 'app-hero-form',
      templateUrl: './hero-form.component.html',
      styleUrls: ['./hero-form.component.css'],
    })
    export class HeroFormComponent {
      @ViewChild('heroForm', { static: true }) heroForm!: NgForm;        // (2)
      powers = ['Really Smart', 'Super Flexible', 'Super Hot', 'Weather Changer'];
      model = new Hero(18, 'Dr IQ', this.powers[0], 'Chuck Overstreet');
      submitted = false;

      constructor() {}

      onSubmit() {
        console.log(this.heroForm.value);                                // (3)
        this.submitted = true;
      }
    }
    ```

    - (1): 從 `@angular/core` 中引入 `ViewChild`
    - (2) 利用 @ViewChild 獲得訪問 template 中的 `heroForm`
    - (3) 當使用者按下 submit 按鈕時顯示目前 Form 中所有欄位的內容（真實情況可以將這一組數據做別的處理）

![img](https://i.imgur.com/R602aCy.gif)


# 結論

本章中介紹了如何建立一個 template-driven form，可以對數據進行修改、驗證等等，使用 `[(NgModel)]` 雙向綁定 component 中的 property，使用 `ngModel` 中的 `valid` 來判斷使用者輸入的內容是否符合規定，至於 submit 按鈕的 event binding 並不像之前的例子一樣綁定在 `<button>` 上，而是要將 `(ngSumbit)` 綁定在 `<form>` 上，這樣才可以獲得整個 form 的內容，而在 component 中要獲得表單的內容需要使用 `@ViewChild` 綁定 `<form>` 上的 template variable。

下一篇將要介紹 Angualr 中的另一種 form，`Reactive forms` 他相較於 template form 來說會複雜一點，但是比較有彈性且比較可測試性，對於大型的 form 來說是非常好用且方便的，詳細的內容就明天再講解吧，那麼明天見吧。


# Reference

- [Angular.io - Building a template-driven form](https://angular.io/guide/forms#submit-the-form-with-ngsubmit)
