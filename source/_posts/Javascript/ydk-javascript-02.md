---
title: You Don't Know JavaScript [Get Started] - Into Programming
date: 2020-10-04 15:24:42
tags:
- Javascript
- Front-end
- Get Started

categories:
- You Don't Know JavaScript
---

本章會介紹什麼是編成以及 Javascript 中變數的不同類型。

<img src="https://miuc.org/wp-content/uploads/2020/08/6-Reasons-why-you-should-learn-Programming-1280x720.png" alt="drawing" width="860"/>

<!-- more -->


# Code
代碼是一組告訴計算機要執行什麼任務的特殊指令，通常被保存在文本文件中，合法的格式與指令的組成被稱為一種`程式語言`

## 語句
在JS中一個合法的語句看起來像下面這樣 : 
```javascript
a = b * 2;
```
字符`a`與`b`被稱為`變量`，可以把它當作可儲存任何東西的盒子，而`=`與`*`是`操作符`他們對值實施動作，比如賦值或數學運算，而在JS中大多語句都以`分號(;)`代表結束。

而上面代碼的意思變可以解釋為 : 
1. 取得變量b中所儲存的數。
2. 將取得的數 * 2。
3. 將結果賦予給另一個變量a。

## 表達式
語句是由一個或多個`表達式`組成的。
```
a = b * 2;
```
在上面的語句中有四個表達式 : 
1. `2`是一個`字面表達式`。
2. `b`是一個`變量表達式`，代表著將取出b所存放的數。
3. `b * 2`是一個`算數表達式`，代表著進行數學運算(乘法)。
4. `a = b * 2`是一個賦值表達式，代表將運算過後的結果賦予給變量a。

除了上述這些表達式之外，還有一種更常被使用的表達式，那便是`調用表達式(函式)`。

# Values & Types
在一個程序中會根據開發者打算對這些值做什麼來選擇不同的表達形式，再編成終將這些不同的表達形式稱為`類型`。

JS中有六種資料型別 : 
1. Boolean
2. Null
3. Undefined
4. Number 
5. String
6. BitInt
7. Symbol

# Variables
將一個值賦予給一個符號容器，那他就被稱為一個變量，而JS屬於弱型別語言，代表著你宣告的變量可以存取任何類型的數據，並不會被類型進行約束。

# Blocks
在開發的時候，我們常需要將一個系列的語句分組再一起，這就稱為模塊化，在JS中模塊被定為在一個`大括號內{...}`中的一個或多個語句。
```javascript
let amount = 99.99;

//Blocks
{
    amount = amount * 2;
    console.log(amount); // 199.98
}
```

# Scope
作用域的在JS中的概念，每一個函數都有自己的作用域代表著訪問變量的規則，也就是說只有自己函數內部的帶把才能訪問到這個函數作用域的變量，值得注意的是，在同一個作用域內只能存在唯一的變量名稱，但是在不同作用域中可以有兩個一模一樣名子變量，因為它們存在於不同作用域所以互不相干擾。

另外作用愈也可以嵌套在另一個作用域中，那們內部作用域變可以訪問到外部作用愈的變數，但是`外部作用域無法訪問內部作用域`。

參考文獻 : 
[You Don't Know JavaScript](https://github.com/getify/You-Dont-Know-JS/blob/1ed-zh-CN/up%20%26%20going/ch1.md)