---
title: You Don't Know JavaScript [Async & Performance] - Promises
date: 2021-01-20 09:54:58
tags:
- Javascript
- Front-end
- Async & Performance

categories:
- You Don't Know JavaScript
---

在上一張中我們介紹了使用callback function的目的與缺點，雖然可以幫助我們處理非同步問題，但缺乏`順序性`與`可靠性`，這時人們想著：能不能有一個"在當前任務完成後才要發生的事"，不會造成順序變化？並且可能返回這個非同步任務是成功還是失敗。

於是乎Promise就這麼誕生了，而`Promise代表為一個容器，保存著某個未來才會結束的事件(非同步)的結果`，它他可以像同步事件一樣返回結果，`可以返回以解決的值或是未解決的原因`並且無法更改結果。

這看起來似乎是個美妙的東西，不僅可以解決頭痛的非同步問題還可以避免掉callback function的缺點，讓我們仔細的介紹一下這個神奇的方法。

<img src="https://miro.medium.com/max/1400/1*ZqwKR-S8PUH8samiTIJ93g.jpeg" alt="drawing" width="860"/>

<!-- more -->

# What is Promise?
> Promise是一個存放著非同步事件的結果，可以回傳成功的結果或未成功的原因。

這句話雖然可以解釋Promise是什麼，但是可能對於剛接觸或是對JS不熟的朋友們還是有點難度，讓我換一個說法：

想像一下你今天去麥當勞點了一個大麥克、勁辣雞腿堡、蕈菇安格斯黑牛堡再加點四塊麥克雞塊，漢堡都搭配套餐薯條可樂(太多了...)，雖然店員會一臉疑惑地覺得你真的吃得下嗎，但是他還是會給你一個號碼牌，而這個號碼牌就代表著JavaScript的Promise。

你可以那著這個號碼牌到旁邊一邊滑手機一邊等餐，而這就是非同步事件，而你在滑手機的時候不會擔心你的餐點不見，因為你拿到了你的號碼牌（當然還是因為你付錢了...），`這就是一個Promise一個承諾，承諾會給你一個結果`。

但是也有另一種可能，當櫃檯叫到你的號碼時你滿心期待的到櫃台想拿到你的餐點，但是他們跟你說由於剛剛櫃檯不知道廚房備料的狀況，所以你的餐點賣光了無法給你並把錢退還給你，這就是Promise的另一個重點，如果失敗了他會告訴你`他失敗了`並把失敗的原因告訴你不會讓你在旁邊癡癡的等，這就是Promise的概念。

# How Promises Work?
Promise是一個Object他會有三種狀況：
1. Pending: 成功或失敗前的初始狀態。
2. Resolved: 事件成功。
3. Rejected: 事件失敗。

![https://ithelp.ithome.com.tw/upload/images/20210119/20124767VvxXize7eF.png](https://ithelp.ithome.com.tw/upload/images/20210119/20124767VvxXize7eF.png)
（圖片來源：[JavaScript Promise Tutorial: Resolve, Reject, and Chaining in JS and ES6](https://www.freecodecamp.org/news/javascript-es6-promises-for-beginners-resolve-reject-and-chaining-explained/)）

舉個例子，當我們使用Promise向伺服器請求後端數據，他會先進到Pending狀態，因為還在處理尚未得到數據或是失敗，而這個狀態持續到我們得到結果為止。

如果我們成功獲得後端的數據，則Promise從Pending變為Resolved代表事件成功，相反的如果沒有獲得數據則從Pending變為Rejected代表事件失敗，無論是事件完成或是失敗都會在回傳一個Promise讓我們鏈接新的事件在後面。

> 一旦Promise的狀態發生改變就無法更改，Promise結果的不變性是非常重要的。

# Error Handling
Promise有自己的錯誤處理機制，這對我們編寫非同步程序來說是非常重要的，由於非同步事件的特性會讓我們對非同步事件的結果不好掌握常常會造成不可與其的錯誤，需要加上許多判斷式來確保我們的非同步事件的確定性，但Promise可以很有效的幫我們解決這個問題。

then可以接收兩個callback function參數，一個是代表Promise成功反之代表失敗。
```javascript
let promise = Promise.reject(new Error('This is reject'));

function resolved(result){
    console.log('Resolved');
}
function rejected(err){
    console.log(err);
}

promise.then(resolved, rejected); // This is reject
```

雖然上面這種寫法可以在我們promise失敗時捕獲到錯誤，但是如果當promise成功進到resolved後卻在resolved中發生錯誤，而這個錯誤將會被忽略。
![https://ithelp.ithome.com.tw/upload/images/20210119/20124767DSQFhGM21h.png](https://ithelp.ithome.com.tw/upload/images/20210119/20124767DSQFhGM21h.png)
（圖片來源：[Master the JavaScript Interview: What is a Promise?](https://medium.com/javascript-scene/master-the-javascript-interview-what-is-a-promise-27fc71e77261#:~:text=A%20promise%20is%20an%20object,%2C%20a%20network%20error%20occurred.&text=Promise%20users%20can%20attach%20callbacks,or%20the%20reason%20for%20rejection.)）

於是Promise提供了另一種寫法。
```javascript
let promise = Promise.reject(new Error('This is reject'));

function resolved(result){
    console.log('Resolved');
}
function rejected(err){
    console.log(err);
}

promise.then(resolved).catch(rejected); // this is resolve
```
這種寫法可以使catch捕獲到promise或resolved的錯誤。
![https://ithelp.ithome.com.tw/upload/images/20210119/20124767GzFH9lYURQ.png](https://ithelp.ithome.com.tw/upload/images/20210119/20124767GzFH9lYURQ.png)
（圖片來源：[Master the JavaScript Interview: What is a Promise?](https://medium.com/javascript-scene/master-the-javascript-interview-what-is-a-promise-27fc71e77261#:~:text=A%20promise%20is%20an%20object,%2C%20a%20network%20error%20occurred.&text=Promise%20users%20can%20attach%20callbacks,or%20the%20reason%20for%20rejection.)）


# Terminology: Resolve, Fulfill, and Reject
在更深入的了解Promise之前，我們需要先了解resolve、fulfill和reject之間的區別。

- Promise.resolve：會返回一個給定值的Promise Objcet，他一定會使Promise狀態從pending變為Resolved並可以使用then獲取得訂得值。
```javascript
let promise = Promise.resolve('this is resolve');

promise.then(val => console.log(val)); // this is resolve
```

- Promise.reject：會返回一個Promise Objcet，他一定會使Promise狀態從pending變為Rejected並可以使用catch獲得錯誤內容。
```javascript
let promise = Promise.reject(new Error('This is reject'));

promise.catch(err => console.log(err)); // This is reject

function resolved(result){
    console.log('Resolved');
}
function rejected(err){
    console.log(err);
}

promise.then(resolved, rejected); // This is reject
```

# Refarence
- [You Don't Know JavaScript](https://github.com/getify/You-Dont-Know-JS/blob/1st-ed/async%20&%20performance/ch3.md)
- [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)
- [Master the JavaScript Interview: What is a Promise?](https://medium.com/javascript-scene/master-the-javascript-interview-what-is-a-promise-27fc71e77261#:~:text=A%20promise%20is%20an%20object,%2C%20a%20network%20error%20occurred.&text=Promise%20users%20can%20attach%20callbacks,or%20the%20reason%20for%20rejection.)