---
title: You Don't Know JavaScript [Scope & Closures] - The (Not So) Secret Lifecycle of Variables
date: 2020-10-12 14:55:34
tags:
- Javascript
- Front-end
- Scope & Closures

categories:
- You Don't Know JavaScript
---

經由前幾篇文中應該對於全域作用域或嵌套全域作用域有一定的了解，但這僅僅只知道這麼變量是在哪一個作用域中宣告而已，若是我們在宣告這個變數之前就使用它會發生什麼事？又或是我們在同一個作用域中對同一個變數宣告兩次會發生什麼事？

<img src="https://images.ctfassets.net/mrop88jh71hl/5CHsSHA2hvUjqR8EBiIfjF/7a2cb4fc75dc6da6593d5cbdafbe5733/AdobeStock_274134297-min.jpeg?w=800&q=100" alt="drawing" width="860"/>

<!-- more -->

# When Can I Use a Variable?
據我們所知，若我們要在這個作用域使用一個變量必須要在宣告他之後，但是這個觀念是不一定的。
```javascript
greeting(); //Hello!

function greeting(){
    console.log('Hello!');
}
```
上面的程式碼中你可能會困惑，為什麼在 greeting(...) 被宣告之前就可以在第一行中使用？

回想 [What is Scope?](https://ithelp.ithome.com.tw/articles/10249231) 中我們提到的，所有的標示在JS引擎編譯的時候就已經註冊到各自的作用域中了，此外每個標示都是在所屬的作用域開頭便被創建，因為這個現象所以就算變量被宣告在下方，但是上面還是可以使用這個變量，這個現象被稱為 `hoisting`。

但是不能單純使用 `hoisting` 來解釋這個問題，我們在程式的一開始就看到了一個 `greeting` 的標示，但是為什麼我們可以在宣告他是 function 之前就呼叫 `greeting(...)` 呢？

這是因為 function 宣告的一個特別現象，稱為 `function hoisting`，當 function 的名稱標示在作用域頂部被註冊時，他會自動初始化對這個 function 的引用，換句話說 greeting(...) 這個 function 在這個作用域被註冊的時候，便會去初始化所有使用到他的引用（第一行的greeting();）

另一個細節是 function hoisting 與變量的 hoisting 都是將自身的標示附加到最近的`封閉函式作用域（若沒有則是全域作用域）`而不是塊狀作用域中。

## Hoisting: Declaration vs. Expression
function hoisting 只會發生在`正式的 functio n宣告`而不會發生在 `function expression`。
```javascript
greeting();  // TypeError

var greeting = function greeting() {
    console.log("Hello!");
};
```
在第一行中 JS 擲出了一個 `TypeError`，一般來說 TypeError 代表著我們嘗試使用一個不合法的值進行操作，在正常環境下 JS 會提供一些比較有用的錯誤訊息，比如說 `undefined in not a function` 或 `greeting is not a function`，但是這邊只有顯示 TypeError。

這個 Error 並不是代表著 greeting 沒有在這個作用域中被找到的 `ReferenceError`，TypeError 代表雖然有找到這個標示但是在這個時刻並不是函數的引用，因為他`尚未被 reference 給function` 所以自然沒辦法被呼叫。

若是使用 `var` 宣告一個變量，那麼他會在`作用域開頭自動初始化這個標示將他初始化為 undefined`，一但初始化那麼他就可以在這個作用域中被使用，所以第一行的 greeting 他只被定義為`undefined`，要到第三行才被 assigned 為 function，所以自然會出錯。

## Variable Hoisting
```javascript
greeting = "Hello!";
console.log(greeting); // Hello!

var greeting = "Howdy!";
```
對於上面的程式碼中可能會有疑問，明明 greeting 是在後面才宣告的但是為什麼在第一行就可以賦值？而且console出來的也不是宣告的值？
這邊有兩個解釋：
- 標示的hoisting
- 自動初始化為`undefined`
所以對於 greeting 來說，他在一進到作用域中就被自動初始化為 undefined(hoisting)，所以第一行便可以對 greeting 賦值（undefined -> Hello）。


# Hoisting: Yet Another Metaphor
hoisting 不是 JS 引擎執行之前的具體執行步驟，而是將JS在執行程式之前設置程序所做的動作可視化，簡單來說可以把它想像為JS引擎在執行前會重寫這段程序，因此會將上面的程式改寫為
```javascript
var greeting;           // hoisted declaration
greeting = "Hello!";    // the original line 1
console.log(greeting);  // Hello!
greeting = "Howdy!";    // `var` is gone!
```
hoisting 建議 JS 對原始程式進行預先處理以便在執行之前將所有宣告都移動到各自的作用域頂部，當然函數的宣告也會被移動到最上層。
```javascript
studentName = "Suzy";
greeting(); // Hello Suzy!

function greeting() {
    console.log(`Hello ${ studentName }!`);
}
var studentName;
```
對於 hoisting 的規則會要求JS將所有的 function 宣告移動到各自作用域頂部，等待所有 function 結束後才會輪到變量宣告
```javascript
function greeting() {
    console.log(`Hello ${ studentName }!`);
}
var studentName;

studentName = "Suzy";
greeting(); // Hello Suzy!
```
hoisting 將程式重新編排的機制是一個簡單易懂的方法，但是實際上 JS 引擎並不是這麼做的，因為他不可能向前看並找到宣告，準確來說能夠達到這個功能的唯一方法是`完全解析程式`，


# Re-declaration?
如果我們在同一個作用域中不只一次的宣告同一個變數會發生什麼事？
```javascript
var studentName = "Frank";
console.log(studentName); // Frank

var studentName;
console.log(studentName);   // ???
```
對於上面的程式碼可能會覺得第二個 var studentName 會重新宣告這個變數（reset），所以第二個 console 會變成 `undefined`，但是實際上並不是這樣的，從我們上面提到的 hoisting 來看，這段程式碼會變成
```javascript
var studentName;
var studentName; // second declared

studentName = "Frank";
console.log(studentName); // Frank 

console.log(studentName); // Frank
```
因為 hoisting 會將所有的宣告移動到作用域的上方，所以原本在中間的宣告會被 hoisting 到上方，所以一樣會輸出 `Frank`，而第二次的宣告則是一個無意義的操作，但是如果使用 `var studentName = undefined`，那麼結果會是完全不同的
```javascript
var studentName = "Frank";
console.log(studentName); // Frank

var studentName; //the no-op
console.log(studentName); // Frank

// let's add the initialization explicitly
var studentName = undefined;
console.log(studentName); // undefined
```
將 student 顯性的再次定義為 `undefined`，那他的結果就會跟被動賦予 undefined 的結果不一樣。

重複的使用 var 去宣告一個一樣的變量是沒意義的，實際上會是什麼都不做
```javascript
var greeting;

function greeting() {
    console.log("Hello!");
}

var greeting; // basically, a no-op

typeof greeting; // "function"

var greeting = "Hello!"; //re declrate to string

typeof greeting; // "string"
```
第一行中宣告了一個 greeting 並自動初始化為 `undefined`，由於這個標籤已經被宣告了，所以 function 不需要對這個標籤再次宣告一次只需要將 function hoisting，他會自動初始化並覆蓋原本這個標籤的設定（undefined -> function），而第二個 var 的宣告並不會有任何操作，因為他已經被初始化過了;而實際上將 `Hello!` 賦予給 greeting，使他的值從 function 變為string`與 var 本身無關`。

那如果是使用 `let` 或 `const` 重複宣告呢？
```javascript
let studentName = "Frank";

console.log(studentName);

let studentName = "Suzy";
```
這樣的操作並不會被運行，因為他會擲出一個 `SyntaxError`，而這個錯誤的意思代表 studentName 這個變量`已經被宣告過了`，換句話說重複宣告對使用 let/const 來說是不允許的。
```javascript
var studentName = "Frank";

let studentName = "Suzy"; //SyntaxError
```
```javascript
let studentName = "Frank";

var studentName = "Suzy"; //SyntaxError
```
對於上面這兩種情況來說，都會在第二次宣告的時候擲出 SyntaxError，這意味著如果要嘗試使用 `re-declare` 則必須是全程使用 var 宣告才行。

## Constants?
對於 `const` 的使用規範要比 `let` 來得嚴格，const不 能在同一個作用域中重新宣告，但是他的這個規則與 let 不一樣，`const 要求宣告的變量要有初始值`，若沒有則會擲出SyntaxError。
```javascript
const empty; // SyntaxError
```
const也不能重新宣告
```javascript
const studentName = "Frank";
console.log(studentName); // Frank

studentName = "Suzy";   // TypeError
```
上面的程式中擲出的錯誤是 TypeError 而不是 SyntaxError，因為 SyntaxError 是代表`語法錯誤導致程式無法執行`，TypeError 則是代表`程序執行期間出現的錯誤`，由於在程式中已經執行並將第一個宣告的 studentName console 出來，所以是屬於`執行中`的錯誤。

## Loops
由上面的介紹中可以發現，JS不希望我們對一個變數重複宣告，但是這個行為在迴圈中也是嗎？
```javascript
var keepGoing = true;
while (keepGoing) {
    let value = Math.random();  
    if (value > 0.5) {
        keepGoing = false;
    }
}
```
上面的程式碼中我們在 while 迴圈中不斷的使用let重新宣告 `value = Math.rendom();`，這樣的操作會造成錯誤嗎？

答案是不會的，因為每個作用域都遵守作用域規則，換句話說在 while 在每次迴圈執行的時候都會將整個作用域重置，所以每個迭代的 while 迴圈都是自己的一個新作用域，對這些作用域來說 value 只有被宣告一次所以並不會造成錯誤，但是如果我們將value的宣告改為使用 `var`會發生什麼事？
```javascript
var keepGoing = true;
while (keepGoing) {
    var value = Math.random(); //change let to var
    if (value > 0.5) {
        keepGoing = false;
    }
}
```
會因為 var 可以允許而不斷的重新宣告嗎？答案是不會的，因為 var 不屬於塊狀作用域宣告，所以他會將自身附加到全域作用域中，所以根本來說 value 是和 keepGoing 一樣的全域作用域中，所以他只被宣告了一次，所以不會有重新宣告的問題。

那如果是 `for loop` 呢？
```javascript
for (let i = 0; i < 3; i++) {
    let value = i * 10;
    console.log(`${ i }: ${ value }`);
}
/* 
  0: 0
  1: 10
  2: 20 
*/
```
我們已經了解對於 `value `來說，因為每次迴圈他都在新的作用域中所以不會有重複宣告的錯誤，但是對於 `i` 來說呢？

要解決這個問題我們需要先了解 i 是處於哪個作用域中，雖然他看起來像在全域作用域中但實際上他是處於 `for loop` 的作用域中
```javascript
{
    // a fictional variable for illustration
    let $$i = 0;

    for ( /* nothing */; $$i < 3; $$i++) {
        // here's our actual loop `i`!
        let i = $$i;

        let value = i * 10;
        console.log(`${ i }: ${ value }`);
    }
    // 0: 0
    // 1: 10
    // 2: 20
}
```
這樣可以清楚的了解，其實i與 value 一樣都一直處於新的作用域中，所以不會有重複宣告的問題發生，那麼問題又來了，如果對於 for loop 使用 `const` 來宣告i結果還會一樣嗎？
```javascript
for (const i = 0; i < 3; i++) {
  //...
}
```
我們將for loop中的i由let宣告改為使用const宣告原本預期會跟使用let一樣，但是其實不一樣，因為若是以觀測i的作用域來說
```javascript
{
    // a fictional variable for illustration
    const $$i = 0;

    for ( ; $$i < 3; $$i++) {
        // here's our actual loop `i`!
        const i = $$i;
        // ..
    }
}
```
雖然對於i來說，他是處於for loop作用域中所以不會有問題，但是有問題的是在for loop外的作用域使用const宣告的一個假的$$i變數，由於是使用const做的宣告，所以並不能在for loop中進行`++`的動作（re-assignment），所以這時候便會報錯。


# Uninitialized Variables (aka, TDZ)
當使用 var 去宣告一個變數的時候，會因為 hoisting 的作用將這個變數提升到作用域的頂層並自動初始化為 `undefined`，因此讓這個變數在整個作用域中都可以使用，但是 `let` 與 `const` 並`沒有`這個功能。
```javascript
console.log(studentName); // ReferenceError

let studentName = "Suzy";
```
在第一行擲出了一個 ReferenceError，它代表著你不能夠在還沒初始化這個變數之前就使用它。

但是若是錯誤訊息表示我們在還沒初始化之前就使用這個變數，那麼我們將程式改寫一下
```javascript
studentName = "Suzy";   // let's try to initialize it!

console.log(studentName); // ReferenceError

let studentName;  //declarate variable
```
就算這樣更改程式後依然發生錯誤，但是我們已經在一開始的地方對他初始化了，那麼是為什麼又發生錯誤？

對於 let/const 的初始化來說需要在宣告與句後面加上賦值，這樣便能完成對於 let/const 宣告的變數初始化。
```javascript
let studentName = "Suzy"; //intialized
console.log(studentName); // Suzy
```
除了這種方法之外也可以將宣告與賦值分成兩段
```javascript
let studentName; // let studentName = undefined;
 
studentName = "Suzy"; //assignment value

console.log(studentName); // Suzy
```
這邊會有一個很特別的現象，對於 `var studentName` 來說他並不是等於 `var studentName = undefined`，但是對於 let 來說他們是相同的，區別在於 var studentName 會在作用域頂部自動初始化而 let studentName 並不會這麼做。

當使用 let/const 宣告變量尚未被初始化之前的這段時間稱為 TDZ(Temporal Dead Zone)，在這段期間內是不能對這個變量進行訪問，只有編譯器在原始聲明中所留下的指令執行初始化之後才能自由地在所屬的作用域中使用，以技術上來說 va r也是具有 TDZ 的，只是他不會被我們察覺。

對於TDZ中所提到的`時間`他確實是指時間而`不是程式碼中的位置`。
```javascript
askQuestion();  // ReferenceError

let studentName = "Suzy";

function askQuestion() {
    console.log(`${ studentName }, do you know?`);
}
```
雖然 askQuestion(...) 中的 console 是放在 let studentName 宣告之後，但是以時間上來說 askQuestion(...) 被呼叫的時間早於 studentName 被宣告，所以會產生ReferenceError。

## let/const don't hoist?
許多人會覺得 let/const `不會 hoisting`，但這其實是不對的，其實 let/const 他也會有 hoist 的現象，不過他與 var 的區別在於 `let/const 的 hoist 不會在作用域頂部自動初始化`，書中的作者認為`自動註冊變數到作用域頂部與自動初始化是不一樣的操作`，不應該將他們都歸類於 hoisting，我們可以舉一個例子:
```javascript
var studentName = "Kyle";
{
    console.log(studentName);  // ???

    let studentName = "Suzy";

    console.log(studentName);  // Suzy
}
```
如果以 let/const 不會 hoisting 的觀念看這個程式碼應該會覺得第一個 console 會打印出 `kyle`，因為在這個時候只有外部作用域有一個 studentName 的宣告，但事實上這段程式碼也會TDZ Error，這代表在`{...}`中 `let studentName = "suzy";` hoisting 到這個 bloc k的最上方了只是還沒初始化，所以第一個c onsole 才會擲出 TDZ Error。

總結來說，會發生 TDZ Error 是因為 let/const`確實將宣告的參數移動到作用域得頂部但卻不像 var 一樣會自動為他們初始化`，他將初始化的動作推遲到原始聲明的出現，而在這段時間內對變數進行操作都會導致錯誤，所以要減少 TDZ Error 的方法最好是將所有的 let/cons t宣告放在作用域的頂部，讓你TDZ的時間幾呼趨近於0。

參考文獻：
[You Don't Know JavaScript -2nd](https://github.com/getify/You-Dont-Know-JS/blob/2nd-ed/scope-closures/ch5.md)