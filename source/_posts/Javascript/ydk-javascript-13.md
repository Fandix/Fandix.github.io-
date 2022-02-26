---
title: You Don't Know JavaScript [This & Object Prototypes] - This All Makes Sense Now! [下]
date: 2020-10-29 17:57:42
tags:
- Javascript
- Front-end
- This & Object Prototypes

categories:
- You Don't Know JavaScript
---

在[this All Makes Sense Now! [上]](https://ithelp.ithome.com.tw/articles/10253946)中我們介紹了什麼是call-site與4種綁定的規則，我們需要做的就是觀察一個程式找到他的call-site並了解他適用於哪個規則，這樣就可以找到 this 所指向的是什麼了。

<img src="https://www.sciencealert.com/images/2022-01/processed/WomanWithBlueAndPurpleNebulaOverlaidHerHead_600.jpg" alt="drawing" width="860"/>

<!-- more -->

# Everything In Order
透過找到call-site與適用的規則可以找到this所指向的地方，但是如果一個call-site符合多個規則那怎麼辦？這些規則具有`優先順序`，接下來我們將介紹這些規則的順序，首先要先了解default binding是所有規則中優先級最低的，所以先不討論。

我們先討論一下是implicit binding還是explicit binding?
```javascript
function foo() {
	console.log( this.a );
}

var obj1 = {
	a: 2,
	foo: foo
};

var obj2 = {
	a: 3,
	foo: foo
};

obj1.foo(); // 2
obj2.foo(); // 3

obj1.foo.call( obj2 ); // 3
obj2.foo.call( obj1 ); // 2
```
上面的程式碼中可以看到當對function使用顯性綁定後，`顯性綁定的規則會高於隱性綁定`，這意味著當你需要使用隱性綁定的時候需要檢查funciton是否有被顯性綁定住。

接下來要看看`new binding`的優先級是如何。
```javascript
function foo(something) {
	this.a = something;
}

var obj1 = {
	foo: foo
};

var obj2 = {};

obj1.foo( 2 );
console.log( obj1.a ); // 2

obj1.foo.call( obj2, 3 );
console.log( obj2.a ); // 3

var bar = new obj1.foo( 4 );
console.log( obj1.a ); // 2
console.log( bar.a ); // 4
```
new binding的優先級是比隱性綁定高的，但是要如何確定new binding與顯性綁定誰的優先極高呢？
```javascript
function foo(something) {
	this.a = something;
}

var obj1 = {};

var bar = foo.bind( obj1 );
bar( 2 );
console.log( obj1.a ); // 2

var baz = new bar( 3 );
console.log( obj1.a ); // 2
console.log( baz.a ); // 3
```
bar強制綁定著obj1，之後使用new binding將bar中函數的this指向這個物件，但是結果卻是用new創出來的baz沒有改變到obj1.a的值，因為使用new binding後回傳的會是一個新的物件並將函數的this指向這個新物件，所以他與bar中綁定的obj1是沒關係的。

## Currying
> Currying（柯里化），又稱為 parital application 或 partial evaluation，是個「將一個接受 n 個參數的 function，轉變成 n 個只接受一個參數的 function」的過程。

bind(...)可以將輸入的參數（需綁定物件後面的參數）默認的當作前函數的標準參數
```javascript
function foo(p1,p2) {
	this.val = p1 + p2;
}

var bar = foo.bind( null, "p1" ); // 將"p1"默認的當作每次呼叫foo的參數
var baz = new bar( "p2" );
var bax = new bar( "p3" );

baz.val; // p1p2
bax.val; // p1p3
```

## Determining this
了解了綁定的優先級，我們可以總結一下判斷this的規則。
1. 如果是函數使用new binding那麼this所指向的便是`被創建出來的新物件`。
```javascript 
var bar = new foo(); // this指向bar 
```
2. 函數通過顯性綁定(call或apply)或是bind的硬性綁定，則this指向被綁定的物件。
```javascript
var bar = foo.call(obj2); //this指向obj2
```
3. 函數通過環境物件而被調用（隱性綁定），則this指向呼叫function reference的環境物件。
```javascript
const = obj1 = {
    foo: foo
}
obj1.foo();
```
4. 不符合以上條件則屬於default binding，this在非嚴格模式下指向全域物件，否則是undefined。

# Binding Exceptions
凡事均有特例，對於this的判定也不例外。

## Ignored this
如果傳遞`null`或`undefined`給call,apply或bind的參數，那麼這些值就會被忽略綁定規則會自動回到default binding。
```javascript
function foo() {
	console.log( this.a );
}

var a = 2;
foo.call( null ); // 2
```
既然傳遞非法的參數（null,undefined）會造成binding的忽略那麼為什麼會需要使用？

在ES6中我們可以使用`擴展運算符(...)`來將陣列中的元素拆開，但是在ES6之前要達到同樣的效果需要使用`apply`來來達到。
```javascript
// ES5
function foo(a,b) {
	console.log( "a:" + a + ", b:" + b );
}

// spreading out array as parameters
foo.apply( null, [2, 3] ); // a:2, b:3

// ES6
foo(...[2, 3]); // a:2, b:3
```
將null當作參數傳遞給bind(...)可以達到currying的功能。
```javascript
function foo(a,b) {
	console.log( "a:" + a + ", b:" + b );
}

var bar = foo.bind( null, 2 );
bar( 3 ); // a:2, b:3
```

## Indirection
若是創建了函數的間接引用(indirect reference)，這種情況下會失去原本的bind而導致變回適用default binding。
```javascript
function foo() {
	console.log( this.a );
}

var a = 2; // declaration in global
var o = { a: 3, foo: foo };
var p = { a: 4 };

o.foo(); // 3
(p.foo = o.foo)(); // 2
```
對於p.foo = o.foo來說，雖然看起來像是將o物件中的foo(...)賦予給p物件，但實際上p所拿到的是foo(...)本體的reference，代表著他的位置與o物件中的foo是不一樣的，所以只適用default binding。


# Lexical this
對於普通函數來說我們可以遵守4條綁定規則中找到this所指向的位置，但是ES6引入了一個不適用於這些規則的函數`箭頭函數`。

箭頭函數不採用4個標準的綁定規則而是從封閉的（函數或全域）範圍採用this綁定。
```javascript
function foo() {
	// return an arrow function
	return (a) => {
		// `this` here is lexically adopted from `foo()`
		console.log( this.a );
	};
}

var obj1 = {
	a: 2
};
var obj2 = {
	a: 3
};

var bar = foo.call( obj1 );
bar.call( obj2 ); // 2, not 3!
```
在foo(...)中創建一個箭頭函數，這個箭頭函數會自動綁定foo()被調用時的this，以上面的例子來說當foo(...)被呼叫並且this被綁定為obj1，那麼箭頭函數中的this也會綁定obj1，而且`箭頭函數的綁定是不能被覆蓋的`。

最常見的用法式將箭頭函數應用在callback function中
```javascript
function foo() {
	setTimeout(() => {
		// `this` here is lexically adopted from `foo()`
		console.log( this.a );
	},100);
}

var obj = {
	a: 2
};
var a = 4; // declaration in global

foo.call( obj ); // 2
```
使用箭頭函數當作callback function的參數傳遞給setTimeout並不會像我們在[this All Makes Sense Now! [上]](https://ithelp.ithome.com.tw/articles/10253946)提到的，因為隱性賦值而產生binding遺失，箭頭函數的this會在foo()被呼叫的時候就指定給foo()所綁定的物件。

其實這種綁定的方式在ES6的箭頭函數出來之前就有類似的方式了
```javascript
function foo() {
	var self = this; // lexical capture of `this`
	setTimeout( function(){
		console.log( self.a );
	}, 100 );
}

var obj = {
	a: 2
};
var a = 4; // declaration in global

foo.call( obj ); // 2
```
透過在一進function後就先捕獲foo中this所指向的物件，這樣就不會因為隱性賦值的問題導致binding遺失。

# 結論
當我們需要找到函數中this所指向的位置，我們需要找到這個函數的`call-site`與看這個函數符合4種綁定規則的哪一種。
1. 通過`new`調用funciton則this綁定`新創建的物件`。
2. 通過`call`、`apply`或`bind`調用則this綁定`指定的物件`。
3. 通過環境物件調用函數reference則this綁定`呼叫函數的環境物件`。
4. default，嚴格模式下是`undefined`否則this指向`全域物件`。

ES6提供的箭頭函數不遵守上面的4個綁定規則，箭頭函數的this綁定取決於他被創建的當下所綁定的物件。

# Reference
- [You Don't Know JavaScript](https://github.com/getify/You-Dont-Know-JS/blob/1st-ed/this%20%26%20object%20prototypes/ch2.md)