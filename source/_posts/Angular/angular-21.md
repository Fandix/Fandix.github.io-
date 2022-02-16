---
title: Day21. Common Routing Tasks (一)
date: 2021-09-21 21:43:10
tags:
- Angular
- Front-end

categories:
- 2021 鐵人賽
---

上一張介紹完什麼是 router 與他的基本用法之後，接著要來繼續介紹 Angular router 的其他一些比較詳細的細節與用法，那就接著看下去吧！

![https://ithelp.ithome.com.tw/upload/images/20210819/20124767pIY0V2tSzX.png](https://ithelp.ithome.com.tw/upload/images/20210819/20124767pIY0V2tSzX.png)

<!-- more -->

# What is URL?

URL 是瀏覽器用來檢索網路上任何已發布資源的機制，他的全名叫做 `Uniform Resource Locator`，理論每個有效的 URL 都會指向一個`唯一`的資源，URL 是由不同部分所組成的，一部分是一定要有而另一部分則是可選的，他的組成為下圖

![https://ithelp.ithome.com.tw/upload/images/20210819/20124767g2R4tJ6Uli.png](https://ithelp.ithome.com.tw/upload/images/20210819/20124767g2R4tJ6Uli.png)

## Scheme
![https://ithelp.ithome.com.tw/upload/images/20210819/20124767bxYRnYkWD2.png](https://ithelp.ithome.com.tw/upload/images/20210819/20124767bxYRnYkWD2.png)

URL 的第一部分是 scheme 他表示瀏覽器請求資源時所使用的通訊協議，對於網站而言通常是使用 `http` 或 `https`，除了網站常用的這兩個之外他也知道該如何處理其他方案，比如說填入 mailto 就打開郵件功能等等。

## Authority
![https://ithelp.ithome.com.tw/upload/images/20210819/20124767a13Y5g4IxU.png](https://ithelp.ithome.com.tw/upload/images/20210819/20124767a13Y5g4IxU.png)

接下來是 Authority 他通過字串 `://` 與 Scheme 分開，Authority 部份包括 `Domain Name` 與 `Port` 兩個部分組成，兩者由`冒號（：）`分隔

- domain 表示正在請求的 web 伺服器，通常會是一個 domain name 也可以使用 IP Adress
- Port 代表訪問 Web 伺服器上資源的 `gate`，使用 HTTP 為 80 而 HTTPS 為 443，不過如果是使用 HTTP 協議的標準來授與對其資源的訪問權限時通常會省略他

## Path to resource
![https://ithelp.ithome.com.tw/upload/images/20210819/20124767rr4ZuNbS50.png](https://ithelp.ithome.com.tw/upload/images/20210819/20124767rr4ZuNbS50.png)

上面例子中的 path to resource 是 Web 伺服器上資源的路徑，在早期這樣的路徑代表 Web 伺服器上的物理文件路徑。

## Parameters
![https://ithelp.ithome.com.tw/upload/images/20210819/20124767c0mygWppvq.png](https://ithelp.ithome.com.tw/upload/images/20210819/20124767c0mygWppvq.png)

parameters 是用來提供 Web 伺服器的額外參數，這些參數是由 & 與 / 分隔，Web 伺服器可以透過這些參數再回傳資源前做額外的處理。

## Anchor
![https://ithelp.ithome.com.tw/upload/images/20210819/20124767YuHmZFJj9n.png](https://ithelp.ithome.com.tw/upload/images/20210819/20124767YuHmZFJj9n.png)

Anchor 代表資源內的一種`書籤`，為瀏覽器提供顯示位置於該書籤的位置，例如在 HTML 中新增帶有 id 的標題，當點擊某個標題時可以將瀏覽器瞬間移動到那個標籤那邊，要注意的是 # 後面的部分`永遠不會`隨 request 一起發送到伺服器中。


# Getting route information

雖然了解了如何使用基本的 router 但似乎對於開發專案來說還是不太夠，可能會遇到再開發專案時需要將信息從一個 component 傳遞到另一個 component ( 這邊不是向父子層傳遞參數，是 component 的平行傳遞 )，比如說一個顯示購物清單的應用程序，每個商品項目都有一個唯一的 `id`，要編輯某個商品的內容時使用者要點及編輯按鈕，打開 EditGroceryItem component，希望將 component 的商品 id 傳入以便向使用者顯示整卻的信息。

遇到這個情況就可以使用 router 將信息傳遞給你需要的 component，為此可以使用 `ActivatedRoute interface` 該怎麼使用就來舉個例子吧，一樣拿上一篇的 router 例子來改寫就好

1. 首先先更改 app-routing.module.ts 中的設定

    ```typescript
    import { NgModule } from '@angular/core';
    import { RouterModule, Routes, ActivatedRoute, ParamMap } from '@angular/router';
    import { FirstComponent } from './first/first.component';
    import { SecondComponent } from './second/second.component';

    const routes: Routes = [
      { path: 'first-component/:name', component: FirstComponent },
      { path: 'second-component/:name', component: SecondComponent }
    ];

    @NgModule({
      imports: [RouterModule.forRoot(routes)],
      exports: [RouterModule]
    })
    export class AppRoutingModule { }
    ```

    在 routes 的陣列中設定每個 component 對應到的路徑，這邊將路經更改為 `componentName/:name` ，經過上面對 URL 的介紹就會知道我們希望在 path 中添加 parameters。

2. 在 app.component.html 中調整點擊按鈕後送出的 url path

    ```html
    <!-- app.component.html -->

    <h1>Angular Router App</h1>
    <nav>
      <ul>
        <li><a routerLink="/first-component/first-one-component" routerLinkActive="active">First Component</a></li>
        <li><a routerLink="/second-component/second-one-component" routerLinkActive="active">Second Component</a></li>
      </ul>
    </nav>
    <router-outlet></router-outlet>
    ```

3. 接著在這兩個 component 中新增方法來獲得 url 傳遞的參數

    ```typescript
    import { Component, OnInit } from '@angular/core';
    import { ActivatedRoute } from '@angular/router';                 // (1)

    @Component({
      selector: 'app-first',
      templateUrl: './first.component.html',
    })
    export class FirstComponent implements OnInit {
      name!: string | null;                                           // (2)
      constructor( private route: ActivatedRoute,) { }                // (3)

      ngOnInit(): void {
        this.name = this.route.snapshot.paramMap.get('name')          // (4)
      }
    }
    ```

    ```typescript
    import { Component, OnInit } from '@angular/core';
    import { ActivatedRoute } from '@angular/router';

    @Component({
      selector: 'app-second',
      templateUrl: './second.component.html',
    })
    export class SecondComponent implements OnInit {
      name!: string | null;
      constructor( private route: ActivatedRoute,) { }

      ngOnInit(): void {
        this.name = this.route.snapshot.paramMap.get('name')
      }
    }
    ```

    - (1): 從 `@angular/router` 中引入 `ActivatedRoute`
    - (2): 建立一個 property 用於接收 url 的參數並顯示在畫面中
    - (3): 將 ActivatedRoute 注入到 component 中
    - (4): 獲得 url 的參數
4. 接著在兩個 component 的 template 中顯示獲得的 name

    ```html
    <h1>Component Name is: {{name}}</h1>
    ```
    
![img](https://i.imgur.com/zzGkGs3.gif)

在畫面中可以看到隨著我們點擊不同的超連結會進到不同的 component 中，而畫面中的內容會隨著 url 的參數改變，就代表 component 確實有獲得 url 的參數並將它顯示在各自的 view 中。


# Setting up wildcard routes

再開發這種有使用 router 的專案時，需要注意要能夠正常的處理當使用者輸入了專案中不存在路徑時會發生什麼事，要處理這種情況需要將 `wildcard route` 添加到你的設定中，每當請求的 url 與任何一個路徑不匹配時 Angular 就會選擇這個路徑

```typescript
{ path: '**', component:  }
```

兩個星號（**） 向 Angular 表明這個 route 是 `wildcard route`，對於 component property 則可以使用任一 component 不過通常會多做一個 component 用來處理這種狀況，比較常見的是建立一個 `PageNotFoundComponent` 來向使用者顯示 `404` 或重新導向首頁之類的，記得要將 `wildcard route` 放在最後一條，因為他會匹配任何一個 url。

來用上面的例子新增一個 PageNotFoundComponent 吧

```typescript
const routes: Routes = [
  { path: 'first-component/:name', component: FirstComponent },
  { path: 'second-component/:name', component: SecondComponent },
  { path: '**', component: PageNotFoundComponent  }
];
```

接著在 pageNotFound.component.html 中新增顯示沒有此頁面

```html
<h1>Page Not Found</h1>
```

![img](https://i.imgur.com/GqY1qDl.gif)
可以看到當輸入了一個不符合任何 route 的路徑時會導向 pageNotFoundComponent。


# Nesting routes

隨著專案變大單純的一層 route 已經不夠應付複雜的專案架構，這時就需要 nesting routes 的出現，他可以讓你 `<router-outlet>` 中顯示的 component 中也有自己的 `<router-outlet>`，因為他是對 AppComponent 中的 `<router-outlet>` 的補充，以上面的例子來說除了 app.component 中有 `<router-outlet>` 用來顯示 firstComponent 或 secondComponent 之外，還可以在 firstComponent 在添加自己的 `<router-outlet>`。

1. 先在 firstComponent 中添加自己的 `<router-outlet>`

    ```html
    <!-- first.component.html -->

    <h1>Component Name is: {{name}}</h1>
    <nav>
      <ul>
        <li><a routerLink="child-a">Child A</a></li>
        <li><a routerLink="child-b">Child B</a></li>
      </ul>
    </nav>

    <router-outlet></router-outlet>
    ```

2. 接著在 app-routing.module.ts 中添加 nesting routes

    ```typescript
    const routes: Routes = [
      {
        path: 'first-component/:name',
        component: FirstComponent,
        children: [
          { path: 'child-a', component: ChildAComponent },
          { path: 'child-b', component: ChildBComponent },
        ],
      },
      { path: 'second-component/:name', component: SecondComponent },
      { path: '**', component: PageNotFoundComponent },
    ];
    ```
    
![img](https://i.imgur.com/QPdsp1J.gif)

畫面中可以看到只有選擇 First Component 後才會出現 Child-a 與 Child-b 兩個導覽列，各自點選  Child-a 或 Child-b 後就會顯示對應的 component view。


# 結論

本章中介紹了什麼是 url 以及他的結構，對於現代網頁而言 url 是非常重要的觀念，需要充分的了解他的結構與該怎麼使用它。

介紹了如何透過 url 傳遞參數讓符合路徑的 component 可以獲得參數，介紹了如何利用 wildcard route 建立 pageNotFoundComponent，用於處理當使用者輸入了不屬於本專案的 route 路徑時該怎麼處理，記得要將它放在所有 route 的最後面，因為他會匹配所有的路徑，至於路徑匹配的優先權會在明天講解，最後介紹了如何做嵌套 route，在符合 route 路徑的 component 中也能有屬於他自己的 `<router-outlet>`。

由於 route 的章節比較多，所以無法一次在本篇中全部介紹所以將它分為兩篇，這樣會對每個例子或用法可以比較深入了介紹他的用法與功能，明天也會是介紹 Angular route 的下半部，會介紹如何使用相對路徑、Lazy loading 和其他好用的功能，那就明天見吧


# Reference

- [What is a URL?](https://developer.mozilla.org/en-US/docs/Learn/Common_questions/What_is_a_URL#scheme)
- [Angular.io - Common Routing Tasks](https://angular.io/guide/router)
