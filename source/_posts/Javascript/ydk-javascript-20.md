---
title: You Don't Know JavaScript [Async & Performance] - callbacks
date: 2021-01-18 14:16:59
tags:
- Javascript
- Front-end
- Async & Performance

categories:
- You Don't Know JavaScript
---

在JavaScript中Callback是最基礎的也使最常用來處理非同步的方式，而callback是什麼？`簡單來說就是在另一個函數完成執行之後執行就稱為callback`，而複雜來說由於JavaScript中的函數屬於一級成員，代表著函數可以作為參數傳遞並且可以由其他函數返回，而`作為參數傳遞的任何函數都稱為callback function`，使用callback function雖然可以解決非同步的問題，但是他同時也有許多缺點，本章終將仔細地介紹callback function的缺點。

<img src="https://miro.medium.com/max/1400/1*58vcVucp3Jn6HCnLalYlwQ.png" alt="drawing" width="860"/>

<!-- more -->

# Why do we need Callback Functions?
在介紹callback的缺點之前，讓我們再複習一次倒底JavaScript為什麼會需要一個callback function，由於JavaScript是個由上而下運行的程式但是某些情況下會需要使用非同步的方式運行，而`callback function可以確保他在我們指定的任務完成之前不會被運行，但當我們指定的任務完成時他會立刻運行`，這使我們在開發非同步程式的時候能夠確保我們的流程順序不會有錯誤。

讓我們舉個例子，當我們開發了一個專案，我們需要使用ajax獲得後端的數據再將它顯示出來，我們需要保證我們確實拿到了後端的資料（ajax已經完成）後才將它顯示出來不然會發生錯誤，這樣的情況下使用callback function就能夠確保我們的時間順序，不會在ajax任務尚未完成之前就將data顯示。
```javascript
ajax("https:ajax.com", function(data){
    console.log(data);
})
```

# What is callback funciton Disadvantages?
我們了解了callback function最初被設計出來是為了要解決什麼問題，他確實能夠在我們處理非同步事件的時候起到一的很好的作用，但是他確實也有需多缺點，讓我們一一介紹他有哪一些缺點：

## Order problem
當我們在閱讀別人的程式的時候，由於我們習慣由上而下的閱讀所以對於非同步的程式來說我們會看得稍微有點吃力，讓我們舉個例子：
```javascript
// a...
setTimeout(function(){
    // c...
},1000);
// b
```
當我們要將上面這個程式解釋給一個JS新手時，我們該如何解釋他？
- 可能會有人說：先做a，設置一個等待一秒的定時器，一秒過後執行c。
- 也可能會說：先做a，設置一個等待一秒的定時再去做b，一秒過後再回來做c。

雖然第二種說法比較正確，但是說實在的如果我是一個JS新手估計我會完全不知道你在說什麼，為什麼程式執行到下面後又跳回上面？這樣的問題出現在許多開發人員心中。

你可能覺得：這樣就不行了也太弱了吧...那是因為我們只介紹了簡單的callback問題，讓我們再來一個更難的：
```javascript
doA(function(){
    doB();
    
    doC(function(){
        doD();
    })
    
    doE();
});
doF();
```
看到上面的程式估計已經暈的，如果是一個新手來看這個程式我想他已經在放棄的路上了，這個程式的真實發生順序：`doA -> doF() -> doB() -> doC() -> doE() -> doD()`。

當我們在追蹤這種非同步事件的程式時，常常會因為使用callback function導致我們我需要再一個程式中不斷上上下下的去看執行的順序，這對於日後維護或是其他人來看都會是一個不便的事情。
![https://ithelp.ithome.com.tw/upload/images/20210118/201247677dkFwCQn1U.png](https://ithelp.ithome.com.tw/upload/images/20210118/201247677dkFwCQn1U.png)

## Callback Hell
使用callback function除了會讓閱讀程式因為順序問題不斷上上下下增加閱讀難度之外，還有一個在JavaScript中流傳已久的callback hell，讓我們來舉個例子實際體驗一下地獄：
```javascript
listen('click', function handler(event){
    setTimeout(function request(){
        ajax('https://some.url.1'm function responde(test){
            if(test === 'hello'){
                handler();
            } else {
                request();
            }
        })
    })
})
```
由於JS中函數屬於一級公民可以作為參數傳遞，所以就會有這種多重嵌套的問題，我們來分析一下上面程式：
1. 首先我們先註冊一個監聽事件
```javascript
listen('click', function handlet(event){
    // ...
})
```
2. 當發生監聽事件時觸發計時器在500ms之後觸發callback function
```javascript
setTimeout(function request(){
    // ...
})
```
3. 計時結束後觸發callback function ajax進行數據請求
```javascript
ajax('https://some.url.1'm function responde(test){
    // ...
})
```
4. 當獲得ajax數據後再對數據進行判斷
```javascript
if(test === 'hello'){
    handler();
} else {
    request();
}
```
當看完這些程式邏輯後估計已經頭暈了，不過這只是簡單的三個嵌套，還有更恐怖的...
![https://ithelp.ithome.com.tw/upload/images/20210118/20124767CQmRdUH1mA.png](https://ithelp.ithome.com.tw/upload/images/20210118/20124767CQmRdUH1mA.png)


# 結論
callback function是JS中處理非同步的基礎，但是隨著JS的成熟與更加龐大的需求它們對於非同步編程的演化趨勢來講顯得不夠。

我們對於閱讀程式會習慣由上到下的同步化、順序化閱讀而如果使用callback function會因為執行順序的問題導致程式上上下下的執行，對後續的維護與閱讀造成很大的困擾，並且若嵌套多層callback function則這個問題更加明顯。


# Reference
- [You Don't Know JavaScript](https://github.com/getify/You-Dont-Know-JS/blob/1st-ed/async%20&%20performance/ch2.md)
- [JavaScript: What the heck is a Callback?](https://codeburst.io/javascript-what-the-heck-is-a-callback-aba4da2deced#:~:text=Simply%20put%3A%20A%20callback%20is,In%20JavaScript%2C%20functions%20are%20objects.&text=Functions%20that%20do%20this%20are%20called%20higher%2Dorder%20functions.)
- [JavaScript Callback Functions – What are Callbacks in JS and How to Use Them](https://www.freecodecamp.org/news/javascript-callback-functions-what-are-callbacks-in-js-and-how-to-use-them/)