---
title: Day23. Introduction to forms in Angular
date: 2021-09-23 10:15:51
tags:
- Angular
- Front-end

categories:
- 2021 鐵人賽
---

從本章開始會進入 Angular Form 的部分，在現代網頁中與使用者互動的過程變得越來越重要，其中最主要的便是 Form，從一開始的登陸頁面到對商品或對網頁中元素的設定都需要使用到 Form 來取得使用者輸入的資料，也需要對使用者輸入的資料做驗證以防使用者輸入了非格式的數據導致出錯。

Angular 提供了兩種不同的方法來處理使用者的輸入，分別是 `reactive` 與 `template-driven`，兩種方法都是從 view 中捕獲使用者輸入事件、驗證用戶輸入、創建表單模型或更新數據模型，本篇會大概介紹這兩種方法以及他們之間的差別，那就繼續往下看吧！

![https://ithelp.ithome.com.tw/upload/images/20210821/20124767cYDUalx1rV.png](https://ithelp.ithome.com.tw/upload/images/20210821/20124767cYDUalx1rV.png)

<!-- more -->

# Choosing an approach

`Reactive forms` 和 `template-driven forms` 以不同的方式處理和管理表單數據，他們各有各自的優勢：

- **Reactive forms**：提供對底層表單 object 的最直接且明顯的訪問，具有更高的可擴展性、可重用性和可測試性。
- **Template-driven forms**：依賴 template 中的 directive 來創建或操作底層表單 object，常用於添加`簡單`的表單，他很容易被添加到專案中但他的擴展性與可測試性比較差。

## Key differences

可以透過一張表單來總結 `Reactive forms` 和 `template-driven forms` 之間的主要區別

|    |      Description      | Reactive |
|----------|:-------------:|:-------------:|
| Setup of form model | 明顯的，在 Component 中創建 | 隱式的，由 directive 創建
| Data model | 結構化和不可變 | 非結構化且可變
| Data flow | 同步 | 非同步
| Form validation | 透過 function | 透過 directive


## Scalability

如果 Form 是你專案中的核心部分的話，那麼就需要將整個 Form 模型重複使用於各個不同的 Component，所以可伸縮性就非常重要。

Reactive forms 比 template-driven forms `更具有可擴展性` ，他提供了對底層表單 API 的直接訪問，並在 view 和數據模型之間使用`同步`的數據流，這讓你創建大型的表單變得更加容易，他也需要比較少的測試設置，並且測試不需要深入了解就可以正確的測試表單的更新和驗證。

相較於 Reactive forms 而言 Template-driven forms 比較常用在 `簡單且不可重複使用的場景`，他對底層表單 API 的訪問比較抽象，並在 view 與數據模型之間使用`非同步`的數據流，由於他的對 API 的訪問較為抽象所以對於測試來說比較不好撰寫，非常需要依賴手動更改檢測執行才能成功，需要非常多的設置。


# Setting up the form model

`Reactive forms` 和 `template-driven forms` 都會追蹤使用者與畫面中的表單和 Component 中表單數據之間的變化，他們在底層來說是共享同一個地層構建模塊，但在創建和管理表單控制實例的方法不同。

## Common form foundation classes

Reactive forms 與 template-dirven forms 都建立在以下的 base classes 之上：

- **FormControl**：會追蹤`單個`表單控制元件的值和驗證狀態，簡單來說他會負責追蹤畫面中一個綁定的 `<input>` 元素的內容。
- **FormGroup**：會追蹤`多個`表單控制元件的值和狀態，簡單來說他像是 javeascript 的 object，裡面包含了很多個不同的表單控制元件，可能由多個 FormControl 或 FormArray 所組成，甚至裡面在包含一個 FormGroup。
- **FormArray**： 會追蹤表單控制陣列中的值和狀態，簡單來說可以想像是 FormControl 的陣列。
- **ControlValueAccessor**：在 Angular FormControl instnce 與 DOM 之間建立了一個溝通的橋樑。

## Setup in reactive forms

使用 reactive forms 可以直接在 component 中定義你的表單模型，`[formControl]`  directive 會使用 `internal value accessor` 將創建的 FormControl intance 鏈結到 view 中被綁定的單個輸入元素，來舉個例子吧

1. 首先先在 app.module.ts 中引入 Form module

    ```typescript
    import { NgModule } from '@angular/core';
    import { BrowserModule } from '@angular/platform-browser';

    import { AppRoutingModule } from './app-routing.module';
    import { AppComponent } from './app.component';
    import { FormsModule, ReactiveFormsModule } from '@angular/forms';

    @NgModule({
      declarations: [
        AppComponent,
      ],
      imports: [
        BrowserModule,
        AppRoutingModule,
        FormsModule,
        ReactiveFormsModule
      ],
      providers: [],
      bootstrap: [AppComponent]
    })
    export class AppModule { }
    ```

2. 在 app.component.ts 中建立 FormControl

    ```typescript
    import { Component } from '@angular/core';
    import { FormControl } from '@angular/forms';                 // (1)

    @Component({
      selector: 'app-root',
      templateUrl: './app.component.html',
    })
    export class AppComponent {
      favoriteColorControl = new FormControl('');                 // (2)
    }
    ```

    - (1): 從 `@angular/forms` 中引入 FormControl
    - (2): 將 FormControl 實例化並賦予給 component 的 property
3. 在 app.component.html 中綁定表單控制元件

    ```html
    <!-- app.component.html -->

    <div>
      Favorite Color: <input type="text" [formControl]="favoriteColorControl" />
    </div>

![https://ithelp.ithome.com.tw/upload/images/20210821/20124767foRImuOGGA.png](https://ithelp.ithome.com.tw/upload/images/20210821/20124767foRImuOGGA.png)

![img](https://i.imgur.com/TOrJdYH.gif)

將 `<input>` 元件與 formControl 綁定後，他會追蹤畫面中 `<input>` 內容的變化並將它傳遞給 component 中被綁定的 property ( favoriteColorControl )，其實跟著上面的範例是無法在 console 中獲得輸入的值，要怎麼獲得我之後會詳細講解。

## Setup in template-driven forms

接著來看如何建立 template-dirven forms 的表單控制模型，在 template-dirven forms 中的控制模型是隱性的，使用 directive `MgModel` 為給定的表單元素創建和管理一個 formControl 實例，一樣舉個例子

1. 在 app.component.ts 中新增一個 property

    ```typescript
    import { Component } from '@angular/core';

    @Component({
      selector: 'app-template-favorite-color',
      templateUrl: './app.component.html',
    })
    export class FavoriteColorComponent {
      favoriteColor = '';
    }
    ```

2. 在 app.component.html 中使用 `NgModel` 綁定剛剛建立的 property

    ```html
    <!-- app.component.html -->

    <div>
      Favorite Color: <input type="text" [(ngModel)]="favoriteColor" />
    </div>
    ```
    
![https://ithelp.ithome.com.tw/upload/images/20210821/20124767k4yN0zPmBx.png](https://ithelp.ithome.com.tw/upload/images/20210821/20124767k4yN0zPmBx.png)

![img](https://i.imgur.com/k4EhckL.gif)

在畫面中可以看到透過 `NgModel` 將指定的 property 綁定，當畫面中的表單元素內容發生變化時會傳遞給 component 中被綁定的 property，一樣上面的範例是無法看到 console 的內容之後會講解。


# Data flow in forms

介紹完如何綁定表單的控制元件後，接著要來看看 Form 的數據流，當你的專案中有使用表單時，Angular 必須保持 view 與 component 模型內容的同步，當使用者更改畫面中的值時這個新的值需要立即的反映在數據模型中，相同的當數據模型的值更新時也需要立馬反映在畫面中，而兩種不同的 form 有著不同的處理數據與更改畫面的方式。

## Data flow in reactive forms

在 reactive forms 中 view 的每一個表單元素都直接鏈結到 component 中的表單模型（formControl instance），從 view 到表單模型以及表單模型到 view 的更新是`同步`的，不依賴於 UI 的呈現方式。

view 到表單模型顯示了當在畫面中輸入字串後從 view 通過以下步驟流向表單模型：

1. 使用者在 `<input>` 中輸入了一個值，例子中是輸入了喜歡的顏色（blue）。
2. 表單輸入元素（`<input>`）發送了帶有最新值（blue） 的`輸入事件`。
3. `control value accessor` 會監聽輸入元素的事件，當接收到事件後會立即將最新的值中繼到 FormControl 實例上
4. FormControl 實例會通過 `valueChanges` 這個 `observable` 發出最新的值（ valueChange.subscribe(val => { ... }) ）。
5. valueChanges observable 的任何訂閱者都會收到這個最新的值。

![https://ithelp.ithome.com.tw/upload/images/20210821/2012476752sXiPKza9.png](https://ithelp.ithome.com.tw/upload/images/20210821/2012476752sXiPKza9.png)

而表單模型到 view 顯示了對表單模型進行編成更改後如何通過以下步驟更新 view 的值：

1. 調用 `setValue()` method 用於更新 FormControl 的內容。
2. FormControl 實例通過 valueChanges observable 發出新的值。
3. valueChanges observable 的任何訂閱者都會收到新的值。
4. `control value accessor` 使用這個最新的值更新畫面表單元素

![https://ithelp.ithome.com.tw/upload/images/20210821/20124767KNa7RJO17u.png](https://ithelp.ithome.com.tw/upload/images/20210821/20124767KNa7RJO17u.png)

## Data flow in template-driven forms

在 template-driven forms 中每個表單元素都鏈結到一個內部管理表單模型的 directive，從 view 到模型顯示了當在畫面中輸入字串後從 view 通過以下步驟流向表單模型：

1. 使用者在畫面中的 `<input>` 中輸入新的值（blue）。
2. `<input>` 元素發出一個帶有 blue 的 `input 事件`。
3.  附加在 `<input>` 的 `control value accessor` 會觸發 FormControl 實例上的 setValue() method。
4. FormControl 實例通過 `valueChanges observable` 發出新的值。
5. valueChanges observable 的任何訂閱者都會收到新的值。
6. control value accessor 還會調用 `NgModel.viewToModelUpdater()`，這個 method 會發出一個 `ngModelChange` 的事件。
7. 因為 component 的 template 對 `favoriteColor` property 使用雙向屬性綁定 (Two-way binding) ，所以 component 中的 favoriteColor property 被更新為 `ngModelChange` 事件發出的值 （blue）。

![https://ithelp.ithome.com.tw/upload/images/20210821/20124767cOKhVGuSFk.png](https://ithelp.ithome.com.tw/upload/images/20210821/20124767cOKhVGuSFk.png)

表單模型到 view 顯示了當 favoriteColor 從藍色變為紅色時，數據會如何從表單模型流向 view，通果以下步驟：

1. 在 component 中更新 property favoriteColor 的值。
2. 變更檢測開始。
3. 在變更檢測期間，Lifecycle method `ngOnChanges` 在 `NgModel` directive 實例上被調用，因為他的 input binding 發生更改。
4. ngOnChanges() 透過`非同步`的方式設置 FormControl 實例的值
5. 變更檢測完成。
6. FormControl 實例通過 valueChanges observable 發出最新的值。
7. valueChanges observable 的任何訂閱者都會收到新的值。
8. `control value accessor` 會使用最新的 favoriteColor 值更新 view 中的表單輸入元素

![https://ithelp.ithome.com.tw/upload/images/20210821/20124767ZCtQxTcxVF.png](https://ithelp.ithome.com.tw/upload/images/20210821/20124767ZCtQxTcxVF.png)

## Mutability of the data model

更改和追蹤的 method 在兩種不同的 form 會有不同的作用：

- **Reactive forms**：通過將表單數據模型作為`不可更改`的數據結構用來保持表單數據模型的純淨，每次在表單數據模型上觸發更改時，FormControl 實例都會返回一個`新的`表單數據模型而不是更改現有的模型，這使你可以通過 FormControl 的 `Observable` 追蹤表單數據模型的更改，讓更改檢測更有效，因為他只要追蹤 FormControl 的更改，由於數據的更新遵循 reactive 的模式，所以可以使用 observable 的操作符改變表單數據。
- **Template-driven forms**：依靠`雙向數據綁定`的可變性在 template 中進行更改後也更改 component 中的 property，由於在使用雙向數據綁定時沒有辦法只跟蹤表單數據模型，所以效率較低。

以上面例子而言兩種不同的 form 對於數據的變更處理：

- 對於 reactive forms 來說，當 FormControl 的值更新時，會返回一個新的值。
- 對於 template-driven forms 來說當更新值時會將上一個內容取代為新的值。


# 結論

本篇中介紹了 Angular 的兩種不同的 Form，分別是 `Reactive forms` 與 `template-driven forms`，雖然兩種方法都可以做到與使用者的互動，但是還是有些的不同，`Reactive forms` 有更高的可擴展性、可重用性和可測試性，而 `template-driven forms ` 則是使用雙向綁定的方式將 component 的 property 與 template 的輸入元件綁定，所以常用在比較簡單的 Form 結構上。

雖然在本篇中簡單的介紹了該如何使用 `Reactive forms` 與 `template-driven forms`，不過只有稍微提到而已，接下來會分別對他們兩個做比較詳細的介紹與如何使用，那就明天見吧！


# Reference

- [Angualr.io - Introduction to forms in Angular](https://angular.io/guide/forms-overview)