---
title: Day19. Dependency providers
date: 2021-09-19 21:30:55
tags:
- Angular
- Front-end

categories:
- 2021 鐵人賽
---

在上一篇中提到了如何建立與使用一個 Service，也大概介紹了什麼是 Dependency Injection，在介紹使用 @Injectable() 裝飾 service 的 class 的時候提到了要在他的 metadata 中設定 `providedIn`，如果將它設定為 `root` 的話代表這個 service 在整個專案中都是可被使用的，但如果沒有設定這個 property，則要在 component 中注入這個 service 時需要在 @Component() 的 metadata 中設定 providers ，將要注入的 service 放進去，在上一章中的用法是 `providers: [ProductService]`，這其實是一個縮寫，本篇就是要來詳細的介紹他，那我們就繼續往下看吧！

![https://ithelp.ithome.com.tw/upload/images/20210821/20124767ZpsjOVE7SU.jpg](https://ithelp.ithome.com.tw/upload/images/20210821/20124767ZpsjOVE7SU.jpg)

<!-- more -->

# Dependency injection tokens
在上一章中提到了什麼是 Dependency injection 不過這邊再來複習一下，這有利於我們接下來要講的內容，簡單來說 Dependency injection 又稱 DI 它是一種設計模式和機制，他用於將某些部分與依賴相分離，以 Angular 的例子來說，他就是用於將複雜的邏輯與 Component 中負責處理畫面或事件的邏輯分離，而這些被分離出來的部分也可以給其他 Component 使用，使它變成可重複使用且可測試的一個部分。

當整個應用程式啟動時，會創建一個名為 `injector` 的東西，可以將它看成會檢查客戶訂單且滿足客戶要求的服務生，舉例來說他可以製作一些咖啡（實例化 CoffeeService）將它提供給客戶，而如果有其他的客戶想要喝茶，那麼這個聰明的服務生就會實例化 MintTeaService 和 CamomileTeaService 用於滿足不同客戶的需求。

然而要讓我們的服務生（injector）知道該出什麼餐點則需要客戶點單（component），他可以在 component 中的 constructor 中要求他要吃什麼（dependency）

```typescript
constructor(private coffee: CoffeeService){};
```

`coffee: CoffeeService` 就是一個 `Dependency injection tokens` 它是一種 inject class 的快捷方式，現在我們的服務生從客戶那邊收集了所有的訂單（ injector 向 component 收集需求 ），接著 服務生（injector） 會使用 CoffeeService 來找咖啡的標記，將客戶點的咖啡記在他的筆記本上。

記錄完客戶的點單後，接著就需要知道如何製作指定的咖啡，這時就需要配方（provider）了，這個配方是一個 object 他會定義如何獲取與 Dependency injection tokens 有關連的可注入依賴項，簡單來說他會告訴服務生（injector）該如何製作這杯咖啡（ 將 service 實例化成一個 object 並將它注入給 component ）

```typescript
providers: [CoffeeService, BurgerService, PizzaService]
```

上面的例子來說服務生將知道如何做出咖啡、漢堡、Pizza 並提供給客戶，以程式的角度來看可以看成 injector 知道 CoffeeService 要使用 CoffeeService 作為模板將它實例化成一個 object 並注入給 component 讓他使用，這邊可能看不出來可以換另一種更詳細的表達方式

```typescript
providers: [
	{ provide: CoffeeService, useClass: CoffeeService },
    { provide: BurgerService, useClass: BurgerService },
	{ provide: PizzaService, useClass: PizzaService }
]
```

這樣應該更可以了解，provide property 作爲 locating a dependency value 和 configuring the injector 的 Token，可以把它理解為他是一個 ID，以服務生的例子來說他就等於是客戶點的餐點名稱，以程式的觀點來看就等於他是 component 需要使用的 service name。

而第二個 property 是一個提供 provider 定義的一個 object，他會告訴 injector 該如何建立 dependency 的值，以上面的服務生例子來說他就是菜單，他會告訴服務生該怎麼製作客戶點的咖啡，而這個定義 provider 的 key 可以是 `useClass` 就像上面使用的，也可以是 `useExisting`、`useValue` 或 `useFactory` 他們每一個都提供了不同類型的 dependency，下面將仔細介紹他們的區別。

![https://ithelp.ithome.com.tw/upload/images/20210815/20124767PRR6NqC8Uy.png](https://ithelp.ithome.com.tw/upload/images/20210815/20124767PRR6NqC8Uy.png)



# Specifying an alternative class provider

首先介紹的是 `useClass` ，不同的 class 可以提供給相同的 service，比如說

```typescript
[{ provide: Logger, useClass: BetterLogger }]
```

上面的例子中代表 component 向 injector 説他需要使用 Logger，而 provider 向他提供了 `BetterLogger` 這個 class 讓 injector 將它實例化後注入給 component 使用。

### Configuring class providers with dependencies

如果使用 useClass 的這個 class 有自己的 dependencies，則要在父層 module 或 component 的 metadata 中的 providers 提供他自己於他的 dependencies，舉個例子

```typescript
@Injectable()
export class EvenBetterLogger extends Logger {
  constructor(private userService: UserService) { super(); }

  log(message: string) {
    const name = this.userService.user.name;
    super.log(`Message to ${name}: ${message}`);
  }
}
```

```typescript
@Component({
	providers: [UserService, { provide: Logger, useClass: EvenBetterLogger }]]
})
```

在 component 中使用 useClass 的 class 有自己的  dependencies （UserService），所以在 component 的 metadata 中也需要提供 UserService。



# Aliasing class providers

在我們使用 useClass 注入指定 class 內容時可能會遇到一個情況，一開始我們建立了兩個 service 分別是 `oldService` 與 `newService`，這兩個 service 一開始負責不同的服務但是到後來的增加跟刪除下，發現只要使用 newService 就好，當遇到這種情況可能有些人會到每個有使用到 oldService 的 component 或 module 去把它改掉，但其實 Angular 他提供了另一個方法那就是別名。

在提到這個方法之前先來釐清一下 useClass 的用法

```typescript
providers: [{provide: Class1, useClass: Class1}]
```

當使用上面這個方法可以等價為將 Class1 注入到 component 或 module 中

```typescript
providers: [{provide: Class1, useClass: Class3}]
```

而上面這個方法可以看成在 Component 或 module 中有一個名叫 Class1 的 Token 要用 Class3 創建並注入，那麼問題來了下面這樣該怎麼解釋？

```typescript
providers: [
	Class3,
	{provide: Class1, useClass: Class3}
]
```

其實很簡單，你可以想像在這個 component 或 module 中有一個 Class3 的 Token 利用 Class3 創建並注入，還有第二個 Token 名叫 Class1 但是也是用 Class3 創建並注入，等於說這個 component 或 module 有`兩個 class3 的實例`。

有上面的概念後就可以來看 `useExisting` ，在一開始提到的我們希望 `oldService` 都替換成 `newService` 這時有人會下意識的覺得

```typescript
providers: [
	newService,
	{provide: oldService, useClass: newService}
]
```

上面的這種改法可以看成在這個 component 或 module 中注入一個 newService 的實例並且在將一個名為 oldService 的 Token 也使用 newService 創建，等於說會有兩個 newService 這是我們不希望看見的，這時就可以用 `useExisting` 改寫

```typescript
providers: [
	newService,
	{provide: oldService, useExisting: newService}
]
```

上面的寫法改成使用 `useExisting` 就可以避免建立兩個 newService 的情況了，可以想像是 `newService` 這個實例但是用了 `oldService` 這個別名。



# Injecting an object

除了使用 useClass 直接提供一個 class 之外，也可以使用 `useValue` 將一個 object 提供給 injector，舉例來說

```typescript
export const silentLogger = {
	logs: ['Silent logger says "Shhhhh!". Provided via "useValue"'],
	log: () => {
		console.log('You are log in');
	}
}
```

```typescript
@Component({
	providers: [{ provide: Logger, useValue: silentLogger }]
})
```

上面的例子中可以看到，在別的地方建立一個 Object 並將它提供給 injector 讓他注入給 component 中讓 component 中可以使用到這個 Object 中的 property 與 method。

## Using an InjectionToken object

在上面可以看到我們在填入 property provide 的時候都是預定他是一個 service，那們可不可以填入某個 object 或一個 function 呢？答案是可以的，不過你得使用 `InjectionToken` 強制將你想填入的內容產生一個 token，舉個例子吧

```typescript
import { Component, Inject, OnInit, InjectionToken } from '@angular/core';

export const APP_CONFIG = new InjectionToken<{                          // (1)
  apiEndpoint: string;
  title: string;
}>('app.config');

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  providers: [                                                            // (2)
    {
      provide: APP_CONFIG,
      useValue: {
        apiEndpoint: 'api.heroes.com',
        title: 'Dependency Injection',
      },
    },
  ],
})
export class AppComponent implements OnInit {
  title!: string;
  apiEndpoint!: string;
  constructor(
    @Inject(APP_CONFIG) config: { apiEndpoint: string; title: string }    // (3)
  ) {
    this.title = config.title;
    this.apiEndpoint = config.apiEndpoint;
  }

  ngOnInit() {
    console.log(this.title);
    console.log(this.apiEndpoint);
  }
}
```

- (1): 替 APP_CONFIG 產生一個 token，參數的字串（'app.config'）只是對他的描述
- (2): 將 APP_CONFIG 使用 useValue 提供模板給 injector
- (3): 使用 `@Inject()` 裝飾器告知使用哪一個 token 注入

![https://ithelp.ithome.com.tw/upload/images/20210815/20124767fXP00j0QCs.png](https://ithelp.ithome.com.tw/upload/images/20210815/20124767fXP00j0QCs.png)



# Using factory providers

在介紹完 `useValue` 後可以看到就算是普通的 object 也可以使用 InjectionToken 強制產生一個 token，不過要這樣做的前提是要事先建立好要替代的 token 實體，但這種事先建立的情況在實際中不太常發生，比較多都是需要動態產生的，這時就可以使用 `useFactory` 把複雜的動態計算放在 factory 裏面，動態的建立需要的 token 實體，舉個例子
1. 建立 service1 與 service2

    ```bash
    ng generate service service1
    ng generate service service1
    ```

2. 在兩個 service 中添加一個 getName() method 但是回傳不同的字串

    ```typescript
    import { Injectable } from '@angular/core';

    @Injectable({ providedIn: 'root' })
    export class Service1Service {
      constructor() { }
      getName() {
        return 'This is Service-1 method'
      }
    }
    ```

    ```typescript
    import { Injectable } from '@angular/core';

    @Injectable({ providedIn: 'root' })
    export class Service2Service {
      constructor() { }
      getName() {
        return 'This is Service-2 method'
      }
    }
    ```
    
3. 在 app.component.ts 中使用 useFactory
    ```typescript
    import { Component, InjectionToken, OnInit  } from '@angular/core';
    import { Service1Service } from './service1.service';
    import { Service2Service } from './service2.service';
    import { AppService } from './app.service';

    export enum ServiceList {                                                 // (1)
      SERVICE_1 = 0,
      SERVICE_2 = 1
    }

    export const SelectService = new InjectionToken<number>('selectService'); // (2)

    export const serviceFactory = (selectService: number) => {                // (3)
      if (selectService === ServiceList.SERVICE_1) {
        return new Service1Service();
      } else {
        return new Service2Service();
      }
    }

    @Component({
      selector: 'app-root',
      templateUrl: './app.component.html',
      providers: [
        {
          provide: SelectService,                                             // (4)
          useValue: ServiceList.SERVICE_1
        },
        {
          provide: AppService,                                                // (5)
          useFactory: serviceFactory,
          deps: [SelectService]
        }
      ],
    })
    export class AppComponent implements OnInit {

      constructor(private appService: AppService) {}                          // (6)

      ngOnInit() {
        console.log('Useing Service: ', this.appService.getName());
      }
    }
    ```
    - (1): 定義一個選擇使用哪一個 service 的 enum
    - (2): 使用剛剛介紹的 `Using an InjectionToken object` 為 SelectService 這個 number 產生一個 Token
    - (3): 建立一個 Factory function，傳入 select number 用來決定要使用哪一個 service
    - (4): 使用剛剛介紹的 `useValue` 將 SelectService Token 賦予 ServiceList.SERVICE_1 的值並將他創建出來注入給 app.component
    - (5): 使用 `useFactory` 將 Factory function 的結果提供給 AppService Token 並將他創建出來注入到 app.component 中
    - (6): 在 component 的 constructor 中注入 service

這邊特別介紹一下 `deps` 這個 property，他是 injector 要解析的 Token list，然後他列表中的值會作為 useFactory 的參數傳入 factory function 中，這就是為什麼上面的例子中要先用 `InjectionToken` 產生一個 Token，這樣 injector 才能將 SelectService 解析並將其中的值，也就是使用 `useValue` 賦予的值傳遞近 factory function 裏面。

如果使用 useValue 設定值為 `ServiceList.SERVICE_1` 那們在 console 中會看到
![https://ithelp.ithome.com.tw/upload/images/20210815/20124767nVTPjutgPn.png](https://ithelp.ithome.com.tw/upload/images/20210815/20124767nVTPjutgPn.png)
app.component 中注入的 service 是 service1 的內容，而 useValue 設定值為 `ServiceList.SERVICE_2` 時
![https://ithelp.ithome.com.tw/upload/images/20210815/20124767AUS3HBvDqV.png](https://ithelp.ithome.com.tw/upload/images/20210815/20124767AUS3HBvDqV.png)
app.component 中注入的 service 變為 service2 的內容，這就是 `useFactory` 的概念，不過實際上 useFactory 用法會遠比上面的例子還要複雜得多，不過這邊介紹的是一個概念。


# 結論

本章中介紹了什麼是 injector 與 provider 和他們之間的關係，用了一個客戶與服務生的例子，不過一樣實際上背後的原理比這個複雜得多，不過這邊就是介紹他的概念不會太過鑽研他背後的原理。

也介紹了 provider 可以透過使用 `useClass`、`useValue`、`useFactory` 或 `useExisting` 將 component 或 module 需要的 Token 利用不同的來源建立出來並注入回去，基本上大多都使用場景都是使用 useClass 跟 useValue，不過在面對比較複雜的專案時也會需要用到另外兩個，這樣才能使你的專案更加的靈活。

下一篇將會介紹 Angular 的 Router，他在 Angular 中也是佔有非常重要的地位，現代的網頁中沒有人只做一頁的，所以就需要 Router 來控制不同的 url 會進到那一個頁面，詳細的內容就留到明天再講解吧，那們一樣明天見囉！


# Reference

- [Angular.io - Dependency providers](https://angular.io/guide/dependency-injection-providers)
- [Angular.io - FactorySansProvider](https://angular.io/api/core/FactorySansProvider#deps)
- [Angular.io - FactoryProvider](https://angular.io/api/core/FactoryProvider)
- [[Angular 大師之路] Day 21 - 在 @NgModule 的 providers: [] 自由更換注入內容 (1)](https://ithelp.ithome.com.tw/articles/10207990)
- [[Angular 大師之路] Day 21 - 在 @NgModule 的 providers: [] 自由更換注入內容 (2)](https://ithelp.ithome.com.tw/articles/10208240)
- [Angular里useExisting和useClass的使用场景](https://cloud.tencent.com/developer/article/1739373)
- [This Won’t Hurt a Bit — Dependency Injection Tokens in Angular](https://medium.com/ngconf/this-wont-hurt-a-bit-dependency-injection-tokens-in-angular-2fa5f6e6293)
- [Angular Injector, @Injectable & @Inject](https://www.tektutorialshub.com/angular/angular-injector-injectable-inject/)
- [Dependency Injection in Angular](https://blog.thoughtram.io/angular/2015/05/18/dependency-injection-in-angular-2.html)
