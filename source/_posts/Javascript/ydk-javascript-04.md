---
title: You Don't Know JavaScript [Scope & Closures] - Lexical Scope
date: 2020-10-08 21:26:16
tags:
- Javascript
- Front-end
- Scope & Closures

categories:
- You Don't Know JavaScript
---

了作用域它主宰著 JS 引擎該如何在作用域或包含它的任意嵌套作用域中查詢一個變量，我們也證明了 JS 的 scope 是在編譯時決定的，而這種範圍的術語稱為 `Lexical Scope`。

`Lexcal Scope` 的關鍵思想就是它完全由 `function`, `block`, `變量宣告`的放置位置來控制，如果你的宣告變數放置在 function 中，則編譯器在解析的時候會將該變數與 function 的範圍作連接;若是放置在 block(let/const)，則它會與最近的封閉block`{...}`相連。

必須將變量定義在可以引用的範圍之一，否則它會被定義為 `undefined`，如果在當前作用域中沒有找到這個變量則會往下一個(外部)的作用域去尋找，持續找到匹配的變量或是退到全域作用域，若到全域作用域都沒有找得則會值出一個錯誤。

<img src="https://lotschool.nl/wp-content/uploads/2020/09/Brain-comparison-article-Copyright-Getty-1.jpg" alt="drawing" width="860"/>

<!-- more -->

# Marbles, and Vuckets, and bubbles
```javascript
// outer/global scope: RED

var students = [
    { id: 14, name: "Kyle" },
    { id: 73, name: "Suzy" },
    { id: 112, name: "Frank" },
    { id: 6, name: "Sarah" }
];

function getStudentName(studentID) {
    // function scope: BLUE

    for (let student of students) {
        // loop scope: GREEN

        if (student.id == studentID) {
            return student.name;
        }
    }
}

var nextStudent = getStudentName(73);
console.log(nextStudent);   // Suzy
```
標記了三種顏色以代表三個作用域：
- **RED** : global scope
- **BLUE** : function `getStudentName(...)`
- **GREEN** : `for...in` loop
![https://ithelp.ithome.com.tw/upload/images/20201008/20124767K5alkLKbYA.png](https://ithelp.ithome.com.tw/upload/images/20201008/20124767K5alkLKbYA.png)
(圖片來源 : [You Don't Know JavaScript 2nd](https://github.com/getify/You-Dont-Know-JS/blob/2nd-ed/scope-closures/ch2.md)

Scope 的範圍是在編譯期間根據 function/blocks 的位置與彼此之間嵌套關係做決定的，每個作用域一定會被包含在一個父層作用域中，絕對不會有一個作用域分別位於兩個不同的作用域中。

而綠色作用域幾乎在藍色作用域中，而藍色作用域也幾乎都在紅色作用域中，這代表作用域可以相互嵌套並且根據需求嵌套任何深度，你可以使用自身或上層作用域中所宣告的變量但是不能使用下一層作用域的變量。

而當我們要尋找一個變量的時候，若在自身作用域中沒有找到那麼他就會往上層作用域尋找直到全域作用域，舉個例子我們在 for...of 中使用了 `students` 這個變量，但他並不是在藍色作用域中宣告個，所以他會往上一層作用域（綠色作用域）中尋找，若也沒有便會在往上一層（紅色作用域），在這裡便可以成功照到students這個變量的宣告。


# Lookup Failures
當作用域在不斷的往上層尋找變量，直到來到了全域作用域後任然沒有找到指定的變量，那麼便會擲出一個錯誤，但是這個錯誤的處理方式會根據變量的作用或程式的運行模式各有不同。

## Undefined Mess
若是變量是source，若是往上尋找發現這是一個沒有被宣告的變數，那們JS引擎就會擲出一個ReferenceError;若變量是target並且在嚴格模式下運行，則也會擲出一個ReferenceError。

在大部分的情況下，JS 的錯誤訊息都會是 `"Reference Error : XYZ is not defined."`，雖然在英文中 `not defined` 與 `undefined` 都是未定義的，但是在程式中是完全不一樣概念。
- **not defined** : 它意味著在詞法的範圍中並沒有一個可以匹配的變量。
- **undefined** : undefined代表在詞法範圍中有一個這個變量，但是他卻沒有值。

## Global....What?
若我們在非嚴格模式下使用了一個為定義的 target，則會發生一個奇怪的事情，而這其中最麻煩的便是他會自動在`全域`變數中創建一個符合的變數。

```javascript
function getStudentName() {
    // assignment to an undeclared variable :(
    nextStudent = "Suzy";
}

getStudentName();

console.log(nextStudent);
// "Suzy" -- oops, an accidental-global variable!
```
當 getStudentName(...) 中使用了一個未被定義的 target 變數，因為在自身作用域中沒有找到這個變量的宣告，所以會到上一層尋找`（全域）`，而在全域中也沒有找到，那麼他會先檢查是否是處於嚴格模式下，若不是JS會自動在全域中創建一個符合的變量。

若是處於嚴格模式下，當檢查到全域作用域中發現沒有這個變量，那麼便會擲出一個 `ReferenceError`。

參考文獻：
[you Don't Know JavaScript 2nd](https://github.com/getify/You-Dont-Know-JS/blob/2nd-ed/scope-closures/ch2.md)