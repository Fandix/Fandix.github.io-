---
title: You Don't Know JavaScript [Scope & Closures] - The Module Pattern
date: 2020-10-16 16:03:16
tags:
- Javascript
- Front-end
- Scope & Closures

categories:
- You Don't Know JavaScript
---

在本章節中將介紹這本書最重要的程式組織之一，`module`，module會用到我們之前所介紹的所有觀念(lexical scope,closure...)，我們以不同角度介紹了lexcail scope，從全域作用域到嵌套塊狀作用域，並利用 lexical scope 了解了 closure，而本章節的目標是能夠理解 module如何體現這些特性的重要性。

<img src="https://media.webteam.puppet.com/uploads/2019/06/D2549-Modules-1200x626-7.png" alt="drawing" width="860"/>

<!-- more -->

# Encapsulation and Least Exposure (POLE)
封裝個概念基礎且廣泛的被用在`物件導向(OO)`的程式中，封裝的目的是將信息(數據)與行為(功能)捆棒再一起，以發揮共同的作用，而封裝的精神可以透過簡單的方式實現，例如對於不同專案使用不同文件分別保存，以文件的形式封裝。

現代的程式語言的組件體系結構近一步的推廣了封裝，可以很自然的將類似功能的內容合併到一個程式邏輯中，然後將這個集合定義為一個`component`。

另一個主要目標是控制封裝數據和功能的隱藏性，回想在[Limiting Scope Exposure ?](https://ithelp.ithome.com.tw/articles/10252422)中提到的`POLE原則`，我們應該盡可能地將不必要暴露的變量或功能隱藏起來以提高安全性，而在JS中我們經常通過lexcal scope來達到這個功能。

將相同的程式組合在一起，並選擇性的限制訪問我們認為私有(private)的部分，將不被視為私有(private)的部分設定為公開(public)，這樣可以更好的組織我們的程式，了避免了過度暴露數據與功能，可以更好的維護。

# What is a Module?
Module相關數據和功能(method)的集合，他的特徵是他可以劃分為`需要隱藏的信息(private)`與`可以公開的信息(public)`，而公開的部分通常稱為`public API`。

## Namespaces(Stateless Grouping)
如果將一組相關的funciton組合在一起而沒有數據，那麼實際上沒有滿足module的預期封裝，這種對於狀態function分組的行為稱為`namespaces`。
```javascript
// namespace, not module
var Utils = {
    cancelEvt(evt) {
        evt.preventDefault();
        evt.stopPropagation();
        evt.stopImmediatePropagation();
    },
    wait(ms) {
        return new Promise(function c(res){
            setTimeout(res,ms);
        });
    },
    isValidEmail(email) {
        return /[^@]+@[^@.]+\.[^@.]+/.test(email);
    }
};
```
這裡的`Utils`是一組function的集合，但他們都是與狀態無關的function，雖然將function組合再一起是一個好習慣，但他並不能成為module。

## Data Structures (Stateful Grouping)
如果將數據和function捆綁再一起但卻沒有設定他的可見性，那麼就沒有使用到封裝的POLE，那們他也不算是module，通常稱這種型態為Data Structures。
```javascript
// data structure, not module
var Student = {
    records: [
        { id: 14, name: "Kyle", grade: 86 },
        { id: 73, name: "Suzy", grade: 87 },
        { id: 112, name: "Frank", grade: 75 },
        { id: 6, name: "Sarah", grade: 91 }
    ],
    getName(studentID) {
        var student = this.records.find(
            student => student.id == studentID
        );
        return student.name;
    }
};

Student.getName(73); // Suzy
```
由於`records`是公開可以訪問的數據而不是隱藏在public API後面的，所以此處的Student並不是一個真正個module，他雖然有包含了數據與功能，但沒有控制可見性，所以最好是稱之為Data Structures。

## Modules (Stateful Access Control)
為了體現module的全部經精神，我們不僅需要分組與狀態，還需要通過可見性(private/public)進行訪問控制，我們可以將上面的`student`更改為一個module。
```javascript
var Student = (function defineStudent(){
    var records = [
        { id: 14, name: "Kyle", grade: 86 },
        { id: 73, name: "Suzy", grade: 87 },
        { id: 112, name: "Frank", grade: 75 },
        { id: 6, name: "Sarah", grade: 91 }
    ];

    var publicAPI = {
        getName
    };

    return publicAPI;

    // ************************

    function getName(studentID) {
        var student = records.find(
            student => student.id == studentID
        );
        return student.name;
    }
})();

Student.getName(73);   // Suzy
```
將Student更改為一個module，他有一個`public API(getName(...))`，只有這個API可以訪問到內部的`records`。

從外部`Student.getName(73)`來調用內部的function，而records透過closure將他保存在記憶體中讓之後調用的`getname(...)`依然可以使用這個變量，雖然上面的例子中是將public API放在`object publicAPI`之中，但是實際上可以單純只返回`getname(...)`，這樣也能夠滿足module的所有核心要求。
```javascript
var Student = (function defineStudent(){
    var records = [
        { id: 14, name: "Kyle", grade: 86 },
        { id: 73, name: "Suzy", grade: 87 },
        { id: 112, name: "Frank", grade: 75 },
        { id: 6, name: "Sarah", grade: 91 }
    ];

    return function getName(studentID) {
        var student = records.find(
            student => student.id == studentID
        );
        return student.name;
    }
})();

Student(73);   // Suzy
```

對於lexcial scope的工作原理來說，在外部module定義的函式中定義的變量與函式都會默認為private，只有return出去的public API才可以供外部使用。

### Module Factory(Multiple Instances)
如果我們希望程序中定義一個可以支援多個實例的moudle，我們可以調整我們的程式。
```javascript
// factory function, not singleton IIFE
function defineStudent() {
    var records = [
        { id: 14, name: "Kyle", grade: 86 },
        { id: 73, name: "Suzy", grade: 87 },
        { id: 112, name: "Frank", grade: 75 },
        { id: 6, name: "Sarah", grade: 91 }
    ];

    var publicAPI = {
        getName
    };

    return publicAPI;

    // ************************

    function getName(studentID) {
        var student = records.find(
            student => student.id == studentID
        );
        return student.name;
    }
}

var fullTime = defineStudent();
fullTime.getName(73); // Suzy
```
沒有將defineStudent()定義為IIFE而是將它定義為普通函式，這樣可以將它賦予給多個不同的變量，這個稱為`module factory`。

### Classic Module Definition
- 必須有一個外部作用域（通常是module factory），並且至少被呼叫一次。
- module內部至少要有一個代表著module狀況的信息。
- module必須在public API上return一個對private數據引用的function。

# Node CommonJS Modules
在[Around the Global Scope ?](https://ithelp.ithome.com.tw/articles/10251263)提到了Node Common Module，與前面介紹的經典module不同，Node Module可以將module factory或IIFE與其他程式碼(包括其他module)捆綁再一起，`CommonJS module以文件為基礎`，所以每個文件就是一個module。
```javascript
module.exports.getName = getName;

// ************************

var records = [
    { id: 14, name: "Kyle", grade: 86 },
    { id: 73, name: "Suzy", grade: 87 },
    { id: 112, name: "Frank", grade: 75 },
    { id: 6, name: "Sarah", grade: 91 }
];

function getName(studentID) {
    var student = records.find(
        student => student.id == studentID
    );
    return student.name;
}
```
雖然`records`與`getName(...)`他們是處於這個文件中的最上層作用域，但他並不是`全域作用域`，在預設的情況下這個文件中的所有內容都是`private`，若要在CommonJS module的public API上加入需要公開的內容，你可以將需要公開的內容加入到`module.exports`作為他的屬性。

若是要在引入其他module的實例，可以使用Node提供的`require(...)`method
```javascript
// another module

var Student = require("/path/to/student.js"); //use method in Node

Student.getName(73); // Suzy
```
上面的`Student`就reference了其他module的public API，CommonJS module與使用IIFE一樣都是單例實例，意味著無論你對同一個module require(...)多次，依然只會得到對單個module的實例。

rqeuire(...)是一種全有或全無的機制，它包括了對整個module中public API的引用，如果只想訪問public API期中的一部分
```javascript
var getName = require("/path/to/student.js").getName;

// or alternately:

var { getName } = require("/path/to/student.js");
```
與典型的module相似，CommonJS module的API也會透過closure將內部module的數據儲存起來。


# Modern ES Modules (ESM)
ES Module的格式與CommonJS Module的格式有些類似，ESM也是以文件為基礎，module時例為單例並且默認下所有內容都是private，而他們之間明顯的區別在於ESM是自動使用`嚴格模式`而不需要在開頭宣告，並且`無法將ESM設定為非嚴格模式`。

ESM並非像CommonJS Module一樣使用`module.exports`的方式將public API公開，而是使用`export`，在引入的部分也從`import`替換了`require(...)`，我們可以調整student.js以ESM的格式呈現。
```javascript
export { getName };

// ************************

var records = [
    { id: 14, name: "Kyle", grade: 86 },
    { id: 73, name: "Suzy", grade: 87 },
    { id: 112, name: "Frank", grade: 75 },
    { id: 6, name: "Sarah", grade: 91 }
];

function getName(studentID) {
    var student = records.find(
        student => student.id == studentID
    );
    return student.name;
}
```
上面的程式碼中唯一的變化`export{ getName }`，和以前一樣隨然他們都被定義在當前文件的最上層作用域，但是他們卻不是屬於全域作用域中，ESM對於`export`語句提供了很多變化
```javascript
export function getName(studentID) {
    // ..
}
```
將export定義在function關鍵字前面，這樣他依然是一個function declaration並且可以順利被export，也就是說`getName`透過function hoisting到此文件作用域的最上方，所以在這個module的整個範圍都能用。

還有一種export的方式，他稱為default export，這邊就要先介紹一下什麼事default export，他與一般的export(name export)差別在哪。

## default export vs name export
export可以區分為兩種，這兩種的匯出手法略有不同，他會影響到其他module的import運用。
- named export(具名匯出)：可以匯出獨立的`物件`、`變數`、`函式`等等，匯出之前需要`給予特定名稱`，使用import的時候也需要使用相同的名稱，一個moudle`可以有多個named export`。
- default export(預設匯出)：一個module中`只能有一個default export`，而不需要給予名稱。
兩者可以兩者可以共存在同一個module中，但是`default export只能有一個`。


```javascript
export default function getName(studentID) {
    // ..
}
```
由上面的介紹可以了解，使用default export的function可以在引入的module中定義它的名字。

至於`import`需要在其他module的頂層使用，並在語法上也有許多變化。
```javascript
import { getName } from "/path/to/students.js";

getName(73); // Suzy
```
上面的程式中，在最上方import了其他module public API的內容，並將它添加到當前module的最頂層作用域中，可以在`{...}`中列出多個需要引入的API成員，並用`逗號`區分，也可以使用`as`關鍵字將他們重新命名。
```javascript
import { getName as getStudentName } from "/path/to/students.js";

getStudentName(73); // Suzy
```

參考文獻：
[You Don't Know JavaScript -2nd](https://github.com/getify/You-Dont-Know-JS/blob/2nd-ed/scope-closures/ch8.md)