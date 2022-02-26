---
title: You Don't Know JavaScript [Async & Performance] - Now & Later
date: 2021-01-12 17:07:24
tags:
- Javascript
- Front-end
- Async & Performance

categories:
- You Don't Know JavaScript
---

由於JavaScript是一個單線程的程式語言，這意味著JavaScript一次只能做一件事，雖然只有一個執行緒可以簡化程式不用擔心並發與衝突的問題，但是這代表JS引擎不能執行長時間的操作，如果一個函數依賴於另一個函數的結果，那麼就必須等待上一個函數結束後才能進行，因此他會將整個主執行緒阻塞導致網頁沒有反應從而降低使用者體驗。

在電腦能夠擁有多個處理器的時代，可以將其他任務放到另一個處理器上處理並讓使用者知道何時會完成這樣比坐在那等卻什麼都沒有還來的有意義並可以同時完成其他工作，而這就是Asynchronous的用途了，使用Asynchronous（callback function、promise、async/await...）可以達到執行長時間的網路請求的同時又不會阻塞主執行緒。

<img src="https://res.cloudinary.com/practicaldev/image/fetch/s--IB0Ikc71--/c_imagga_scale,f_auto,fl_progressive,h_900,q_auto,w_1600/https://cl.ly/3N0P302P0H2g/Image%25202018-07-19%2520at%25209.16.55%2520AM.png" alt="drawing" width="860"/>

<!-- more -->

# How Does Synchronous JavaScript Work?
在了解非同步JavaScript之前，我們要先了解在JavaScript中是如何運行同步的程式，舉個例子：
```javascript
const second = () => {
  console.log('Hello there!');
}
const first = () => {
  console.log('Hi there!');
  second();
  console.log('The End');
}
first(); // Hi there! -> Hello there! -> The End
```
要了解JavaScript引擎是如何執行上面的程式，我們需要了解`Execution Context`與`call stack`的概念。

## Execution Context (EC)
在JavaScript中EC是一個抽象的概念，包含著有關在其中執行當前程式碼的環境信息，這句話有點深奧，換句話說當JavaScript運行你的程式碼的時候，他會建立一個全域的EC，它包含著這個程式中全域的值、變量、物件、函數，而這些可以被訪問的東西就稱為環境，而每次運行都只會有一個EC的存在，在JS的Execution Context中主要分為三種類型：
1. Global execution context (GEC): 默認的EC會在`首次加載到瀏覽器時`會加入，所有的全域程式都會在這裡執行。
2. Functional execution context (FEC): 會在JS引擎發現函數時創建的EC，`每個函數都有自己的EC`用來包含這個函數中所使用到的變量、物件等等，而FEC可以訪問到GEC的程式，而反之亦然。
3. Eval: eval函數執行的EC。

## Call Stack
在資料結構中有一個重要的概念LIFO(Last in, First out)，而JS中的call stack也是這種資料結構的模式，讓我們舉個例子：
```javascript
const second = () => {
  console.log('Hello there!');
}
const first = () => {
  console.log('Hi there!');
  second();
  console.log('The End');
}
first();
```
![https://ithelp.ithome.com.tw/upload/images/20210112/20124767voyVOLdwYP.png](https://ithelp.ithome.com.tw/upload/images/20210112/20124767voyVOLdwYP.png)
(圖片來源：[Understanding Asynchronous JavaScript](https://blog.bitsrc.io/understanding-asynchronous-javascript-the-event-loop-74cd408419ff))
- 當執行了這個程式時JS引擎會先創造一個GEC （由上圖main()表示）並送入call stack中，而當JS引擎發現了first這個函式時，他會為這個函數建立一個FEC並將他放在call stack中。

- 接下來當執行完console.log('Hi there!')之後將他pop出call stack，之後調用second()，而JS引擎也會為了second建立一個FEC並放在call stack中。

- 一樣的當second中的console.log('Hello there!!')完成後pop出call stack，JS引擎知道second已經執行完畢便將他也pop出stack。

- 最後將console.log('The end')放入call stack中執行，完成後一樣pop出stack，這時first也執行完畢被pop出stack。

- 當這一系列的操作完成後JS引擎確定沒有其他需要執行的程式，便將GEC也pop出call stack代表整個程式完成運行。

# Event Loop
在我們討論到什麼事Event loop之前我們需要了解到為什麼我們需要他，就像我們在前言中提到的，JS是單線程的程式語言，所以當JS做了一件長時間處理的事情時，會造成其他動作被卡住就為了等他完成，這個現象就是所謂的`Blocking`，所以為了解決這個問題所以JS才提供了asynchronous這個方法。

```javascript
const network = () => {
  setTimeout(() => {
    console.log('Async Code');
  }, 2000);
  console.log('Hello World');
};

network();
```
當我們使用setTimeout這個函數，他會將我們的call back function在2秒後再呼叫，所以會先輸出`Hello World`後在輸出`Async Code`，對於新手來說可能會覺得疑惑，應該是要過兩秒後輸出Async Code後在輸出Hello World才對啊！接下來我們就要深入介紹到底發生什麼事。

## Event Loop
我們先來看一個經典的圖：
![](https://ithelp.ithome.com.tw/upload/images/20210112/20124767c2mfzFrI4K.png)
對於JS來說雖然他本身只有單執行緒，但是由於JS是在瀏覽器中運行所以能夠使用到瀏覽器的WebAPI，接下來我們來看看對於非同步運行到底發生了什麼事，

![](https://i.imgur.com/e8S3FCJ.gif)
(圖片來源：[所以說event loop到底是什麼玩意兒？| Philip Roberts | JSConf EU](https://www.youtube.com/watch?v=8aGhZQkoFbQ))

上面的影片可以充分的解釋JS的非同步事件到底發生了什麼事，當JS遇到了非同步事件時，他會將這個事件暫時放在webAPIs中處理，讓JS的其他同步事件可以持續進行，而當在webAPIs中處理的事情完成後會將他放在task queue等待call stack中的同步事件完成後，才會透過`event loop`將這個非同步事件的結果放回call stack中。

由於queue是FIFO(First in, First out)的資料結構，所以如果有多個非同步事件處理完成，會在task queue中照著完成的進度排隊，依序的透過event loop放回call stack中，這就是event loop的概念。

# Parallel Threading
許多人會將`parallel`與`async`搞混，但是他們是完全不同的概念讓我們來仔細將他們釐清。

## Parallel
現在多處理器非常的流行，而多處理器代表機器在同一時間內擁有處理多個事情的能力。
![https://ithelp.ithome.com.tw/upload/images/20210112/20124767lAS0ISYHHC.png](https://ithelp.ithome.com.tw/upload/images/20210112/20124767lAS0ISYHHC.png)
（圖片來源：[Asynchronous and Parallel Programming](https://medium.com/@letienthanh0212/asynchronous-and-parallel-programming-in-c-net-1e0f14e1db80)）

## Asynchronous
非同步處理消除了同步處理會造成因為單一線程處理而卡住UI的缺點，可以使用後台運行耗費大量時間的操作，但是對於主要架構來說他依然是只有一個線程，只是將耗時任務委託給後台（其他API）處理 （event loop）。
![https://ithelp.ithome.com.tw/upload/images/20210112/20124767tTXp4wrkW5.png](https://ithelp.ithome.com.tw/upload/images/20210112/20124767tTXp4wrkW5.png)
圖片來源：[Asynchronous and Parallel Programming](https://medium.com/@letienthanh0212/asynchronous-and-parallel-programming-in-c-net-1e0f14e1db80)）

了解了這兩種編譯方式的不同後，我們要來介紹對於非同步操作的一些要注意的部分。

```javascript
var a = 20;

function foo() {
	a = a + 1;
}

function bar() {
	a = a * 2;
}

// ajax(..) is some arbitrary Ajax function given by a library
ajax( "http://some.url.1", foo );
ajax( "http://some.url.2", bar );
```
由於上面的程式對於函數的呼叫都是使用非同步的方式，所以這種情況可能會有兩種不同的輸出結果，因為對於非同步事件來說都會委託給webAPI進行處理，當處理完成後會將結果放到task queue中等待call stacl任務完成才會通過event loop傳給call stack。

但是由於兩個非同步的處理完成的時間不一樣，先完成的會先被放在task queue中而後完成的則會排在它後面，所以
當foo先完成就會先將a+1所以會輸出42，而若bar先完成則會先將a*2輸出41，所以當我們使用非同步呼叫函數時，對於同一個變量的操作會因為先後完成順序的不同而導致有不同的結果有可能會造成非預期的錯誤，所以在使用非同步呼叫時要額外的小心。

# 結論
由上面的介紹可以對JavaScript非同步的操作多一點概念，下面我們將整理一下本章的重點：
- JavaScript是單線程語言，這意味著一次只能處理一件事。
- 非同步是用來解決同步會發生的blocking問題。
- Execution Context代表執行當前程式碼的環境信息，通常有三種類型：
    1. Global execution context (GEC): 默認的EC會在`首次加載到瀏覽器時`會加入，所有的全域程式都會在這裡執行。
    2. Functional execution context (FEC): 會在JS引擎發現函數時創建的EC，`每個函數都有自己的EC`用來包含這個函數中所使用到的變量、物件等等，而FEC可以訪問到GEC的程式，而反之亦然。
    3. Eval: eval函數執行的EC。 
- call stack代表程式運行的流程
- event loop是JS非同步操作背後的原理
- parallel programming代表可以同時處理不同的事
- asynchronous programming可以將耗時的事交給背景處理，不會造成UI卡住
- 多個非同步呼叫同一個變數需要小心，`不同順序完成會有不同的結果`。



# Reference
- [You Don't Know JavaScript](https://medium.com/@happymishra66/execution-context-in-javascript-319dd72e8e2c)
- [ECMAScript® 2020 Language Specification](https://www.ecma-international.org/ecma-262/#sec-execution-contexts)
- [Execution context, Scope chain and JavaScript internals](https://medium.com/@happymishra66/execution-context-in-javascript-319dd72e8e2c)
- [Understanding Asynchronous JavaScript](https://blog.bitsrc.io/understanding-asynchronous-javascript-the-event-loop-74cd408419ff)
- [所以說event loop到底是什麼玩意兒？| Philip Roberts | JSConf EU](https://www.youtube.com/watch?v=8aGhZQkoFbQ)
- [Asynchronous and Parallel Programming](https://medium.com/@letienthanh0212/asynchronous-and-parallel-programming-in-c-net-1e0f14e1db80)
