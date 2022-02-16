---
title: Day20. Angular Routing
date: 2021-09-20 13:20:46
tags:
- Angular
- Front-end

categories:
- 2021 鐵人賽
---

在現代網頁中常會使用一種稱為 `single page application（SPA）`的技術，可以通過顯示或隱藏特定的 component 來更改用戶看到的畫面，而不是像伺服器重新請求整個頁面的數據對其重新加載，可以使用 `Angular router` 來處理當你需要處理從一個畫面到另一個畫面的任務，Router 通過將瀏覽器的 URL 解析為更改畫面的指令，將使用者導航到指定的畫面。

![https://ithelp.ithome.com.tw/upload/images/20210821/20124767VAxO4x8GEi.jpg](https://ithelp.ithome.com.tw/upload/images/20210821/20124767VAxO4x8GEi.jpg)

<!-- more -->

# What is single page application (SPA) ?

single page application (SPA) 中文翻譯為`單頁面應用程序`，就如同他的名字他是一個只有一個頁面的應用程序，在網頁中可以看成使用者對網頁的任何操作都不會讓網頁整個重新載入，簡單來說他會讓大部分的信息跟畫面保持不變只更新少部分需要變動的資訊或畫面，舉例來說你在使用 gmail 的時候無論你是在收件夾還是在寄件夾，畫面的標題跟左邊的側欄基本上都不會更改，更改的只有中間的那一部分，這就是 SPA 的概念。

SPA 最大的好處在於，他不像過去的網頁當使用者點擊某個任務時需要將整個畫面重新載入，會造成使用者每次的點擊都會花費大量的時間等待畫面重新載入，這嚴重引響了使用者體驗，而 SPA 則不一樣，他通過動態重寫當前頁面的方式做到快速的更換畫面，大大提升使用者的使用體驗，這邊只是簡單介紹一下他的觀念，有需要詳細了解 SPA 與 MPA 的差別與各自的好處，可以看[這一篇](https://asperbrothers.com/blog/spa-vs-mpa/)文章。


# What is router?

在開發一個大型的專案時一定不可能只有一個畫面，隨著功能越來越多越來越複雜，就會需要更多的頁面專門處理他們各自的任務，比如說登錄頁面專門用來處理登陸的任務，購物車頁面專門用來處理購物車的存放與增刪，諸如此類有非常非常多的任務與畫面存在於我們的專案中。

而當在主頁面中點擊了購物車的功能按鈕就會進到購物車的頁面中，就好比主頁面連接著購物車頁面
![https://ithelp.ithome.com.tw/upload/images/20210815/20124767IICbH9PtB5.png](https://ithelp.ithome.com.tw/upload/images/20210815/20124767IICbH9PtB5.png)

而使用者又可以點擊購物車頁面的商品內容按鈕，他又回連接到商品內容頁面，商品內容頁面又可以連接回主頁面或去到登陸頁面，諸如此類的互相關聯著就如同

![https://ithelp.ithome.com.tw/upload/images/20210815/20124767qNfTFTuqDd.png](https://ithelp.ithome.com.tw/upload/images/20210815/20124767qNfTFTuqDd.png)

光看到就頭皮發麻， 隨著專案越來越大越來越複雜，上面這個圖只會越來越密集越來越大，為了應對這種麻煩的情況 Router 出現了。

router 只有一個工作就是`接收使用者傳來要去到哪一個頁面的任務並把畫面導向對的地方`，當使用者點擊購物車按鈕，他會發送一個`使用者需要到購物車頁面`的任務給 Router，而 Router 正確地把畫面引導向購物車頁面，而在購物車頁面中使用者又點擊了商品內容按鈕，他又會發送一個`使用者要到商品內容頁面`的任務給 Router，當它收到任務後會再次精準地把畫面導向商品內容頁面

![https://ithelp.ithome.com.tw/upload/images/20210815/20124767iHe8uy4GSV.png](https://ithelp.ithome.com.tw/upload/images/20210815/20124767iHe8uy4GSV.png)

這樣是不是乾淨整潔多了呢？雖然在開發的過程中沒有一定需要使用 Router 來達到 SPA 的效果，但是可以看到整個程式會變得非常非常複雜，所以當要建立比較大型的網頁時 Router 就是非常重要的功能，當然他不只是接受任務導向畫面這麼簡單，還可以傳遞參數、紀錄資訊等等，詳細的內容就繼續望下看吧。


# Generate an application with routing enabled
了解了什麼是 SPA 與 Router 後接著來看看在 Angular 中該如何實現吧，一樣舉個簡單的例子，當要創建 Angular 專案時可以使用 Angular CLI 自動化建立專案，在建立時有一個選項

![https://ithelp.ithome.com.tw/upload/images/20210815/20124767uEueB4YNCA.png](https://ithelp.ithome.com.tw/upload/images/20210815/20124767uEueB4YNCA.png)

可以選擇在創建專案時直接選擇要加入 Angular routing，那麼專案生成時就會附加一個 `app-routing.module.ts` 檔案，用於建立專案的 router 設定，而如果沒有在一開始生成也沒關係，一樣可以使用 Angular CLI 建立出來

```bash
ng generate module project --routing
```

這邊要提醒一下當你是手動建立 routing module 的話，需要將你建立出來的 RoutingModule 放入 app.module.ts 中 @NgModule 的 metadata 的 `imports` 裏面，這樣才可以正常使用喔

```typescript
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';

import { AppRoutingModule } from './app-routing.module';
import { AppComponent } from './app.component';

@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    BrowserModule,
    AppRoutingModule,        // remember to settings routing module in here
  ],
  providers: [],
  bootstrap: [AppComponent]   
})
export class AppModule { }
```

## Adding components for routing

1. 建立完 routing.module 後接著來建立兩個 component 讓畫面可以切換這兩個 component 的 view

    ```bash
    ng generate component first
    ng generate component second
    ```

2. 接著將這兩個 component 引入到 app-routing.module.ts 中

    ```typescript
    import { NgModule } from '@angular/core';
    import { RouterModule, Routes } from '@angular/router';

    import { FirstComponent } from './sub-component/first/first.component';
    import { SecondComponent } from './sub-component/second/second.component'

    const routes: Routes = [];

    @NgModule({
      imports: [RouterModule.forRoot(routes)],
      exports: [RouterModule]
    })
    export class AppRoutingModule { }
    ```

3. 在 routes 陣列中設定要導向的路線

    ```typescript
    const routes: Routes = [
      { path: 'first-component', component: FirstComponent },
      { path: 'second-component', component: SecondComponent },
    ];
    ```

    在 routes 陣列中每個 route 都是一組包含兩個 property 的 javascript object，第一個 property 定義了 route 的 `url 路徑`，第二個 property 定義了 Angular 看到這個 url 後應該`顯示哪一個 component`。

4. 將 route 加到專案中

    ```html
    <!-- app.component.html -->

    <h1>Angular Router App</h1>

    <nav>
      <ul>
        <li>
          <a routerLink="/first-component" routerLinkActive="active">First Component</a>
        </li>
        <li>
          <a routerLink="/second-component" routerLinkActive="active">Second Component</a>
        </li>
      </ul>
    </nav>

    <router-outlet></router-outlet>
    ```

    1. 首先建立兩個 `<a>` 元件，代表在畫面中可以點擊兩個連結分別切換兩個 component 的 view
    2. 在 `<a>` 的 attribute 中使用 `routerLink` 指定 url 的路徑
    3. 在 `<a>` 的 attrubute 中加上 `routerLinkActive`，他是用來處理當現在的網址與所設定的連結一致時要加上去的Class名稱，簡單來說當 router 為當下那個 link 時，就會將在該link 加上對應的 css class
    4. 最後在下方加上 `<router-outlet>` 用來顯示切換的 component，等於說要被切換的 component 會出現在這邊

![img](https://i.imgur.com/Nh7xpWU.gif)

在畫面中可以看到，當點擊不同的連接的時候會先更改瀏覽器的 url ，接著會隨著 url 的切換也改變畫面顯示的 component，這就是 router 的功用。



# 結論

本章中介紹了什麼是 SPA 以及他的優點，而在開發 SPA 時會隨著頁面越來越多而形成越來越複雜的互相連接的關係，所以就需要 `Router` 的出現，他可以讓我們開發專案時減少複雜的頁面連接，專心開發 component 與畫面就好，導向的工作就由 Router 來完成。

講解了最基本的 Router 的用法，下一篇中將會介紹 Router 在 Angular 中的更多用法，那我們就明天見吧！



# Reference

- [Single Page Application (SPA) vs Multi Page Application (MPA) – Two Development Approaches](https://asperbrothers.com/blog/spa-vs-mpa/)
- [How does the Internet work?](https://developer.mozilla.org/en-US/docs/Learn/Common_questions/How_does_the_Internet_work)
- [Angular.io - Angular Routing](https://angular.io/guide/routing-overview)
- [Angular.io - Common Routing Tasks](https://angular.io/guide/router)