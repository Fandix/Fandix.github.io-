---
title: Day30. Angular Module（一）
date: 2021-09-30 09:21:11
tags:
- Angular
- Front-end

categories:
- 2021 鐵人賽
---

在 Angular 中應用程式是模塊化的，Angular 有自己的模塊化系統稱為 `NgModule`，他可以包含 `component`、`service providers` 和其他程式的文件，還可以導入從其他 NgModules 導出的功能，並導出顯訂的功能以提供給其他 NgModules 使用，每一個 Angular app 都至少有一個 NgModule 他被稱為 root module，通常會叫做 `AppModule`。

雖然一個小的專案中可能會只有一個 module (root module)，但在大多數大型的專案中都會建立多個 modules 用於將功能分割，達到將專案模塊化的目的。

![https://ithelp.ithome.com.tw/upload/images/20210829/20124767ZK4CbvmUSc.jpg](https://ithelp.ithome.com.tw/upload/images/20210829/20124767ZK4CbvmUSc.jpg)

<!-- more -->

# NgModule metadata

NgModule 是由一個用 `@NgModule()` 裝飾的 class，@NgModule() 是一個接收單個 `metadata` 物件的函數，它具有以下的 properties：

- **declarations**：屬於這個 NgModule 的 component、pipes 或 directive。
- **exports**：要導出的功能，可以讓其他的 NgModules 引入並且使用
- **imports**：引入來自其他 NgModules 所導出的功能
- **providers**：在這個 NgModule 中所使用的所有 Service
- **bootstrap**：主應用程序的 view 稱為 root component，他乘載著所有應用程式的 view，簡單來說就是定義應用程式中 view 的最上層 component

metadata 提供了該如何編譯 component 的 template 以及如何在運行時創建注入器 ( injector )，它識別了 NgModule 的 component、pipe 或 directive，並通過 exports property 將需要給其他 NgModules 使用的部分公開，還可以使用 NgModule 為 service 添加 provide ，以便 serviec 在其那地方也可以被使用。

可以在 metadata 中的 declarations 中定義哪些 component、pipes 或 directive 屬於這個 Module。

```typescript
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';

import { AppRoutingModule } from './app-routing.module';
import { AppComponent } from './app.component';

@NgModule({
  declarations: [
    AppComponent,
  ],
  imports: [
    BrowserModule,
    AppRoutingModule,
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

通常 root module 不需要設置 exports property，因為沒有其他的 NgModules 需要引入 root module。

## The declarations array

在 NgModule 的 declarations 中定義了哪些 `component`、`pipes` 或 `directive` 屬於這個 Module，當你使用 Angular CLI 創建新的 component 時會`自動將新的 component 添加到這裡`，所以如果是手動建立新的 component 時要記得要把他添加到這裡，相對的如果移除一個 component 也要記得將它從 declarations 中移除。

如果你使用一個沒有聲明的 component 則 Angular 會返回錯誤，而 declarations 只能接收可聲明的，可聲明的包括 `component`、`pipes` 或 `directive` ，Module 的所有聲明項都必須在 declarations 當中，而這個 declarations 只能屬於`一個` Module 不然會有多重宣告的錯誤，這些聲明項只有在 Module 中是可見的，除非將他們使用 `export` 導出，否則在其他地方是不可見的。

## The imports array

imports 只出現在 `@NgModule()` 的 metadata 中，它告訴 Angular 這個 Module 的運行需要使用到其他 Modules 的內容，當在此 Module 中引入其他 Module 所導出的內容時，該 Module 中的 component 就可以使用別的 Module 導出的 component、pipe 或 directive。

## The bootstrap array

Angular 的應用程式透過 root module 啟動，也稱為 `entryComponent`，在創建應用程式的過程中會創建 bootstrap 中的 component 並將它插入瀏覽器的 DOM 中，每一個 bootstrapped component 都是本身 component tree 的`基礎（最上層的 component）`，所以當插入一個 bootstrapped component 後會觸發一連串的 component 創建然後填充本身 component tree。

大多數應用程式都只有一個 component tree 並引導單個 root component，但是你也可以創建多個 component tree 來擴大你的應用程式，而這個 root component 通常稱為 `AppComponent` 並位於 root module 的 bootstrap 中。


# NgModules and components

NgModule 為 component 提供了 compilation context，簡單來說 NgModule 提供了創建 component 所需要的所有資訊，除了 root module 總是有一個 root component 之外，其他的任何 NgModules 都可以有任意數量的 component，而這個 NgModule 會為他底下的所有 component 提供 compilation context。

![https://ithelp.ithome.com.tw/upload/images/20210829/20124767olOJtbHwPU.png](https://ithelp.ithome.com.tw/upload/images/20210829/20124767olOJtbHwPU.png)

一個 component 和他的 template 定義了一個 view，但是一個 component 可以包含一個有層次結構的 view，他允許你創建、修改或銷毀一個嵌入式 view，而層次結構的 view 可以混合在屬於不同 NgModules 的 component 中定義的 view，簡單來說就是你可以在 componentA 的 view 中創建一個屬於另一個 NgModule 的 componentB 的 view。

![https://ithelp.ithome.com.tw/upload/images/20210829/20124767Cmvc8VcS1N.png](https://ithelp.ithome.com.tw/upload/images/20210829/20124767Cmvc8VcS1N.png)

當你嵌入一個 view 時，他會直接與一個稱為 `host view` 的 view 進行連接，host view 可以是這組 view 的 root，他可以包含所有嵌入的 view，就算這些 view 是來自別的 NgModules 也可以，而嵌入的 view 又可以在嵌入自己的 view，所以可以嵌套到任一深度。


# JavaScript modules vs. NgModules

在 Javascript ES6 中引入了屬於 Javascript 的 module 系統，雖然了解 Javascript 的 Module 對於了解 Angular 的 Module 很有幫助，但還是與 NgModule 有一些不同的地方，不過 Angular 同時使用了這兩種 Module 的概念

## JavaScript modules: Files containing code

Javascript Module 是帶有 Javascript 程式的`單個文件`，通常包含用於應用程式中特定目的的 class 或 library，Javascript 的 Module 可以讓你將工作分散到多個檔案中。

要讓  Module 中的內容提供給其他 Module 使用，要在這個 Module 的末尾使用 `export` 語法，比如說

```typescript
export class AppComponent { ... }
```

而要使用其他 Module 中的內容時，要使用 `import` 將它導入到自身 Module 中

```typescript
import { AppComponent } from './app.component';
```

每一個 Module 都有自己的 `top level variable`，簡單來說就是每一個 Module 中的 variable 或 method 在別的 Module 或地方是看不到的，每一個 Module 都有自己的命名空間以防止他們與其他 Module 中的內容發生衝突。


# Frequently-used modules

一個 Angular 應用程式至少需要一個 Module 作為 root module，當像應用程式添加新功能時，可以將這些功能對應的 Module 添加到你的 Module 中，下面介紹一些經常使用的 Angular module 以及他們所包含的一些內容

| NgModule | Import it from | Why you use it
| ------------- | ------------- | ------------- 
| BrowserModule | @angular/platform-browser | 當你想在瀏覽器中運行你的專案
| CommonModule | @angular/common | 當你需要使用 `NgIf` 或 `NgFor` 等等
| FormsModule | @angular/forms | 當你想要構建 template-driven forms 時
| ReactiveFormsModule | @angular/forms | 當你想要構建 reactive form 時
| RouterModule | @angular/router | 當你要使用 `RouterLink`, `.forRoot()` 或 `.forChild()` 時
| HttpClientModule | @angular/common/http | 當你需要與 server 溝通時

## BrowserModule and CommonModule

在 BrowserModule 中導入了 CommonModule，它提供了許多常用的 directive 例如 `ngIf`, `ngFor` 等等，此外 BrowserModule 重新的導出了 CommonModule 這使其所有的 directive 可以用於導入 BrowserModule 中。

對於在瀏覽器中運行的應用程式而言，在 root module 中導入 `BrowserModule` 是必須的，因為它提供了啟動和運行瀏覽器應用程式必不可少的服務，他是為整個應用程序所提供的，所以他只應該在 `root module` 中被導入，所以不可以將它放在延遲加載的 feature module 中，因為會造成錯誤。

![https://ithelp.ithome.com.tw/upload/images/20210829/20124767kFmuniMozj.png](https://ithelp.ithome.com.tw/upload/images/20210829/20124767kFmuniMozj.png)


# Guidelines for creating NgModules

通常再開發 Angular 的應用程式時會因為不同的目的創建不同的 NgModule，NgModule 是組織程式並將與特定功能或特性相關的程式與其他部分分開，使用 NgModule 將 component、pipe 和 directive 合併回內聚的功能模組，將一個功能模組集中在服務某一個功能或業務上面，所以通常會建立下面幾種類型的 NgModule

## Domain NgModules

使用 `Domain NgModule` 通常是為了專門提供於`特定功能或應用程序的用戶體驗`，比如說編輯客戶或下訂單，他組織與某個`功能`相關的程式，包括構建該功能的所有 component、routing 和 template。

通常會將最上層的 component 或 root component 作為唯一導出的 component，Domain NgModule 主要由 `declarations` 所組成。

## Routed NgModules

對所有 `lazy-loaded NgModule` 使用 `Routed NgModule`，使用 Routed NgModule 的最頂層 component 作為路由器導航的路徑，而 Routed NgModule `不會導出`任何內容，因為他們的 component 永遠不會出現在其他 component 的 template 中。

還要注意不要將 `lazy-loaded NgModule` 導入另一個 NgModule 中，因為這樣會會立馬將這個 Module 載入到應用程式中，就失去了 lazy-loaded 的目的了。

Routed NgModule 只需要很少的 `providers`，因為對於應用程式而言只有在需要的時候才會載入 Routed NgModule，所以如果在他的 providers 中填入 service 的話將會變得不可用，因為 injector 不知道 lazy-loaded NgModule。

## Routing NgModules

`Routing NgModule` 為 `Domain NgModule` 提供路由配置，通常會由 Routing NgModule 完成以下任務

- 定義路由
- 將路由器配置添加到 NgModule 的 imports 中
- 向 NgModule 的 providers 提供 service

Routing NgModule 的名稱應該與其同伴的 NgModule 名稱平行，比如說 contact.module.ts 中的 ContactModule 在 contact-routing.module.ts 中有一個名為 ContactRoutingModule 的 Routing NgModule。

如果 Routing NgModule 是 root App Module 則 AppRoutingModule 是使用 `RouterModule.forRoot(routes)` 將路由器配置添加在 imports 中

```typescript
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';

const routes: Routes = [];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule { }
```

而如果是其他 Routing NgModule 都是使用 `RouterModule.forChild(routes)` 將路由器配置添加到他們的 imports 中。

要注意`不要`在 Routing NgModule 中使用 `declarations`，component、pipe 或 directive 的聲明是在伴隨的 `Domain NgModule` 中聲明的。

## Service NgModules

`Service NgModule` 用於提供實用的服務，例如數據的訪問或消息的傳遞，理想的 Service NgModule 應該全由 providers 所組成，如果要創建 Service NgModule 可以查看 Angular 的 `HttpClientModule` 他是一個非常好的例子，要注意的是請只在 `root AppModule` 中導入所有的 Service NgModules

## Widget NgModules

使用 Widget NgModule 使 component、pipe 或 directive 可被外部的 NgModule 給使用，當某個 NgModule 中的 component 需要使用到 Widget NgModule 中的某一個 component 或功能時，將

Widget NgModule 到入到這個 NgModule 中即可，需多第三方 UI component libaray 都是作為 Widget NgModule 然後提供給各個 NgModule 中， 一個 Widget NgModule 應該`完全`由 `declarations` 所組成，其中大部分都是導出的這樣才能讓其他地方使用。

## Shared NgModules

最後介紹 Shared NgModule 他是將常用的 directive、pipe 或 component 統一放入一個 NgModule 中，通常會命名為 `SharedModule`，然後再應用程式的其他部分需要時導入，這樣只導入一個 Module 就可以獲得許多功能。

Shared NgModule 不應該包含 `providers`，任何其導入或導出的 NgModule 都不應該包含 providers。


# 結論

本篇中介紹了 Angular 的 NgModule 的觀念，NgModule 中 metadata 是由 `declarations`、`exports`、`imports`、`providers` 與 `bootstrap` 所組成

- **declarations** 是用於宣告屬於這個 NgModule 的 component、pipe 或 directive
- **exports** 是用於將這個 NgModule 需要給其他 NgModule 使用的部分導出
- **imports** 是用於引入其他 NgModule 導出的功能
- **providers** 是用於提供 NgModule 所使用的 Services
- **bootstrap** 是用於定義應用程式的 `entryComponent`

也介紹了 Angular 同時使用了 Javascript Module 與 NgModule 的概念，對於一個檔案中的內容要導出的給其他部分使用的話要使用 `export`，而要使用別的檔案中提供的內容時需要使用 `import` 將它導入。

最後介紹了再開發 Angular 專案時會建立的幾種 NgModule

- **Domain** 是圍繞功能或用戶體驗所組成的
- **Routed** 中的頂部 component 充當路由器導航路徑的目的地
- **Routing** 是為另一個 NgModule 提供路由配置的
- **Service** 是提供實用服務的，例如數據訪問或消息傳達
- **Widget** 是將某一部分的 component, pipe, directive 所組合起來讓其他 NgModule 可以使用，常用於第三方 UI library
- **Shared** 是將共享的所有 component, pipe, directive 所組合，當其他 NgModule 要使用時引入它即可

Angular Module 的觀念還有很多，由於文章長度的問題沒辦法一次講完，所以一樣將他分為兩段這樣可以介紹的比較完整，下半部的 NgModule 就明天見吧


# Reference

- [Angular.io - Guidelines for creating NgModules](https://angular.io/guide/module-types)