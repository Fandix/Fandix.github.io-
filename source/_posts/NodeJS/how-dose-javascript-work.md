---
title: NodeJS - How does javascript work
date: 2022-02-25 20:50:22
tags:
- Node JS

categories:
- Node JS
---

如果你今天有一台新電腦，有人跟你說: 『嘿！ 我這裡有一份 Javascript 的 Code 你能你能讓他跑起來嗎？』，你會怎麼回答？仔細想想一份 Javascript 該如何在任何一台電腦甚至主機上運行。

有些人第一個反應會是，`啊我可以在『瀏覽器』中執行 Javascript` ，打開瀏覽器並開啟開發者模式，接著你可以在 console 中執行 Javascript 程式對吧？

不過雖然可以在這裡執行 Javascript 程式，但是這並不是在你的主機中執行，你可能會覺得不對啊，不就是在我的主機上執行的嗎？但實際上你要做到這個操作你得先在你的主機中先安裝 `瀏覽器` 才可以，如果今天拿到的是一台空白的新電腦，那麼這個方法便不奏效，那麼 Javascript 是如何在瀏覽器中執行的呢，讓我們先從頭介紹一下 Javascript。

<img src="https://miro.medium.com/max/1400/1*LyZcwuLWv2FArOumCxobpA.png" alt="drawing" width="860"/>

<!-- more -->

# How Javascript run in ****browser ?****

當你有一個 Javascript source code 要怎麼讓電腦知道這份 code 是什麼呢？ 這就跟你今天到了法國的一家餐廳，你需要跟服務生說你需要什麼餐點，那麼你該如何將你的語言轉換成讓法國的服務生聽懂的語言呢，這時就需要翻譯軟體了對吧，在電腦中 Javascript 也是需要經過翻譯軟體轉換後才能讓電腦知道在寫什麼東西，而這個轉換的軟體稱為 `Javascript engine` 。

<img src="https://ithelp.ithome.com.tw/upload/images/20220225/20124767Q3WSkDewPw.png" alt="drawing" width="800"/>

## The Anatomy of the JavaScript engine

每個瀏覽器中都提供了一個 Javascript engine，比如 Google Chrome 使用的 V8 engine，Firefox 使用的 spiderMonkey 或 Safari 使用的 JavaScriptCore 等等，不過本章會著重介紹 google 的 V8 engine。

最一開始 JavaScript engine 的工作是獲取 JavaScript Source code 並將其編譯為 CPU 可以理解的二進制指令，JavaScript engine 包含一個 `baseline compiler` ，其工作是將 JavaScript Source code 編譯成 Intermediate representation（IR），也稱為 bytecode ，並將這個 bytecode 提供給 interpreter，interpreter 獲取此 bytecode 並將其轉換為可以在 CPU 上運行的機器碼。

<img src="https://miro.medium.com/max/1400/1*JL_8RT-vEMDs5OKvOjWOzA.png" alt="drawing" width="800"/>

baseline compiler 的工作是盡可能快速的將 Javascript source code 轉換成 bytecode 或 machine code，但是在轉換輸出的過程中並不會把 sourece code 進行優化，所以 interpreter 會使用到未優化的 bytecode 或 machine code 從而導致應用程式的速度非常慢，對於需要高度交互的 Web 應用程序來說使用者體驗會非常差，所以 google 使用了 V8 JavaScript engine 來解決這個問題。

### Early google Javascript V8 engine

2010 年的 V8 engine 中為了提高 Javascript 效能在 JavaScript engine 中多添加了 `full-codegen` 和 `crankshaft compiler` 這兩個部分。

<img src="https://miro.medium.com/max/1400/1*c7XEoH24fIPZ3jO2vloYjQ.png" alt="drawing" width="800"/>

full-codegen 是 baseline compiler，其工作是盡可能快速的將 Javascript source code 轉換成未優化的 bytecode 或 machine code 以達到加快應用程序的啟動 (application bootstrap)，接著當應用程式運行時 crankshaft compiler 將啟動並優化 baseline compiler 

為了快速啟動應用程式而轉換的尚未優化的 bytecode 或 machine code，隨著應用程式的使用 crankshaft compiler 會不斷的持續優化 bytecode 或 machine code 給 CPU 使用，這樣的過程會讓應用程式的效能變的更好，這種操作方式稱為 [`JIT (Just-In-Time)`](https://en.wikipedia.org/wiki/Just-in-time_compilation) 。

### How JavaScript is optimized ?

要優化轉換 Javascript source code 有多種標準，比如可以在 Javascript source code 傳遞給 baseline compiler 之前先把他解析為 [`Abstract Syntax Tree (AST)`](https://en.wikipedia.org/wiki/Abstract_syntax_tree) ，簡單來說就是先把程式轉換為樹狀結構，這樣對於 baseline compiler 來說可以 `識別需要立即解析的內容` 並只要先生成 `必須` 的內容即可，這樣可以有效地加快應用程式的啟動(application boostrap)，當然還有其他優化的方式這邊就不一一介紹，有興趣可以參考 [The V8 Engine and JavaScript Optimization Tips](https://www.digitalocean.com/community/tutorials/js-v8-engine) 這篇文章。

### Recent google Javascript V8 engine

在 2017 年 Google 團隊創建了新版本的 V8 engine，他們將 V8 engine 引入了一個新的 interpreter pipeline 稱為 `Ignition`，它的作用是使用 baseline compiler 轉換 Javascript source code 生成 bytecode 或 machine code，然後使用 interpret 解析轉換出來的 bytecode 或 machine code。

TurboFan 是一個 `優化編譯器 (optimization compiler)` ，可以在應用程式運行時在後台 (單獨的線程)中不斷優化 bytecode 或 machine code，這些被非常優化的 bytecode 或 machine code 會把 Ignition 產生的未優化的程式替換掉。

<img src="https://miro.medium.com/max/1400/1*5XSWl0QXq-5ObZ6sF_aHEQ.png" alt="drawing" width="800"/>

# JavaScript at runtime
Javascript 是個 `single-threaded langauge` 簡單來說就是 Javascript 一次只會做一件事，所以只要有一個動作要做特別久，那其他的動作都會阻塞著等到那個動作完成後才會執行，所以你可能會看到

<img src="https://miro.medium.com/max/924/0*w2rEwv9mE9xVPhRy.png" alt="drawing" width="700"/>

不信你可以在瀏覽器的 console 中輸入這個看看
```javascript
while (1) {}
```

接著為了瞭解 Javascript 是如何執行程序，舉個簡單了例子
```javascript
function baz() {
  console.log( 'Hello from baz' );
}

function bar() {
  baz();
}

function foo() {
  bar();
}

foo();
```

Javascript 和其他程式語言一樣，運行時會有一個 `stack` 和 `heap storage` ，stack 是 `LIFO (後進先出)` 的數據儲存，用於儲存程序當前函數的 execution context，所以以上面的例子來說函數 `foo()` 會第一個被放進 stack 裏面，接著是 `bar()` 再來是 `baz()` 最後才是 `console.log('Hello from baz')` ，但執行的話遵照後進先出的原則 `console.log('Hello from baz')` 會第一個被執行以此類推。

<img src="https://miro.medium.com/max/700/1*rRoLpv-Zrmpa-srNhwlbvA.gif" alt="drawing" width="860"/>

在 Stack 中的每個任務稱為 `stack frame`，它包含了調用該函數所需要的一切信息，比如參數, 局部變量, return 的地址等等。

上面提到 Javascript 是 `Single-threded` ，所以只會有一個 Heap 和 Stack，意思就是說其他程序要執行前必須要等待上一個任務執行完畢後才能接著執行，簡單來說就是排隊的概念，通常稱這條單一個 stack 為 `main thred` 或 `main excution thred` 。

那麼照上面的邏輯來說，當瀏覽器利用 Http 請求一個網路資源的時候，畫面應該會卡住不動直到瀏覽器拿到這個網路資源吧？答案是否定的，如果這樣的話使用者體驗未免也太差了吧！

上面有提到瀏覽器中會有一個 Javascript engine 負責執行 Web 中所包含的所有 Javascript，他大概會長這樣

<img src="https://miro.medium.com/max/1400/1*ocCc8yEvUEeOtGU2LNQnpA.png" alt="drawing" width="860"/>

這時候就需要介紹 `Event Loop` 與 `Callback queue` 是個什麼了。

當瀏覽器執行一個非同步事件，比如說 Http 請求或是 Javascript 的 `setTimeout` 或 `setInterval` 對於瀏覽器來說他會將這些非同步事件放在稱為 `WebAPIs` 的地方執行，而原本的 Stack 就繼續執行同步的任務，而當在 `WebAPIs` 中執行的非同步事件處理完畢後，會將結果暫時放在 `callback queue` 裏面， `queue` 和 stack 一樣是個數據儲存結構，但 stack 不一樣的是 queue 是 `FIFO` 代表先進先出。

話說回來當 `WebAPIs` 的非同步任務完成後會將完成的任務放在 `Callback queue` 裏面，而當瀏覽器注意到 Stack 裏面的同步任務都完成了後，便會利用 `Event Loop` 將 callback queue 裏面完成的非同步任務放到 Stack 裏面，接著他就會跟一般的同步任務一樣被執行。

<img src="https://miro.medium.com/max/1400/1*9mv-g9E-87Sji9j7YR08Fw.gif" alt="drawing" width="860"/>

上面介紹了這麼多 Javascript 是如何在瀏覽器中執行，那麼把問題拉回到一開始，「你可以讓 Javascript 的 code 在你的電腦上跑起來嗎？」

# What is NodeJS ?
除了在瀏覽器中可以執行 Javascript 之外，另一個方法便是使用 NodeJS，我們先來看看維基百科是怎麼介紹 NodeJS
> Node.js 是一個 `open-source`、`cross-platform` 的後端 `JavaScript runtime`，它在 V8 engine 上運行並在 Web 瀏覽器之外執行 JavaScript 程式。

可以從上面這句話明顯地看到，NodeJS 是一個 Javascript 的 runtime，他讓你可以在瀏覽器之外的地方執行 Javascript，要介紹 NodeJS 之前，首先我們先介紹一個重要的東西 `libuv`。

## What is libuv?
從 [libuv](http://docs.libuv.org/en/v1.x/) 的官網可以看到 libuv 是個什麼東西
> libuv 是一個專注於 asynchronous I/O 的多平台 library，它主要是為 Node.js 開發的

上面介紹了由於 Javascript 是 Single-threded 所以如果在網頁中執行了需要耗費多時的任務時，會將他移到由瀏覽器提供的 `WebAPIs` 執行從而不影響 V8 engine 的 stack 中同步任務的進行，那麼既然 NodeJS 是能夠在瀏覽器外執行 Javascript 的東西，那麼理所當然的他也會需要一個他自己的 `WebAPIs` 用來處理非同步任務，而這個東西就是 `libuv`。

libuv 除了讓 NodeJS 可以處理非同步任務之外，他還提供了 Javascript 這個程式語言中所沒有的`處理電腦系統`，比如說他提供了`非同步的文件操作系統`，簡單來說他提供了開啟或讀寫你主機中文件的功能，又比如他也提供了`非同步的 DNS 解析`，簡單來說就是可以把 IP address 解析為 domain name，他還有提供很多其他的 Javascript 原生沒有的功能，有興趣可以去看官網。

## NodeJS diagram
所以以架構層面來說，NodeJS 就是一個基於 V8 engine 提供地 Javascript 標準規範語法，加上由 C++ 撰寫的 libuv 來擴充 Javscript 沒有提供的操作電腦系統操作。

<img src="https://www.vskills.in/lms/wp-content/uploads/2019/07/image080.jpg" alt="drawing" width="860"/>

我們在前面多次提到 Javascript 和 V8 engine 是 Single-threded 的，這意味著程式與解析會逐步執行，簡單來說就是一次只做一件事，在瀏覽器中由於瀏覽器提供了 `WebAPIs` 所以才讓 Javascript 可以做到非同步的執行，那麼 NodeJS 也可以做到非同步的執行，他是基於 `libuv` 而達到可以執行非同步任務的效果，libuv 的原理滿複雜的，這邊就簡單的介紹為什麼他可以讓 NodeJS 進行非同步任務的處理。

在上面的圖中可以看到 libuv 裡面有一個 `Event Loop`，他的概念跟上面介紹的 Event Loop 一樣，當我們發出了一個執行非同步的任務時，會將這個非同步事件放到 `libuv` 中，而 libuv 會執行這個非同步事件從而不影響 V8 engine 中的同步事件，而當非同步事件處理完成後會將任務完成的結果放到 `Queue` 中，然後等到 V8 engine 中的同步任務都完成後利用 `Event Loop` 將它放回 stack 並向一般同步任務一樣處理，這樣就可以做到跟瀏覽器一樣的`非阻塞 (no-block) 操作`。

<img src="https://4.bp.blogspot.com/-u3Y10g5Niik/WSZ0eRecuzI/AAAAAAAAzFo/T_w6gaXxwzUxcInWBAb3Pbp0Wl12meMIgCLcB/s1600/%25E8%259E%25A2%25E5%25B9%2595%25E5%25BF%25AB%25E7%2585%25A7%2B2017-05-25%2B%25E4%25B8%258B%25E5%258D%25881.54.20.png" alt="drawing" width="860"/>

# 結論
本章從開頭的一個問題開始介紹了 Javascript 是如何被執行的，無論是在瀏覽器中執行又或是利用 NodeJS 執行，都是能夠回答一開的的問題，當再有人問你能不能執行一段 Javascript 的程式時，就可以跟他說『可以！無論是利用瀏覽器還是不利用瀏覽器都可以執行 Javascript』。

# Reference
- [How does JavaScript and JavaScript engine work in the browser and node?](https://medium.com/jspoint/how-javascript-works-in-browser-and-node-ab7d0d09ac2f)
- [Complete NodeJS Developer in 2022 (GraphQL, MongoDB, + more)](https://www.udemy.com/course/complete-nodejs-developer-zero-to-mastery/)
- [Node.js - wikipedi](https://en.wikipedia.org/wiki/Node.js)
- [libuv](http://docs.libuv.org/en/v1.x/)
- [Node, V8, libuv, 和非阻塞式處理（Unit44）](https://pjchender.dev/nodejs/node-v8-libuv/)
- [Node.JS Architecture](https://www.vskills.in/certification/tutorial/node-js-architecture-2/)