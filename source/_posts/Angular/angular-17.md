---
title: Day17. Dynamic component loader
date: 2021-09-17 10:31:54
tags:
- Angular
- Front-end

categories:
- 2021 鐵人賽
---

介紹完什麼是 template 與 structuarl directive 後，接著回來介紹 component 中被跳過的章節，那就是動態仔入 component，再開發專案時可能會遇到 component template 需要被動態載入的情況，比如說常見的網頁廣告或是當捲軸轉到某個地方時才會顯示出只定 component 的 template，那麼就繼續看下去吧。

![https://ithelp.ithome.com.tw/upload/images/20210818/20124767jPdUbmDQQh.png](https://ithelp.ithome.com.tw/upload/images/20210818/20124767jPdUbmDQQh.png)

<!-- more -->

# Dynamic component loading

以 Angular 官方文檔的例子來介紹一下如何使用 Dynamic component loading，這個例子將會製作一個畫面中的廣告，會隨著時間而顯示不同的廣告內容，要滿足這個條件使用過去靜態 component 載入就顯得不切實際，讓我們一起看看這個例子吧



# The anchor directive

在添加動態 component 之前，需要先使用 directive 來讓 Angular 知道你要將這的動態 component 插入在哪邊，使用 Angular CLI 建立一個 directive

```bash
ng generate directive Ad
```

接著在 ad.directive.ts 中從 `@angular/core` 中引入 `ViewContainerRef` 並將它注入到 class 中

```typescript
import { Directive, ViewContainerRef } from '@angular/core';

@Directive({
  selector: '[appAd]'
})
export class AdDirective {
  constructor(public viewContainerRef: ViewContainerRef) { }
}
```

ViewContainerRef 讓你可以訪問動態 component 的 view container。



# Loading components

接著我們將要動態顯示廣告的邏輯定義在 ad-banner 中，所以使用 Angular CLI 建立一個 component

```bash
ng generate component ad-banner
```

接著在 ad.banner.html 中添加 `<ng-template>` 與使用 `directive` 綁定這個元素

```html
<div class="ad-banner-example">
  <h3>Advertisements</h3>
  <ng-template adHost></ng-template>
</div>
```

在 template 中使用 `<ng-template>` 對於要動態載入 component 的 view 而言非常適合，因為在尚未滿足條件的情況下 Angular 並不會將 `<ng-template>` 的內容放入 DOM 中。



# Create view component

在完成 ad-banner 後，接著要建立負責顯示廣告畫面的 component，一樣先用 Angular CLI 建立 component

```html
ng generate component hero-job-ad
ng generate component hero-profile
```

這邊的設計比較特別，不像一般傳統的 component 設計，他要將這兩個 component 做為參數傳遞給某個 method，藉由這個 method 將 component 實例話而不是像之前的使用他的 `selector`

```typescript
import { Component, Input } from '@angular/core';

@Component({
  template: `
    <div class="job-ad">
      <h4>{{ data.headline }}</h4>

      {{ data.body }}
    </div>
  `,
})
export class HeroJobAdComponent {
  @Input() data: any;
}
```

```typescript
import { Component, Input } from '@angular/core';

@Component({
  template: `
    <div class="hero-profile">
      <h3>Featured Hero Profile</h3>
      <h4>{{data.name}}</h4>

      <p>{{data.bio}}</p>

      <strong>Hire this hero today!</strong>
    </div>
  `
})
export class HeroProfileComponent {
  @Input() data: any;
}
```

由於不需要使用 `selector` 所以將不需要的 property 從 meatdata 中移除只留下 template 設定畫面。



# Create service

在建立完顯示畫面的 component 後，剛剛提到的要將這兩個 component 傳給某個 method 將它實例化，那麼就要建立一個 class 用於將他們實例化，這邊手動新增一個檔案就好

```typescript
// ad-item.ts

import { Type } from '@angular/core';

export class AdItem {
  constructor(public component: Type<any>, public data: any) {}
}
```

這邊使用了 `@angular/core` 中的 `Type` 代表定義的參數的型態是 component 或是其實例，這樣才滿足我們要將 component 傳進這個 class 後實例化的目的。

接著建立一個 service 用於利用剛剛建立出的兩個 component 建立畫面，首先一樣使用 Angular CLI 建立一個 service

```bash
ng generate service ad
```

```typescript
import { Injectable } from '@angular/core';
import { HeroJobAdComponent } from './view/hero-job-ad/hero-job-ad.component';
import { HeroProfileComponent } from './view/hero-profile/hero-profile.component';

import { AdItem } from './ad-item';

@Injectable({
  providedIn: 'root',
})
export class AdService {
  constructor() {}

  getAds() {
    return [
      new AdItem(HeroProfileComponent, {
        name: 'Bombasto',
        bio: 'Brave as they come',
      }),
      new AdItem(HeroProfileComponent, {
        name: 'Dr IQ',
        bio: 'Smart as they come',
      }),
      new AdItem(HeroJobAdComponent, {
        headline: 'Hiring for several positions',
        body: 'Submit your resume today!',
      }),
      new AdItem(HeroJobAdComponent, {
        headline: 'Openings in all departments',
        body: 'Apply today',
      }),
    ];
  }
}
```

在 service 中新增一個獲取廣告的 method，將剛剛建立的 component 作為參數傳遞給 `AdItem` 並將 component 需要的 Input 參數也傳遞進去。


# Finish this project

建立完這些工具後，最後要將它們組合起來讓畫面顯示動態的 component 畫面，首先先在 app.component.ts 中注入剛剛寫的 adService 並調用 method 獲得英雄廣告的列表

```typescript
import { Component, OnInit } from '@angular/core';
import { AdService } from './ad.service';
import { AdItem } from './ad-item';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
})
export class AppComponent implements OnInit {
  ads: AdItem[] = [];
  constructor(private adService: AdService) {}

  ngOnInit() {
    this.ads = this.adService.getAds();
  }
}
```

接著在 app.component.html 中使用 ad.banner 的 selector 並將 ads 綁定為他的 input binding

```html
<!-- app.component.html -->

<app-ad-banner [ads]="ads"></app-ad-banner>
```

再來要進到 ad-banner.component.ts 中增加動態 component 的邏輯
```typescript
import { Component, OnInit, Input, ComponentFactoryResolver, ViewChild, OnDestroy } from '@angular/core';
import { AdItem } from '../ad-item';
import { AdDirective } from '../ad.directive';

@Component({
  selector: 'app-ad-banner',
  templateUrl: './ad-banner.component.html',
})
export class AdBannerComponent implements OnInit, OnDestroy {
  @Input() ads: AdItem[] = [];                                                // (1)
  @ViewChild(AdDirective, {static: true}) adHost!: AdDirective;               // (2)
  interval: any;                                                              // (3)
  currentAdIndex = -1;                                                        // (4)

  constructor(private componentFactoryResolver: ComponentFactoryResolver) { } // (5)

  ngOnInit(): void {
    this.loadComponent();
    this.getAds();
  }

  ngOnDestroy() {
    clearInterval(this.interval);                                             // (8)
  }

  loadComponent() {                                                           // (6)
    this.currentAdIndex = (this.currentAdIndex + 1) % this.ads.length;
    const adItem = this.ads[this.currentAdIndex];

    const componentFactory = this.componentFactoryResolver.resolveComponentFactory(adItem.component);

    const viewContainerRef = this.adHost.viewContainerRef;
    viewContainerRef.clear();

    const componentRef = viewContainerRef.createComponent<{ data: any }>(componentFactory);
    componentRef.instance.data = adItem.data;
  }

  getAds() {                                                                  // (7)
    this.interval = setInterval(() => {
      this.loadComponent();
    }, 3000);
  }
}
```

這邊的邏輯比較複雜一點，來一一說明一下：

- (1): 利用 `@Input()` 將 ads 裝飾為是父層傳遞的數據，並使用 `AdItem` 指定型別。
- (2): 利用  `@ViewChild` 將 adHost 裝飾為可以訪問到 view element 的 property，可以直接在 ad-banner.component 中直接使用 adDirective 中的 method，而將 `static` 設定為 true 代表在更改檢測運行之前會先解析查詢的結果。
- (3): 建立一個 property 用來接收 setInterval 回傳的值，主要用於取消計時器。
- (4): 建立一個 property 用來計算目前要顯示第幾個英雄廣告。
- (5): 將 `ComponentFactoryResolver` 注入到 component 中，主要用來將選擇的 ad-component 解析為一個 `componentFactory`。
- (6): 建立一個 method 用於計算要顯示第幾個英雄廣告並將被選中的英雄廣告 component 透過 `ComponentFactoryResolver` 解析為 `componentFactory` 並將它利用 `createComponent` 實例化。
- (7): 建立一個 method 用於建立一個計時器，每過 3 秒就取得一次英雄廣告
- (8): 在 ngObDestory() 中取消計時器

![img](https://i.imgur.com/EvEFuPO.gif)

在畫面中看到每過三秒就會更換一次畫面，打開網頁中的 conosle 檢查一下

![img](https://i.imgur.com/1atiuaj.gif)

可以看到每過三秒就會更換一次 component，這就是動態載入 component。


# 結論

本篇中使用了滿多之前提到的技巧來完成這個動態載入 component 的功能，可以一邊看 Angular 提供的 [stackbitz](https://stackblitz.com/angular/kklkvybnydr?file=src%2Findex.html) 一邊看我的解釋應該會比較好看懂。

明天開始會進入到 Angualr 中非常重要的一個觀念，`Dependency injection`，可能在前面幾篇中多多少少都有提到一點關於他的內容，不過沒關係之後會詳細的講解他到底是什麼，那我們就明天見吧！


# Reference

- [Angular.io - Dynamic component loader](https://angular.io/guide/dynamic-component-loader)
