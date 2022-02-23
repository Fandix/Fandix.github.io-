---
title: You Don't Know JavaScript [Scope & Closures] - The Scope Chain
date: 2020-10-09 18:02:46
tags:
- Javascript
- Front-end

categories:
- You Don't Know JavaScript
---

在前幾章對辭法範圍做具體的定義與基本概念介紹，也了解作用域的嵌套與找尋變量的像上搜尋，在本章節中將介紹什麼是`作用域鏈`，可以把它想像為是`連接著嵌套作用域之間的通道`，而這個作用域鏈是固定方向的，它意味著變量的查詢只能往上/外層作用域移動。

<img src="https://www.parktool.com/assets/img/repairhelp/_1200x630_crop_center-center_82_none/chain_replace_002.jpg" alt="drawing" width="860"/>

<!-- more -->

# "Lookup" is Conceptual
![https://ithelp.ithome.com.tw/upload/images/20201008/20124767K5alkLKbYA.png](https://ithelp.ithome.com.tw/upload/images/20201008/20124767K5alkLKbYA.png)
我們在前兩章中使用了這個程式來區分作用域，若我們在 for...loop 中呼叫了 `students` 這個變量，我們在 [Lexical Scope](https://ithelp.ithome.com.tw/articles/10249458) 提到了 lookup 的概念，JS 引擎會先在當前的作用域中查詢使否有宣告這個變量，若沒有會持續向上層作用域中查詢直到全域作用域中，這個查詢的過程有助於我們了解作用域個概念，但在正常的情況下`JS通常是不這麼做的`。

通常在編譯的過程中就會決定好變量的 scope，基於這個原理，所以變量的作用域是不會根據程式運行時發生變化，由於 scope 是在編譯中得知並且不可變的，所以這個 scope 的訊息有可能與 AST儲存在一起，然後當程式運行的時候使用這個訊息。

換句話說，其實 JS 並不需要像 [Lexical Scope](https://ithelp.ithome.com.tw/articles/10249458) 說的要一一遍歷作用域來找到這個變量在哪個作用域中宣告，因為這個訊息其實在編譯期間就知道了，透過不需要花費時間進行搜尋，它可以使我們的程序更高效能的運行，

不過凡事都有個例外，若我們使用了一個從未在任何地方宣告個變數，因為其實每一個專案對於JS來說都是獨立的，所以就算在這個專案中沒有辦法找這個變量的宣告，但是他也有可能被宣告在共享的全域範圍中，所以不一定會是個錯誤。

因此可能將這個結果（變量準確宣告的位置）需要推遲到運行的時候才知道，所以當JS還沒編譯到其他專案的這個期間，這個變量都使處於無法確定作用域的情況，而延遲查找最終才會將這個變量宣告（有可能是全域中）但是這個動作只需要進行一次，因為作用域是無法被更改的。

# Shadowing
當兩個或多個變量（每個同名的變量都存在於不同作用域中）時，具有不同 lexical scope 的問題就變個更重要，值得注意的是當你在`同一個作用域中，你不能同時宣告同一個名稱的變量`，他只能接受一個名字一個宣告，因此若需要使用兩個或多個同名的變量則需要使用嵌套作用域。
```javascript
var studentName = "Suzy";

function printStudent(studentName) {
    studentName = studentName.toUpperCase();
    console.log(studentName);
}

printStudent("Frank");  // FRANK
printStudent(studentName);  // SUZY
console.log(studentName);  // Suzy
```
在上面這段程式中我們在第一行中（全域作用域）宣個了一個變量 `studentName`，而相同名字的變量我們也宣告了一個，但是他是存在於 printStudent(...) 作用域中。

藉由 lookup 的概念，當我們使用了一個變量時，他會從當前的作用中往上查詢，一但找到匹配的變量就會停止，所以當 function 中的 console.log(...) 使用到 studentName 這個變量，他先在當前作用域中（printfStudent(...)）尋找，發現在當前作用域中就找到了這個變量的宣告，那麼他就會停止收尋並直接使用這個變量（不會考慮全域作用域中的studentName）。

這個概念便是 lexcal scope 中的 `shadowing`，printfStudent(...) 作用域中的 studentName shadowing 了全域作用域中的 studentName，若是你需要使用到外部作用域的變量，那麼你就不能在自身作用域中宣告一個相同名字的變量，因為這樣做他將永遠只會使用自身作用域的變量。


# Global Unshadowing Trick
`注意！這不是一個好的方法，他可能造成你的程式發生非預期的錯誤`。

其實有方法可以在被變量 shadowing 的情況下使用到全域中的變量。
```javascript
var studentName = "Suzy";

function printStudent(studentName) {
    console.log(studentName); 
    console.log(window.studentName); //uee global
}
// "Frank"
// "Suzy"
```
可以使用 `window.studentName` 這樣的方法來訪問到全域中的 studentName，這是唯一能夠在 shadowing 情況下訪問到外部作用域變數的方法。

這個技巧只適用於訪問全域作用域中的變量（不能訪問到非全域的上一層作用域）。
```javascript
var special = 42;

function lookingFor(special) {
    function keepLooking() {
        var special = 3.141592;
        console.log(special);
        console.log(window.special);
    }
    keepLooking();
}

lookingFor(112358132134);
// 3.141592
// 42
```
在全域中宣告了一個 special，也在 lookingFor(...) 和 keepLooking(...) 中宣告了
一樣名字的變數，在 keepLooking(...) 中使用了special變數，可是因為他自身的作用域中就有宣告這個變數，所以就直接使而不會使用 lookingFor(...) 和全域的，而下一行使用 `window`則會直接訪問到全域作用域中的變數，`所以使用 window 只能訪問全域中的變數而不能訪問非全域的上一層作用域`。

---
即使是全域也只能訪問到 `var` 與 `function`。
```javascript
var one = 1;
let notOne = 2; //use let
const notTwo = 3; //use const
class notThree {} //object

console.log(window.one);       // 1
console.log(window.notOne);    // undefined
console.log(window.notTwo);    // undefined
console.log(window.notThree);  // undefined
```

## Copying Is Not Accessing
```javascript
var special = 42;

function lookingFor(special) {
    var another = {
    // use object to copy value of arguments to another.special
        special: special 
    };

    function keepLooking() {
        var special = 3.141592;
        console.log(special);
        console.log(another.special);  // Ooo, tricky!
        console.log(window.special);
    }
    keepLooking();
}
lookingFor(112358132134);
// 3.141592
// 112358132134
// 42
```
也可以透過這種方法訪問到上層作用域，這個概念是將原本被 shadowing 的變數`複製給另一個變量`，那麼這個被複製的變量就可以被訪問到（除非他也被 shadowing），但是這個行為並不代表是特殊的訪問方法，他只是`藉由另一個容器（擁有被 shadowing 的變量的值）（有點饒舌）`讓我們可以訪問到他，它意味著我們訪問的並不是 lookingFor(...) 中的 special。

## Illegal Shadowing
不是所有宣告都能夠shadowing上一層作用域的變數，`let 可以 shadowing var`，但`var 不能 shadowing let`
```javascript
function something() {
    var special = "JavaScript";
    {
        let special = 42;   // totally fine shadowing
    }
}

function another() {
    {
        let special = "JavaScript";
        {
            var special = "JavaScript";  // ^^^ Syntax Error
        }
    }
}
```
因為 var 宣告會試圖跨越同名的let宣告範圍，換句話說使用 let 宣告的這個 special 會阻止 var 的 hoasting，所以就導致了錯誤。

# Funciton Name Scope
```javascript
var askQuestion = function ofTheTeacher() {
    console.log(ofTheTeacher);
};

askQuestion();  // function ofTheTeacher()...

console.log(ofTheTeacher);  // ReferenceError: ofTheTeacher is not defined
```
當我們呼叫 askQuestion 時，有成功 console 出 ofTheTeacher(...)，但是直接 console ofTheTeacher 卻無法成功，是因為這是一個 `funciton expression`，對於ofTheTeacher(...) 來說，他只存活在賦予 askQuestion 這段時間，一但他將自身賦予給了 askQuestion 後他便會消失，所以在其他地方呼叫他才會是 not defined。

# Arrow Functions
在 ES6 中加入了箭頭函數
```javascript
var askQuestion = () => {
    //...
}
```
使用 `=>` 讓我們可以不需要對這個 function 命名，由於這特性，意味沒辦法透過 function 名稱來呼叫他，雖然箭頭函數是匿名的形式，但是他的辭法規則與正常的 function 一樣，無論帶不帶有{...}的箭頭函數都會創建一個單獨的內部作用域，這個作用域的規則與function一樣。

參考文獻：
[You Don't Know JavaScript](https://github.com/getify/You-Dont-Know-JS/blob/2nd-ed/scope-closures/ch3.md)