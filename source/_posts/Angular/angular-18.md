---
title: Day18. Introduction to services and dependency injection
date: 2021-09-19 14:53:32
tags:
- Angular
- Front-end

categories:
- 2021 鐵人賽
---

在開發專案時你一定會使用到 `Service` 的技巧，Service 是一個廣泛的類別，包括 app 所需要的任何功能、數據或特性，而通常一個 service 是一個具有狹窄且明確定義目的的 class，簡單來説可以想像成一個 service 只會對一個明確的目的提供所以有服務。

Angular 將 component 與 service 拆分開以增加模塊化與可重用性，將處理 component view 的功能與其他類型的功能拆發開來可以讓你在開發 component 時更精簡高效，所以理想的情況下 component 中的邏輯只用來處理使用者的操作，component 應該提供 property binding 的 property 與 event binding 的 method 僅此而已。

而較為複雜的邏輯任務就應該委託給 service，比如`從 server 中獲取數據`、`驗證用戶輸入`或`直接登陸到控制台`等等複雜的邏輯處理都應該在 service 中處理掉，將這些複雜的邏輯寫在 service 中就可以讓他們被任何 component 使用。

不過說了這麼多，其實 Angular 是不強制你做這些動作的，不過建議再開發大型專案時還是將複雜的邏輯從 component 中移到 service 中，保持你的 component 單純乾淨也讓 service 可以透過 inject 提供給其他 component 使用。

![https://ithelp.ithome.com.tw/upload/images/20210813/20124767bKqlcXTE3g.png](https://ithelp.ithome.com.tw/upload/images/20210813/20124767bKqlcXTE3g.png)

<!-- more -->


# What is dependency injection?

dependency injection 可以簡單地稱為 DI ，它是一種`設計模式` 當你要使用一個 class 時，可以直接從外部來源請求而不是像傳統的做法將它實例化，它存在於整個 Angualr 中，他可以讓你的程式保持靈活、可測試與彈性，component 可以在不知道如何創建 service class 的情況下使用到它的內容，簡單來說 DI 讓使用者可以不用去了解提供者的一些不重要的信息。



# Service examples

介紹完什麼是 DI 後，接著來做一個小小的例子，這是再開發專案時基本上都會遇到的

1. 首先利用 Angular CLI 建立一個 service

    ```bash
    ng generate service project
    ng g s project
    ```

2. 打開 project.service.ts

    ```typescript
    import { Injectable } from '@angular/core';

    @Injectable({
      providedIn: 'root'
    })
    export class ProjectService {

      constructor() { }
    }
    ```

    首先看到的是這個檔案使用了 `@Injectable()` 將這個 class 裝飾成 Angualr 可以在 DI 系統中使用的，接著可以看到在他的 matedata 中定義了 `provideIn: 'root'` 這代表這個 service 在整個 app 都是可以被使用的，如果不指定 provideIn 的話在 component 中將無法直接注入到 component 中，舉個例子

    ```typescript
    import { Injectable } from '@angular/core';

    @Injectable()
    export class ProjectService {

      constructor() { }
    }
    ```

    上面的 service 沒有在 metadata 中設定 providedIn，這個情況下如果跟平常一樣在 component 中注入這個 service 就會出錯

    ```typescript
    import { Component, OnInit } from '@angular/core';
    import { ProductService } from '../../second/product.service';

    @Component({
      selector: 'app-third',
      templateUrl: './third.component.html',
    })
    export class ThirdComponent implements OnInit {

      constructor(private productService: ProductService) {}    // (1)

      ngOnInit(): void {
        console.log(this.productService.getName());             // (2)
      }

    }
    ```

    - (1): 在 component 的 constructor 中注入 service
    - (2): 在 component init 後調用 service 中的 method

    ![https://ithelp.ithome.com.tw/upload/images/20210813/20124767b8wiRUhh2R.png](https://ithelp.ithome.com.tw/upload/images/20210813/20124767b8wiRUhh2R.png)

    在 console 中就會看到這個錯誤，代表他找不到 ProductService 這個 provider 可以注入到 component 中，要解決這個問題需要在 component 的 metadata 中添加 providers 這個 property 並將要注入的 service 放進去這樣就可以正常使用了

    ```typescript
    import { Component, OnInit } from '@angular/core';
    import { ProductService } from '../../second/product.service';

    @Component({
      selector: 'app-third',
      templateUrl: './third.component.html',
      providers: [ProductService]
    })
    export class ThirdComponent implements OnInit {

      constructor(private productService: ProductService) { }

      ngOnInit(): void {
        console.log(this.productService.getName());
      }

    }
    ```

3. 最後要將 service 注入到 component 中，雖然上面已經用了不過還是講一下，要注入 service 到 component 中，要在 component 的 constructor 中將它注入，這要就可以在這個 component 中使用這個 service 提供的服務

    ```typescript
    import { Component, OnInit } from '@angular/core';
    import { ProductService } from '../../second/product.service';

    @Component({
      selector: 'app-third',
      templateUrl: './third.component.html',
      providers: [ProductService]
    })
    export class ThirdComponent implements OnInit {

      constructor(private productService: ProductService) { }    // (1)

      ngOnInit(): void {
        console.log(this.productService.getName());
      }

    }
    ```

    - (1):  在 component 的 constructor 中注入 service



# 結論

本章中介紹了如何建立一個 service 並使用 dependency injection 將它注入到 component 中，雖然也介紹了什麼是 dependency injection，不過 DI 的觀念比上面介紹的在複雜更多，比如說 @Injestable() 中的 metadata providedIn 就可以填入 `root`、`module name` 或 `any` 填入每個值都有對應不同的使用場景，其中的關係有點複雜就不在本篇章中介紹，一方面是我也不是非常了解不敢隨便分享怕分享了錯誤的資訊，等之後我在摸熟一點會再開一個系列講解這方面的資訊，所以這邊就大概介紹基本的觀念而已。

下一篇將會介紹上面有提到的內容，當 @Injestable 沒有設定 `providedIn` 時需要在 component 的 metadata 中添加 `providers` 這個其實是個縮寫，詳細內容就明天再仔細講解吧，那就明天見！



# Reference

- [Angular.io - Dependency injection in Angular](https://angular.io/guide/dependency-injection)
- [Angular.io - Introduction to services and dependency injection](https://angular.io/guide/architecture-services)
- [Angular Dependency Injection Explained with Examples](https://www.freecodecamp.org/news/angular-dependency-injection/)
- [How dependency injection works in Angular](https://blog.logrocket.com/how-dependency-injection-works-in-angular/)