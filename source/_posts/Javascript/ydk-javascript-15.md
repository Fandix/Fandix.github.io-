---
title: You Don't Know JavaScript [This & Object Prototypes] - Object [下]
date: 2020-11-01 21:42:07
tags:
- Javascript
- Front-end
- This & Object Prototypes

categories:
- You Don't Know JavaScript
---

在[Object [上]](https://ithelp.ithome.com.tw/articles/10254000)中我們介紹了物件的宣告、型態、拷貝等等特性，接下來我們繼續介紹物件中都有哪些特性。

<img src="https://www.smashingfonts.com/thumbs728/14183_0.jpg" alt="drawing" width="860"/>

<!-- more -->

# Property Descriptors
在ES5之前JS沒有提供一個可以區分物件屬性特徵的方法，但在ES5中可以透過`property descriptor`觀察物件屬性的特性。
```javascript
var myObject = {
	a: 2
};

Object.getOwnPropertyDescriptor( myObject, "a" );
// {
//    value: 2,
//    writable: true,
//    enumerable: true,
//    configurable: true
// }
```
我們對`myObject`中的屬性a使用`getOwnPropertyDescriptor`，可以得到除了value的另外三個特質：`writable`、`enumerable`、`configurable`。

既然我們可以拿到這個數性的特質，那們我們也可透過`Object.defineProperty(...)`來添加新的特性或是修改原本的特性。
```javascript
var myObject = {};

Object.defineProperty( myObject, "a", {
	value: 2,
	writable: true,
	configurable: true,
	enumerable: true
} );

myObject.a; // 2
```
一班如果你要加入一個正常的屬性到物件中是不必要使用`Object.defineProperty`這個方法，除非你想新增特殊型態的屬性。

## Writable
此特性代表你是否能夠更改屬性的值。
```javascript
var myObject = {};

Object.defineProperty( myObject, "a", {
	value: 2,
	writable: false, // not writable!
	configurable: true,
	enumerable: true
} );

myObject.a = 3; // can't re-assignment
myObject.a; // 2
```
對於myObject中的a屬性設定為`不可寫入`則我們無法再次賦值給他，如果這個程式在嚴格模式下執行，會擲出`TypeError`。
```javascript
"use strict";

var myObject = {};

Object.defineProperty( myObject, "a", {
	value: 2,
	writable: false, // not writable!
	configurable: true,
	enumerable: true
} );

myObject.a = 3; // TypeError : Cannot change a non-writable property.
```

## Configurable
Configurable特性控制著我們能否使用`defineProperty(...)`重新改變屬性的特性。
```javascript
var myObject = {
	a: 2
};

myObject.a = 3;
myObject.a;					// 3

Object.defineProperty( myObject, "a", {
	value: 4,
	writable: true,
	configurable: false,	// not configurable!
	enumerable: true
} );

myObject.a;					// 4
myObject.a = 5;
myObject.a;					// 5

Object.defineProperty( myObject, "a", {
	value: 6,
	writable: true,
	configurable: true,
	enumerable: true
} ); // TypeError
```
一旦將物件屬性的`configurable`設定為false後，就算不是處於嚴格模式下，再次嘗試更改數性的設置就會擲出TypeError，這意味著`當你設置屬性的configurable為false後就不可逆轉`。

當你將這個屬性的`configurable`設定為false後可以避免這個屬性被`delete`操作符移除。
```javascript
var myObject = {
	a: 2
};

myObject.a;				// 2
delete myObject.a;
myObject.a;				// undefined

// configurable:false
Object.defineProperty( myObject, "a", {
	value: 2,
	writable: true,
	configurable: false,
	enumerable: true
} );

myObject.a;				// 2
delete myObject.a;
myObject.a;				// 2
```
在JS中`delete`操作符僅用於移除物件的屬性，如果一個物件中的屬性引用的是某一個物件/函數的`最後一個現存引用`，那們當你delete這個屬性那們就`代表移除了最後一個引用`，而這個沒有被任何地方引用的物件/函數則會被回收掉，但如果還有其他地方引用則只是`單純地從這個物件中移除他們的Reference`總結來說`delete僅僅是移除物件的屬性而已`。

## Enumerable
Enumerable特性決定了我們是否可以對這個元素進行列舉操作（for...in），如果將這個特性設定為false則for...in loop終究無法讀取到這個屬性
```javascript
Object.defineProperty( myObject, "a", {
	value: 2,
	writable: true,
	configurable: ture,
	enumerable: false
} );

myObject.b = 2;
myObject.b = 4;

for(let prop in myObject){
    console.log(prop); // disable property a
}
// b
// c
```

# Immutability
有時候我們在開發專案的時候會需要建立一些不希望被更改的物件，ES5中提供了一些方法可以讓我做到讓物件保持不被更改，但是直得注意的是：這些方法都是`淺層不變性`，這意味著他只會固定住本體物件和他的直屬屬性性質，但是如果這個被固定住的物件中有引用其他物件的屬性(陣列、物件、函數)，那麼這些被Reference的物件屬性並`不會跟著被鎖定住`。

## Object Constant
通過將物件屬性`writable`與`configurable`都設定為false讓我們的物件保持不能被更改的狀況。
```javascript
var myObject = {};

Object.defineProperty( myObject, "FAVORITE_NUMBER", {
	value: 42,
	writable: false,
	configurable: false
} );
```

## Prevent Extensions
如果要禁止添加屬性到我們的物件中，可以使用`Object.preventExtensions(...)`。
```javascript
var myObject = {
	a: 2
};

Object.preventExtensions( myObject );

myObject.b = 3;
myObject.b; // undefined
```
在非嚴格模式下b的屬性添加會失敗，否則會擲出TypeError。

## Seal
使用`Object.seal(..)`創建一個被`封印`的物件，實際上是將這個物件使用`Object.preventExtensions(...)`固定住不允許加入其他屬性在對物件中的每一個屬性標記為`configurable:false`讓物件中的元素也不能被刪除或重新配置，但是`可以對現有存在的值進行修改`。

## Freeze
使用`Object.freeze(...)`創建一個`凍結`的物件，實際上是將物件使用`Object.seal(..)`使他不能添加、刪除、重新配置新的屬性，再對每個屬性使用`writable:false`讓物件屬性的值不能被更改，這個方法是`最高級別`的不可變性。


# [[Get]]
關於對於物件屬性的訪問有個重要的細節。
```javascript
var myObject = {
	a: 2
};

myObject.a; // 2
```
使用`myObject.a`來訪問物件中的a屬性，但這個操作不只是在物件中查找名稱為a屬性，實際上他會先在`myObject`上執行一個`[[Get]]`操作，他會檢查物件並在物件中尋找有沒有這個被請求的屬性名稱，如果找到就返回相對應的值，如果沒能在物件中找到屬性則會做另一個重要的事情（遍歷[prototype]鏈），如果通過任何方法都找不到這個屬性的時候便會返回`undefined`。
```javascript
var myObject = {
	a: 2
};

myObject.b; // undefined
```

# [[Put]]
既然從一個屬性中取得值存在一個內部定義的[[Get]]方法，那麼也會有一個內部定義的[[Put]]。

調用[[Put]]時會根據幾個因素表現不同的行為，影響最大的辨識`屬性是否已經在存在於物件`，如果屬性存在[[Put]]會檢查：
1. 如果這個屬性是`訪問器描述符號（Getters/Setters）`中的Setters，則直接調用Setters。
2. 如果這個屬性的`writable`是false，在非嚴格模式下無聲的失敗，否則擲出TypeError。
3. 不符合以上兩個則像正常賦值一樣設置屬性。

如果這個屬性不存在於物件中則[[Put]]操作會更複雜，我們在後面會詳細講解。

# Getters & Setters
ES5中新增了一個方法來覆蓋這些默認操作的一部分，不是針對每個屬性而不是物件，就是通過`getters`和`setters`，Getter 是用來`取得指定屬性的值的方法`而Setter是用來`設定指定屬性的值的方法`。

```javascript
var myObject = {
	// define a getter for `a`
	get a() {
		return 2;
	}
};

myObject.a = 3;
myObject.a; // 2
```
我們僅為a定義了一個getter，就算之後向再次對a賦值也不會成功，他會`無聲的將賦予的值丟棄而不會擲出錯誤`，我們可以多新增一個setters讓我們可以更改a的值。
```javascript
var myObject = {
	// define a getter for `a`
	get a() {
		return this.b;
	},

	// define a setter for `a`
	set a(val) {
		this.b = val * 2;
	}
};

myObject.a = 2;
myObject.a; // 4
```
當我們重新賦予值給a，他會調用myObject中的set(...)，他將我們給的值*2後暫存在一個變量b，之後再透過get(...)將這個值傳回給a。


# Existence 
我們可以透過`in`操作符來確認我的數性是否有存在在物件中。
```javascript
var  myObject  =  { 
	a : undefined 
} ;

console.log(myObject.a); // undefined
console.log(myObject.b); // undefined

( "a"  in  myObject ) ; 				// true 
( "b"  in  myObject ) ; 				// false

myObject . hasOwnProperty (  "a"  ) ; 	// true 
myObject . hasOwnProperty (  "b"  ) ; 	// false
```
雖然`myObject.a`與`myObject.b`都是undefined，但是通過`in`和`hasOwnProperty(...)`就可以確認數性是否存在於物件當中。

# Enumeration
前面我們在介紹屬性的特性的時候有稍微介紹什麼是`enumerable`，現在我們來更仔細的介紹一下這個特性。
```javascript
var myObject = { };

Object.defineProperty(
	myObject,
	"a",
	{ 
        value: 2,
        enumerable: true,  // make `a` enumerable, as normal
    }
);

Object.defineProperty(
	myObject,
	"b",
	{ 
        value: 3
        enumerable: false, // make `b` NON-enumerable 
    }
);

myObject.b; // 3
("b" in myObject); // true
myObject.hasOwnProperty( "b" ); // true

// .......

for (let prop in myObject) {
	console.log( prop );
}
// "a"
```
我們在myObject中建立了兩個屬性(a與b)，我們使用`hasOwnProperty（...）`確認b是確實存在於myObject中，但是由於他的`enumerable`設定為false，所以使用for...in無法將它打印出來。

除了使用for...in測試數性是否為enumerable，也可以使用`propertyIsEnumerable(..)`直接測試這個屬性是否存在於物件中並且他的enumerable為何。
```javascript
var myObject = { };

Object.defineProperty(
	myObject,
	"a",
	{ 
        value: 2,
        enumerable: true,  // make `a` enumerable, as normal
    }
);

Object.defineProperty(
	myObject,
	"b",
	{ 
        value: 3
        enumerable: false, // make `b` NON-enumerable 
    }
);

myObject.propertyIsEnumerable( "a" ); // true
myObject.propertyIsEnumerable( "b" ); // false
myObject.propertyIsEnumerable( "c" ); // false

Object.keys( myObject ); // ["a"]
Object.getOwnPropertyNames( myObject ); // ["a", "b"]
```
也可以使用`Object.keys(..)`他會返回所有`enumerable = ture`的屬性。


# Iteration
在陣列中迭代所有值得方法是使用標準的`for loop`
```javascript
var myArray = [1, 2, 3];

for (let i = 0; i < myArray.length; i++) {
	console.log( myArray[i] );
}
// 1 2 3
```
上面的程式中並不是迭代了陣列中的值，而是迭代了`陣列的index`，而我們透過迭代獲得的index取得陣列中對應的數（muyArray[i]）。

在ES5中提供了一些對於陣列的迭代方法，`forEach(...)`、`every(...)`、`some(...)`。
- forEach(...)會迭代陣列中的所有值並將這些值分別帶入callback function中
- every(...)會持續迭代到最後或著return一個`false`。
- some(...)會持續迭代到最後或著return一個`true`。

如果想直接得到迭代值而不是陣列的index，在ES6中提供了一個新語法`for...of`它可以用來迭代陣列或物件。
```javascript
const myArray = [1, 2, 3];

for(let val of myArray){
    console.log(val);
}
// 1
// 2
// 3
```
for...of被要求要迭代的東西需要提供一個`迭代器物件`，每一次循環中都會調用這個迭代器物件的`next(...)` method 。

在陣列中擁有內建的迭代器物件（@@iterator）所以可以輕易地使用for...of進行迭代，我們也可手動建立一個@@interator來看看他是如何運作的。
```javascript
var myArray = [ 1, 2, 3 ];
var it = myArray[Symbol.iterator]();

it.next(); // { value:1, done:false }
it.next(); // { value:2, done:false }
it.next(); // { value:3, done:false }
it.next(); // { done:true }
```
迭代器的next(...) return 一組物件{value: ... , done: ...}，物件中的value屬性代表當前迭代著值而done則代表是否迭代完成。


# 結論
本章節中我們介紹了物件更多的特質，屬性的特性、如何將物件變為不可變、物件中的取值與賦值等等，我們來做一些總結：
- 物件中的數性都有三種特性：
    - writable：控制這個屬性是否能被更改。
    - Configurable：控制這個屬性是否能更改特性配置。
    - Enumerable：控制這個屬性是否能被迭代。
- 當我們需要建立一個不可變的物件時，可以透過以下辦法達到：
    - 將屬性的 writable與configurable 設定為false，讓這個屬性不能被更改。
    - 使用Object.preventExtensions(...)可以防止物件添加新屬性。
    - 使用Object.seal(...)可以防止添加新屬性並且將數性標記為configurable:false（即不可更改屬性設置），`但是可以更改現有的屬性的值`。
    - 使用Object.freeze(...)可以建立最高級別的不可變物件，即不可新增屬性也不可更改現有屬性。
- 當訪問物件中的屬性則會調用[[Get]]，會在物件中找到這個屬性的名字，若找到則回傳相對應的值，若沒有則回傳undefined。
- 當設定物件的屬性會調用[[Put]]：
    - 當屬性是`setter`則調用setter。
    - 當這個屬性的`writable = false`，在嚴格模式下擲出TypeError否則無聲的失敗。
    - 不符合上面兩點則正常賦值。
- Getter 是用來`取得指定屬性的值的方法`而Setter是用來`設定指定屬性的值的方法`。
- 可以使用`in`與`hasOwnProperty(...)`檢查屬性是否存在於物件中。
- 可以使用`propertyIsEnumerable(...)`測試數性是否存在於物件中別且檢查這個屬性使否是可迭代的。
- 可以使用`Object.keys(...)`他將會返回物件中所有可迭代的屬性。
- 可以使用`for...of`迭代陣列中的值。
- for...of迭代實際上是執行陣列中內建`@@iterator`的`next(...)`。

# Reference
- [You Don't Know JavaScript](https://github.com/getify/You-Dont-Know-JS/blob/1st-ed/this%20%26%20object%20prototypes/ch3.md)