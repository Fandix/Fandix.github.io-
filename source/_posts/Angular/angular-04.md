---
title: Day4. Component
date: 2021-09-04 13:25:45
tags:
- Angular
- Front-end

categories:
- 2021 鐵人賽
---

講完前面幾張比較偏向原理與不常使用到的章節後，從這章節開始會介紹比較實際運用的東西。

就像在 Day1 所提到的，`Angular app 是由無數個大大小小的 Component 所組合而成的`，也可以說 Angular app 的基本單位就是 Component，所以 Component 在 Angular 中是非常重要的觀念，每個 Component 都包含著：

- 一個 HTML Template，用於將 Component 的內容顯示在 UI 上。
- 一個 TypeScript 的 Class
- 一個 selector，用於定義這個 Component 如何在別的 Component 中被使用。
- 一個 CSS style 的檔案，用於決定這個 Component 在 UI 上呈現的樣式。

![https://ithelp.ithome.com.tw/upload/images/20210821/20124767ow9wdaimBy.png](https://ithelp.ithome.com.tw/upload/images/20210821/20124767ow9wdaimBy.png)

<!--more-->

# Create a component
在介紹什麼是 Component 之前，最重要的就是先創建一個 Component 出來，最簡單的方法就是使用 Angular CLI 來自動創建，當然你也可以手動新增檔案然後把所有需要的訊息補上

要自動化創造一個新的 Component，首先要在終端機中輸入 Angular CLI Command
```bash
ng generate component <component-name>
ng g c <component-name>
```

在默認情況下，會自動創建出：

- 以 component-name 所命名的資料夾
- 一個 component file,  <component-name>.component.ts
- 一個 HTML Template file,  <component-name>.component.html
- 一個 CSS file,  <component-name>.component.css
- 一個測試規範文件,  <component-name>.component.spec.ts

![https://ithelp.ithome.com.tw/upload/images/20210730/20124767XlBg9RHWON.png](https://ithelp.ithome.com.tw/upload/images/20210730/20124767XlBg9RHWON.png)
![https://ithelp.ithome.com.tw/upload/images/20210730/20124767c8SVZdpzx9.png](https://ithelp.ithome.com.tw/upload/images/20210730/20124767c8SVZdpzx9.png)


# Specifying a component's CSS selector
每一個 Component 都需要一個 CSS Selector，這個 selector 會讓 Angular 知道他應該要在哪裡找到訊息並將他實例化，簡單來說可以把它當成這個 Component 在 Angular 的`身份證號碼`，當 Angular 在 HTML 中發現這個身份證號碼後，他就會知道該去哪裡把這個 Component 給實例化。

舉個例子，如果今天有一個 component 叫做 hello-world.component.ts，他的 selector 叫 app-hello-world，所以每當 Angular 在任何一個 HTML Template 中出現時，他就知道要去把 hello-world.component.ts 這個 Component 給實例化。

```typescript
@Component({
  selector: 'app-hello-world',
})
```


# Defining a component's template
在了解 Angular 是如何透過 selector 找到對應的 Component 並將他實例化後，接下來介紹的是

Template，他其實就是一個 HTML，它告訴 Angular 要把這個 Component 如何呈現在 UI 上，可以通過兩種方法定義 Template，第一種是引用外部文件，第二種是直接將你的 HTML 寫在 Component 中。
 
- 引用外部文件
``` typescript
@Component({
  selector: 'app-hello-world',
  templateUrl: './hello-world.component.html',
})
```
- 直接寫 HTML 在 Component 中
```typescript
@Component({
  selector: 'app-hello-world',
  template: '<h1>Hello World!</h1>',
})
```
除了使用 ( ' ' ) 包裹著 HTML Tag 之外，也可以使用反引號 ( ` ) 一次輸入多個 Tag
```typescript
@Component({
  selector: 'app-hello-world',
  template: `
	  <h1>Hello World!</h1>
	  <p>This template definition spans multiple lines.</p>
	`,
})
```

兩種方法都可以設定你的 HTML Template，但是要注意，如果使用`外部文件的話是使用 templateUrl`，而`直接填入 HTML Tag 則是使用 template`。



# Declaring a component's styles
有了身份證號碼有了畫面接下來就要讓畫面多一點美感，這時候就需要使用 CSS 來美化我們的 UI，和 Template 一樣可以透過兩種方式聲明 CSS，第一是引用外部文件，第二是直接寫在 Component 中。

- 引用外部文件
```typescript
@Component({
  selector: 'app-hello-world',
  templateUrl: './hello-world.component.html',
  styleUrls: [ './hello-world.component.css' ]
})
```
- 直接寫在 Component 中
```typescript
@Component({
  selector: 'app-hello-world',
  template: '<h1>Hello World!</h1>',
	styles: [ 'h1 { font-weight: normal; }' ]
})
```
Style 與 Template 一樣，如果是`引用外部文件是使用 styleUrls`，而`直接寫在 Component 中是使用styles`。


# 結論
本章節比較輕鬆一點，介紹了如何創建一個新的 Component，並瞭解一個 component 是由哪些元素組成，並且這些元素是代表什麼，下一篇我們會介紹 Component 的 Lifecycle，是在開發 Angular App 中非常常使用到的觀念，那我們就下篇再見吧。


# Reference
- [Angular.io - Component-overview](https://angular.io/guide/component-overview)
