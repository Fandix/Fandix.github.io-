---
title: Day32. Lazy-loading feature modules
date: 2021-10-02 20:46:01
tags:
- Angular
- Front-end

categories:
- 2021 鐵人賽
---

在默認情況下 NgModules 都是`急切加載`的，這意味著一但應用程式加載，所有的 NgModules 無論是否需要都會被載入，但對於許多大型的應用程式而言可以考慮使用`延遲加載`功能，他是一種根據`需要才加載` NgModules 的設計模式，延遲加載有助於讓初始化的檔案保持為較小的大小，從而`減少加載的時間`，下面就來介紹該如何使用延遲加載模式吧。

![https://ithelp.ithome.com.tw/upload/images/20210905/201247675kZ9WQlQIh.jpg](https://ithelp.ithome.com.tw/upload/images/20210905/201247675kZ9WQlQIh.jpg)

<!-- more -->

# Step-by-step setup

要設置延遲加載的 feature module 主要有兩個步驟：

1. 使用 `--route` 通過 Angular CLI 創建 feature module
2. 配置 routes

接著來看看該怎麼創建一個新的延遲載入的 feature module 吧

## Create a feature module with routing

首先需要創建一個帶有 `route` 功能的 feature module，並這個 route 的路徑要指定到 feature module 宣告的某一個 component，可以使用 Angular CLI 一次就完成所有動作，所以是不是很需要 Angular CLI 呢（ gif

```bash
ng generate module customers --route customers --module app.module
```

上面的 CLI Command 會創建一個名為 customers 的文件夾，其中的 customers.module.ts 中會定義一個延遲載入的 feature module 名為 `CustomersModule`，還會創建一個用於定義 route 的 route module 名為 `CustomersRoutingModule`，除此之外還會建立一個 CustomerComponent 並自動導入到 CustomersRoutingModule 的 `declarations` 中。

```typescript
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';

import { CustomersRoutingModule } from './customers-routing.module';
import { CustomersComponent } from './customers.component';

@NgModule({
  declarations: [
    CustomersComponent
  ],
  imports: [
    CommonModule,
    CustomersRoutingModule
  ]
})
export class CustomersModule { }
```

```typescript
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';
import { CustomersComponent } from './customers.component';

const routes: Routes = [{ path: '', component: CustomersComponent }];

@NgModule({
  imports: [RouterModule.forChild(routes)],
  exports: [RouterModule]
})
export class CustomersRoutingModule { }
```

由於這個新的 feature module 是延遲加載的所以這個 Command 不會在 root module 中添加對這個 fearute module 的引用，因為如果將它加入到 root module 的 imports 的話就會變成`立即加載`，相反的他會將這個 feature module 添加到 route 中

```typescript
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';

const routes: Routes = [
  {
    path: 'customers',
    loadChildren: () =>
      import('./customers/customers.module').then((m) => m.CustomersModule),
  },
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule],
})
export class AppRoutingModule {}
```

注意 `loadChildren` 的用法，他後面需要加一個函數，這個函數會使用瀏覽器內置的 `import()` 語法進行動態導入，導入 module 的路徑是`相對路徑`。

## Set up the UI

設定完延遲載入的 module 後，接著來設定一下 UI 畫面讓我們等等可以測試這個延遲載入是否成功，在 app.component.html 中設定按鈕並加上 `<router-outlet>` 用於顯示 route 的 component

```html
<h1>{{title}}</h1>

<button type="button" class="btn btn-primary" routerLink="/customers">Customers</button>
<button type="button" class="btn btn-success" routerLink="">Home</button>

<router-outlet></router-outlet>
```

![https://ithelp.ithome.com.tw/upload/images/20210905/20124767Ws1vG3NoVD.png](https://ithelp.ithome.com.tw/upload/images/20210905/20124767Ws1vG3NoVD.png)

接著來測試一下會不會當點擊 Customers 按鈕後才會加載 feature module 吧，首先先先打開瀏覽器的開發者頁面並選到 Network 頁面，可以將前面載入的資訊先去除以便比較清楚的看到加載過程

![img](https://i.imgur.com/3NtPEj5.gif)

可以看到當我們點擊了 Customers 按鈕後才會載入延遲載入的 feature module。


# forRoot() and forChild()

如果使用 Angular CLI 創建一個 root module 的話，你會看到他將 `RouterModule.forRoot(route)` 添加到 AppRoutingModule 的 `imports` 中，這會讓 Angular 知道 AppRoutingModule 是一個 route module，而 forRoot() 指定這是 `root route module`，他會配置傳遞給他的所有 route 讓你可以訪問 route directive 並註冊 route service，在整個應用程式中 AppRoutingModule 只會使用 forRoot() 一次。

Angular CLI 還會將 RouterModule.forChild(routes) 添加到 feature module 的 `imports` 中，這樣 Angular 就會知道這個只是負責提供額外的 route，並是針對 feature module 的，所以你可以在多個 route modules 中使用 forChild()。


# Provider service in lazy-loading modules

在前幾篇中提到了如果在 root module 中提供 service 時 Angular 會將它建立成一個`單一的`、`共享的` service 實例，所以將 service provider 添加到 root module 後，代表所有地方都可以將它注入並且使用。

但是如果是將 service provider 到 lazy-loading 的 module 時，由於 lazy-loading 的特性，因為這個 module 還沒被載入所以這個 lazy-loading 中添加的 service 就無法在還沒被載入的時候被其他地方的 module 使用，舉個例子

1. 先新增一個 service 並添加一個簡單的 method

    ```typescript
    import { Injectable } from '@angular/core';
    import { CustomersModule } from './customers.module';

    @Injectable()
    export class CustomersService {

      constructor() { }

      getName() {
        return 'Fandix Huang';
      }
    }
    ```

2. 接著在 CustomersModule 的 providers 添加 CustomersService

    ```typescript
    import { NgModule } from '@angular/core';
    import { CommonModule } from '@angular/common';

    import { CustomersRoutingModule } from './customers-routing.module';
    import { CustomersComponent } from './customers.component';
    import { CustomersService } from './customers.service';

    @NgModule({
      declarations: [
        CustomersComponent
      ],
      imports: [
        CommonModule,
        CustomersRoutingModule
      ],
      providers: [CustomersService]
    })
    export class CustomersModule { }
    ```

這時候如果在 app.component.ts 中注入這個 service 的話就會出錯，因為他被 provide 在 lazy-loading 的  module 中，所以還沒被載入的話大家都不知道這個 service 是什麼

```typescript
import { Component, OnInit } from '@angular/core';
import { CustomersService } from './customers/customers.service';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent implements OnInit {
  title = 'Angular blog';
  constructor(private customersService: CustomersService) {}

  ngOnInit() {
    console.log(this.customersService.getName())
  }
}
```

![https://ithelp.ithome.com.tw/upload/images/20210905/20124767YrfNPAxGVj.png](https://ithelp.ithome.com.tw/upload/images/20210905/20124767YrfNPAxGVj.png)

但是如果是注入在 CustomersModule 中 declarations 的 component 中的話就可以正常使用

```typescript
import { Component, OnInit } from '@angular/core';
import { CustomersService } from './customers.service';

@Component({
  selector: 'app-customers',
  templateUrl: './customers.component.html',
  styleUrls: ['./customers.component.css']
})
export class CustomersComponent implements OnInit {

  constructor(private customerService: CustomersService) { }

  ngOnInit(): void {
    console.log(this.customerService.getName())
  }
}
```

![https://ithelp.ithome.com.tw/upload/images/20210905/20124767GKOxZkfphj.png](https://ithelp.ithome.com.tw/upload/images/20210905/20124767GKOxZkfphj.png)



# 結論

本章中介紹了如何建立與使用延遲載入的 module，雖然在中小專案中可以讓所有 module 都是立即載入的，但隨著專案變大這樣所有 module 都是立即載入的話就會讓初始檔案變得非常大，因為應用程式一開始就要載入全部的 module，會讓整個應用程式的載入變得非常慢，這時候就需要使用延遲載入的功能，當有需要才載入對應的 module 這樣可以讓一開始的專案維持在比較小的大小，從而加快應用程式的載入速度。

本章是介紹 module 的最後一張，下一章將會介紹也是常重要的一個部分，在現代的網頁中都會需要串接後端的 API 以獲得數據或是儲存數據到 database，這些操作都需要與後端溝通，Angular 提供了一個方式讓我們可以與後端進行溝通，那就是 `HttpClient` 詳細的內容就留到明天講解吧，那就明天見



# Reference

- [Angular.io - Lazy-loading feature modules](https://angular.io/guide/lazy-loading-ngmodules)