---
title: You Don't Know JavaScript [Scope & Closures] - What is Scope?
date: 2020-10-05 19:44:04
tags:
- Javascript
- Front-end

categories:
- You Don't Know JavaScript
---

上面兩章節簡單的介紹了一下什麼是JS與特性，而本章節開始會正式進入到 Javascript 中，而一開始我們先從`作用域`開始介紹，什麼是作用域？作用域就是`一個變數的生存範圍`，一旦超出了這麼範圍就無法存取到這個變數，定義了變量在哪裡存活與我們能在哪裡找到他，這就是作用域。

<img src="https://www.math-salamanders.com/image-files/perimeter-area-visual-aid-2.gif" alt="drawing" width="860"/>

<!-- more -->

# Compiling Code
作用域主要是在編譯的時候定義的，所以需要了解 JS 編譯與執行之間的關係。

在一般的編譯器中，編譯一般分為三個階段 : 
1. **Tokenizing/Lexing**: 將程式碼分解為對JS有意義的token，舉個例子，若有一段程式碼`var a = 2;`那們它將會被分解為`var`、`a`、`=`、`2`和`;`，至於空格則取決於是否有意義而選擇性的轉換。
2. **Parsing**: 獲取所有的token並將他們轉換為AST(Avstract Syntax Tree)。舉例來若將`var a = 2;`轉換為AST，可以從最頂端的節點`VariableDeclaration`開始，它底下有兩個子節點，一個是代表著變量a的`identifier`與代表擁有數值的`AssignmentExpression`，而代表有數值節點下有一個`NumericLiteral`的子節點代表數值(2)。
![https://ithelp.ithome.com.tw/upload/images/20201006/20124767pPkGK2hJNp.png](https://ithelp.ithome.com.tw/upload/images/20201006/20124767pPkGK2hJNp.png)
3. **Code Generation**: 執行 AST 並將它轉換為可以執行得代碼，這個轉換會因為不同語言而有不同的結果，以 JS 來說它會對 `var a = 2;` 這個程式碼轉換成一組機械指令，將創建一個稱為 a 的變量並將2賦予給這個變量。 

由於 JS 與大部分語言一樣它不會再 build 的時期就提前編譯，而這個動作必須發收生在執行代碼前幾 ms 的時間，為了確保擁有最快的性能，JS 引擎就使用了各種技巧 (JIT,lazy compile...) 這些就不再此處進行討論。

---
為了簡單的描述 JS 對於程式的處理，可以簡單的分為解析/編譯然後是執行，雖然 JS 沒有明確的要求進行`編譯`，但是它對於程式實質上的行為卻是需要編譯後才能執行的。

我們以下面三個例子來證明這點 : 

```javascript
var greeting = 'Hello';
console.log(greeting);
greeting = .'Hi'; //SyntaxError: unexpected token .
```
上面的程式中 `Hello` 並不會被輸出，它擲出了一個 SymtaxError，因為在 Hi 前面有一個非預期的 token，以上面的例子中，若 JS 是由上而下一行一行的編譯的話，那麼應該會先輸出 `Hello` 後才發生錯誤才對，但事實上 JS 引擎能夠知道在第三行中有一個 syntaxError，所以 JS 引擎會在執行前先解析整個程式。

---
```javascript
console.log("Howdy");

saySomething("Hello","Hi");
// Uncaught SyntaxError: Duplicate parameter name not
// allowed in this context

function saySomething(greeting,greeting) {
    "use strict";
    console.log(greeting);
}
```
上面的例子中也會擲出一個 SyntaxError，因為我們在 saySomething 這個函數中使用了`嚴格模式`，而嚴格模式禁止 function 的參數使用一樣的名子，雖然拋出的錯誤並不是語法錯誤，但是在嚴格模式下這個錯誤會在執行前被擲出並被當成 `early error`。

而這個例子再次證明了 JS 會在執行代碼前對程式完全的解析，因為不這麼做的話它就不會知道 function 中有兩個一樣名子的參數與 funciton 中使用的是嚴格模式。

---
```javascript
function saySomething() {
    var greeting = "Hello";
    {
        greeting = "Howdy";  // error comes from here
        let greeting = "Hi";
        console.log(greeting);
    }
}

saySomething();
// ReferenceError: Cannot access 'greeting' before
// initialization
```
上面了例子中，因為在 let 宣告 greeting 之前就使用了這個變量 (let不會hoasting) 所以導致了 ReferenceError，這個例子也證明了 JS 是需要先將全部程式編譯後才會執行。

---
# Compiler Speak
在了解了 JS 引擎處理程式的兩個階段後，我們再回到 JS 引擎是如何識別便量並確定它在程式中的使用範圍。

為了更深入地理解需要了解更多的編譯器術語，這邊會介紹 `LHS查詢（Left-hand Side` 和` RHS查詢（Right-hand Side`，簡單來說當一個變量出現在賦值的左邊時會進行 LHS 查詢，當一個變量出現在賦值操作的右手邊會進行 RHS 查詢，說得更準確一點：
- RHS(source) 是簡單地查詢某個變量的值。
- LHS(target) 是試著找到變量容器以便它可以賦值。

```javascript
var students = [
    { id: 14, name: "Kyle" },
    { id: 73, name: "Suzy" },
    { id: 112, name: "Frank" },
    { id: 6, name: "Sarah" }
];

function getStudentName(studentID) {
    for (let student of students) {
        if (student.id == studentID) {
            return student.name;
        }
    }
}

var nextStudent = getStudentName(73);
console.log(nextStudent); //Suzy
```

## Targets
我們先找到上面例子中的 Targets。

```javascript
students = [...]
```
這個很顯然是一個 Target，因為 students 被`賦予`了一個陣列的值，它與 `nextStudent = getStudentName(73)` 一樣都是賦值操作。

```javascript
for(let student of students)
```
這裡是一個比較難發現的 Target，這句程式的意思是將 students 這個陣列迭代給 student 這個變數，所以也是一個賦值(只是不太明顯)。

```javascript
getStudentName(73);
```
這裡也是一個隱性的 Target，因為它將 73 這個數值賦予給 getStudentName 這個 function 的參數。

## Source
```javascript
for(let student of students)
```
上面提到 student 是屬於被賦值的 target，那麼 students 這個陣列便是`給予`數值的 source

```javascript
if(student.id == studentID)
```
在這邊的 student 與 studentID 都是屬於 source reference。

```javascript
return student.name
```
這邊的 student 也是屬於 source，因為它提供了可以 retuen 的值。

---
在舉個簡單的例子：
```javascript
function foo(a){
    console.log(a);
}
foo(2);
```
上面的例子既有 LHS 也有 RHS，我們仔細地對它進行分析 : 
1. 當程式調用了foo(...)，代表著函數調用要求一個指向 `foo` 的 RHS，它代表著去查詢 foo 的值並將結果交給引擎。
2. 當值2做作為參數傳遞給 foo 時，2被賦予給了參數a，這邊使用了隱含的參數賦值，所以進行了一個 LHS。
3. 而當結果被傳遞給 `console.log(...)` 時，它會對 console 這個物件進行一個 RHS 查詢，查詢是否有一個 log 的 mathod。
4. 最後當值2被傳入 `log(...)` 時，因為2被賦予給了 log 的參數，所以也進行了一次 LHS。

---
# Nested Scope
作用域是通過標示福明成查詢變量的規則，但是通常會有多餘一個作用域的問題需要考慮，就像一個程式嵌套在另一個程式碼區域或函數中，作用域也會被嵌套在其他作用域中，所以如果再直接作用域找不到需要的變量，則會`往外層作用域`尋找，如此繼續找到最外層作用域(全域作用域)。

舉個例子 :
```javascript
function foo(a){
    console.log(a + b);
}

var b = 2;

foo(2); //4
```

以上面的例子來說，當JS引擎在直接作用域(函式foo中)找不到b這個變量，那麼它就會訪問外層的作用域(全域作用域)看是否有 b 這個變數可以讓它進行 RHS。

遍歷嵌套作用域的規則很簡單，JS 引擎從當前執行的作用域開始查找變量，如果沒有找到就向上走一級繼續查找，如此類推。如果到了最外層(全局作用域)，`那麼查找就會停止無論它是否找到了變量。`

---

# Errors
為什麼需要了解 LHS 與 RHS?
因為在一個變量還沒被宣告的情況下，這兩種類型的查詢做了完全不同的行為。

```javascript
function foo(a){
    console.log(a + b);
    b = a;
}
foo(2);
```

- 當b的第一次 RHS 查詢發生時，因為它是一個沒有被宣告過的變量，所以在作用域中找不到它，因此如果在所有的作用域中都找不到b這個變量的時候，JS引擎會拋出一個 `ReferenceError`。
- 相比之下若 JS 引擎在做 LHS 查詢，雖然達到了頂層作用域都沒有找到這個變量，若是沒有在 `嚴格模式(strict)` 下，那麼就會在全局作用域中創建一個同名的新變量。

而若一個 RHS 查詢到了需要的變量，但是卻對這個變量做這個值不可能做到的事，比如將這個非函數的值當作函數執行，或是引用 `null` 或 `undefined`，那們 JS 引擎就會拋出種類錯誤`TypeError`。

所以 `ReferenceError` 是對於作用域解析失敗，而 `TypeError` 則是作用域解析成功了但是對對這個解析成功的變量做非法/不可能的操作。

參考文獻 : 
[You Don'y Know JavaScript](https://github.com/getify/You-Dont-Know-JS/blob/1st-ed/scope%20&%20closures/ch1.md)
[You Don'y Know JavaScript 2nd](https://github.com/getify/You-Dont-Know-JS/blob/2nd-ed/scope-closures/ch1.md)