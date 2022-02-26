---
title: You Don't Know JavaScript [this & Object Prototypes] - Object [上]
date: 2020-10-30 17:27:08
tags:
- Javascript
- Front-end
- This & Object Prototypes

categories:
- You Don't Know JavaScript
---

我們在前面幾章中介紹了this的綁定，說明了this最常被搞混的觀點也介紹了如何透過call-site與綁定的4個規則確定this所指向的物件是那一個，介紹了這麼多this所指向的物件位置，那麼物件到底是什麼？而為什麼我們的this會需要指向它呢？我們會在本篇章中進行講解。

<img src="https://cdn.myfonts.net/cdn-cgi/image/width=720,height=360,fit=contain,format=auto/images/pim/10000/K4PSu1LsztwizswUXl50cozb_ed8f2a2c9c121533c394226c8db6ad43.png" alt="drawing" width="860"/>

<!-- more -->

# Syntax
物件有兩種形式：declaration與constructed。
- declaration：
```javascript
var myObj = {
	key: value
	// ...
};
```
- constructed
```javascript
var myObj = new Object();
myObj.key = value;
```
使用兩種方法產生的物件完全相同，他們唯一的區別在於如果你`使用declaration可以在物件中添加一個或多個key/value`，但是`使用constructed只能將屬性一個一個加入`。


# Type
物件是構建大部分JS的通用模塊，他是JS中6種主要類型之一。
- string
- number
- boolean
- null
- undefined
- object

null有時候會被當成一個物件類型，這種誤解來自於JS中的一個bug使得`typeof null`會回傳object，但這是不對的，因為null有屬於自己的類型，還有一個常見的錯誤判斷`JavaScript中的一切都是物件`，這是不對的。

除了上面的六種著要的型態之外，還有一些特殊的存在稱為`物件子類別（複雜基本類型）`，function是物件的一種子類型，function在JS中被稱為`first class`類型，因為他們基本上就是普通的物件而且可以被當作其他物件一樣處理，而陣列也是一種形式的物件，他在內容的組織的結構化上會比一般物件好。

## Built-in Objects
還有其他物件子類別通常稱為`內置物件`，它們的名稱看起來和它們對應的基本類型有聯繫，但事實上它們的關係更複雜。
- String
- Number
- Boolean
- Object
- Function
- Array
- Date
- RegExp
- Error
但是在JS中這些都只是內建的函數，他們都可以通過new來創建出來，而創建出來的是一個新的子類型constructed物件。
```javascript
var strPrimitive = "I am a string";
typeof strPrimitive; // "string"
strPrimitive instanceof String;	// false

var strObject = new String( "I am a string" );
typeof strObject; // "object"
strObject instanceof String; // true
```
基本類型的"I am a string"他是一個不可變的字串而不是物件，為了對這個字串進行操作（檢查長度...）就會需要將這個字串變為物件形式，但是幸運的事JS對於這種情況，他會在需要的時候`自動的`將`"string"`強制轉換為`String`類型（auto-boxing），這意味著你不需要明確的創建這個字串的物件就可以對他進行操作。
```javascript
var strPrimitive = "I am a string"; // type -> "string"

// auto transform to String(object)
console.log( strPrimitive.length );	// 13
console.log( strPrimitive.charAt( 3 ) ); // "m"
```
對於這種自動轉換型別的也發生在number與boolean，但是`null與undefined沒有物件形式`，他們只有自己的基本類型，Date只能透過new建立所以他沒有基本型態。

無論使用declaration還是constructed建立`Objects`、`Array`、`Function`、`RegExps`他們都是物件。

`Error`很少明確且直接的被創建出來，通常在有異常的時候自動被創建並且擲出，可以由`new Error(...)`建立出來不過很少見。


# Contents
物件的內容會儲存在物件中某些特定命名的位置上，我們稱這些儲存在物件中的值為`properties`。

雖然我們說內容是存在於物件之中，但其實這只是一種`看起來`而已，對JS來說他是以`依賴（implementation-dependent）`的方式儲存並且很有可能`不將內容儲存在物件容器中`，`只有這些properties的名稱儲存在容器`，而這些properites名稱會當作指向儲存內容的位置的指針，換句話說儲存在物件容器內的properties名稱是`物件內容的Reference`。
```javascript
var myObject = {
	a: 2
};

myObject.a;	// 2
myObject["a"]; // 2
```
若要放問到myObject中的a需要使用`.`或`[]`運算符，`.a`通常用於取得物件的property，而`["a"]`用於鍵(key)的訪問，實際上這兩個用法訪問到的位置是相同的所以都可以使用。

兩種訪問最主要的區別在於，如果使用`.`則後面需要一個`兼容標識符（Identifier）`的屬性名稱，而`[".."]`中則可以接收任何兼容UTF-8/unicode的字串作為屬性名，舉個例子若你的物件中有個`Super-Fun!`屬性，就只能使用["Super-Fun!"]來訪問這個屬性，因為他不是一個合法的標識符(Identifier)。

由於`["..."]`是使用`字串`所以可以在程式中動態的變更我們需要訪問的位置
```javascript
var wantA = true;
var myObject = {
	a: 2,
    b: 3
};

var idx;

if (wantA) {
	idx = "a";
}
console.log( myObject[idx] ); // 2
```

由於物件的屬性`必須`得是字串，所以當你填入非字串的屬性名則`會優先將它轉變為字串`，轉換的範圍甚是包誇number。
```javascript
var myObject = { };

myObject[true] = "foo";
myObject[3] = "bar";
myObject[myObject] = "baz";

myObject["true"];				// "foo"
myObject["3"];					// "bar"
myObject["[object Object]"];	// "baz"
```

## Computed Property Names
使用`["..."]`還可以對鍵(key)進行操作，比如說Object[prefix + name]。
```javascript
var prefix = "foo";

var myObject = {
	[prefix + "bar"]: "hello",
	[prefix + "baz"]: "world"
};

myObject["foobar"]; // hello
myObject["foobaz"]; // world
```

## Property vs. Method
對於在物件中的function來說，許多開發者將他與property區分開來，我們稱它為method，因為以技術上來說function他其實是`不屬於物件`，他在物件中只是以一個Reference的形式儲存，所以當物件訪問一個function的時候就很像是一個`方法（method）`，雖然這是一個滿牽強的理由ＸＤ。

## Array
陣列也使用`[]`來訪問其中的元素，但是陣列在儲存值以及儲存位置的結構上更具有組織性，陣列採用`數字索引`這意味著這個元素被儲存的位置，必須是一個`非負整數`。
```javascript
var myArray = [ "foo", 42, "bar" ];

myArray.length;		// 3
myArray[0];			// "foo"
myArray[2];			// "bar"
```

在上面有提到其實陣列也是一種物件，所以你也可以對這個陣列增加屬性
```javascript
var myArray = [ "foo", 42, "bar" ];
myArray.baz = "baz";

myArray.length;	// 3
myArray.baz;	// "baz"
```
雖然陣列可以達到與物件一樣的效果（增加鍵/值）但是不推薦做這種操作，因為陣列本身有他的用途與使用方法，所以建議用物件來儲存鍵/值而不適用陣列。

還有一個直得注意的地方，雖然我們對`myArray`添加了屬性，但是可以發現`myArray.length`的長度並沒有被改變，`但是如果在陣列的屬性中添加的值看起來像個數字，則他最終會變成陣列的索引`。
```javascript
var myArray = [ "foo", 42, "bar" ];

myArray["3"] = "baz";
myArray.length;	// 4
myArray[3];		// "baz"
```
除了會更改陣列長度之外，如果添加的數字屬性名是已經存在於陣列中的index，則會改變其陣列的內容
```javascript
var myArray = [ "foo", 42, "bar" ];

myArray["1"] = "baz";
myArray.length;	// 3
myArray[1];		// "baz"
```

## Duplicating Objects
在我們創建了一個物件時，可能會面臨到需要複製物件的情況，一開始可能會覺得就單純將這個物件複製過去（就跟一般的value一樣），但是其實JS的物件複製比這個來的複雜多了，要介紹物件的複製首先要先區分`淺拷貝`與`深拷貝`的區別。

### 淺拷貝
對於複製物件來說，`obj1 = obj2`他所傳遞的不是obj2的值而是obj2的`Reference`，這意味著他們是共用同一個記憶體空間，所以`當一個更改了另一個也會被影響而一同變更`。
```javascript
var obj1 = { a: 10, b: 20, c: 30 };
var obj2 = obj1;
obj2.b = 100;

console.log(obj1); // { a: 10, b: 100, c: 30 } <-- b 被改到了
console.log(obj2); // { a: 10, b: 100, c: 30 }
```
![https://ithelp.ithome.com.tw/upload/images/20201029/201247676vgYevTMVo.png](https://ithelp.ithome.com.tw/upload/images/20201029/201247676vgYevTMVo.png)
(圖片來源 : [[Javascript] 關於 JS 中的淺拷貝和深拷貝](https://larry850806.github.io/2016/09/20/shallow-vs-deep-copy/))

### 深拷貝
深拷貝與淺拷貝的只複製Reference不同，他會`創造`一個新的物件，新物件與舊物件不會共用一個記憶體空間，所以修改新物件不會同步影響到舊物件。
```javascript
var obj1 = { a: 10, b: 20, c: 30 };
var obj2 = { a: obj1.a, b: obj1.b, c: obj1.c };
obj2.b = 100;

console.log(obj1); // { a: 10, b: 20, c: 30 } <-- b 沒被改到
console.log(obj2); // { a: 10, b: 100, c: 30 }
```
![https://ithelp.ithome.com.tw/upload/images/20201029/20124767YzWOPgJA1n.png](https://ithelp.ithome.com.tw/upload/images/20201029/20124767YzWOPgJA1n.png)
(圖片來源 : [[Javascript] 關於 JS 中的淺拷貝和深拷貝](https://larry850806.github.io/2016/09/20/shallow-vs-deep-copy/))

介紹完什麼是淺拷貝與深拷貝後，我們回到本書
```javascript
function anotherFunction() { /*..*/ }

var anotherObject = {
	c: true
};
var anotherArray = [];

var myObject = {
	a: 2,
	b: anotherObject,	// reference, not a copy!
	c: anotherArray,	// another reference!
	d: anotherFunction
};

anotherArray.push( anotherObject, myObject );
```
將過介紹什麼是深淺拷貝後，可以發現`myObject`中的b、c、d都不是物件的複製而只是共用了相同的Reference（淺拷貝），如果要將b、c、d深拷貝給myObject，有一個解決方法就是使用JSON，將物件使用`JSON.stringify`轉變為字串後再用`JSON.parse`轉回物件，這樣就可以得到一個深拷貝。
```javascript
var obj1 = { body: { a: 10 } };
var obj2 = JSON.parse(JSON.stringify(obj1));
obj2.body.a = 20;

console.log(obj1); // { body: { a: 10 } } <-- 沒被改到
console.log(obj2); // { body: { a: 20 } }
console.log(obj1 === obj2); // false
console.log(obj1.body === obj2.body); // false
```

還有另一種深拷貝的方法，ES6提供了一個新函數`Object.assign`。
```javascript
var obj1 = { a: 10, b: 20, c: 30 };
var obj2 = Object.assign({}, obj1);
obj2.b = 100;

console.log(obj1); // { a: 10, b: 20, c: 30 } <-- 沒被改到
console.log(obj2); // { a: 10, b: 100, c: 30 }
```
`Object.assign({},obj1)`的第一個參數{}代表他會建立一個空的物件，接著再把obj1中的properties複製過去，所以obj2會長得跟obj1一樣但是卻不是共用同一個記憶體位置，不過要注意的是`Object.assign只能複製一層`的物件。

除了使用ES6提供的`Object.assign`之外，也可以使用ES6提供的`...(展開運算子spread operator)`將obj1的物件複製到空物件中
```javascript
var obj1 = { a: 10, b: 20, c: 30 };
var obj2 = {
    ...obj1, // 展開obj1並複製
    b: 100, // 更改obj2中的值
}

console.log(obj1); // { a: 10, b: 20, c: 30 } <-- 沒被改到
console.log(obj2); // { a: 10, b: 100, c: 30 }
```


# 結論
在本章節中我們介紹了物件是什麼、型態與一些特性，讓我們來整理一下
- 物件可以透過declaration與constructed建立出來，他們的結果是一樣的，唯一的差別是`declaration建立的物件可以一次性的加入一個或多個數性`，而`constructed只能一個一個加入`。
- 物件的型態
    - String：隨著需要而將原始型態轉變為物件。
    - Number：隨著需要而將原始型態轉變為物件。
    - Boolean：隨著需要而將原始型態轉變為物件。
    - Object：無論使用declaration或constructed建立，都是物件。
    - Function：無論使用declaration或constructed建立，都是物件。
    - Array：無論使用declaration或constructed建立，都是物件。
    - Date：只能通過constructed建立。
    - RegExp：無論使用declaration或constructed建立，都是物件。
    - Error：可以通過constructed建立，但不常見。
- 可以使用`.`與`["..."]`訪問到物件的properties，他們的區別在於`.`需要符合Identifier而`["..."]`只要是UTF-8/unicode的字串都可以。
- 物件中的值稱為`property`，函數稱為`method`。
- 陣列也是物件，所以也可以對陣列加入屬性（不建議）。
- 對陣列加入屬性，`如果屬性名稱是數字則會改變陣列index的情況`。
- 物件的複製有分`深拷貝`與`淺拷貝`，淺拷貝只是複製物件的Reference所以是共用同一個記憶體位置; 深拷貝是創造一個新的物件。


# Reference
- [You Don't Know JavaScript](https://github.com/getify/You-Dont-Know-JS/blob/1st-ed/this%20%26%20object%20prototypes/ch3.md)
- [[Javascript] 關於 JS 中的淺拷貝和深拷貝](https://larry850806.github.io/2016/09/20/shallow-vs-deep-copy/)