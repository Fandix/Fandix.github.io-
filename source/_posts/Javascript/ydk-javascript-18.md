---
title: You Don't Know JavaScript [This & Object Prototypes] - Prototypes [下]
date: 2020-12-20 16:35:22
tags:
- Javascript
- Front-end
- This & Object Prototypes

categories:
- You Don't Know JavaScript
---

我們在[Prototypes [上]](https://ithelp.ithome.com.tw/articles/10254732)中介紹了什麼是Prototype也介紹了JavaScript中類似Class的機制，而本章將會介紹類似`繼承`的機制。

<img src="https://uxguerrilla.co.uk/wp-content/uploads/2018/03/prototyping.png" alt="drawing" width="860"/>

<!-- more -->

# (Prototypal) Inheritance
我們在上一章中提介紹了一個範例
```javascript
function Foo(name) {
	this.name = name;
}

Foo.prototype.myName = function() {
	return this.name;
};

var a = new Foo( "a" );

a.myName(); // "a"
```
我們透過new關鍵字建立出Object a並將它的Prototype鏈接到Foo以獲得myName這個method，這就是原型繼承機制。

![https://ithelp.ithome.com.tw/upload/images/20201217/20124767YjfAL1WY8B.png](https://ithelp.ithome.com.tw/upload/images/20201217/20124767YjfAL1WY8B.png)
(圖片來源 : [You Don't Know JavaScript](https://github.com/getify/You-Dont-Know-JS/blob/1st-ed/this%20&%20object%20prototypes/ch5.md))

透過上面的圖展現了a1到Foo.prptotype的委派和Bar.prototype到Foo.prototype的委派，某種程度上來說類似於Parent-Child繼承的概念，但是這些箭頭的方向代表著是`鏈接而不是複製`，我們將此上圖的鏈接轉換為程式碼並進行講解。
```javascript
function Foo(name){
    this.name = name;
};

// 委派myName method到Foo.prototype上
Foo.prototype.myName = function(){
    return this.name;
};

function Bar(name, label){
    Foo.call(this, name);
    this.label = label;
};

// link Bar.prototype to Foo.prototype
Bar.prototype = Object.create( Foo.prototype );

// 委派myLabel method到Bar.prototype上
Bar.prototype.myLabel = function(){
    return this.label;
};

const a = new Bar('a', 'Obj a');
a.myName(); // a
a.myLabel(); // Obj a
```

值得注意的是，上面程式碼中`Bar.prototype = Object.create( Foo.prototype );`的操作，它代表著建立一個新的Object並將該Object內部的Prototype鏈接到指定的對象(Foo.prototype)，也可以說是`建立一個新的Bar.prototype並將他鏈接到Foo.prototype（取代原本Bar.prototype）`。

這麼做是因為，由於我們在建立Bar這個函數的時候JavaScript會自動幫我們在Bar中建議一個.prototype的屬性，但他卻沒有鏈接到Foo.prototype，所以我們需要將原本的丟棄並建立一個新的讓他鏈接到Foo.prototype上。

除了使用`Object.create(...)`之外，也可以使用下面兩種方法也可以使用但是可能並不會像我們預期的那樣。
```javascript
// donsn't work like you want
Bar.prototype = Foo.prototype

// works but with side-effects
Bar.prototype = new Foo();
```
1. 使用`Bar.prototype = Foo.prototype`不會和我們預期的行為一樣，這樣Bar.prototype是對、Foo.prototype的Reference（淺拷貝），代表Bar和Foo都鏈接到同一個Foo.prototype Object上，這意味著如果在Bar.prototype上添加了一個method時也會同時修改到Foo.prototype。
2. 雖然使用`Bar.prototype = new Foo();`也可以創建一個新的Object並且也可以鏈接到Foo.prototype上，但是如果使用Foo(...)構造函數執行這個操作的話會有副作用（Foo()會被調用產生預期外的行為）。

所以我們使用Object.create(...)來創建一個新的物件並鏈接到Foo.prototype且沒有調用Foo的副作用，但缺點是我們必須先建立一個新Object之後再將他的prototype給拋棄，不過在ES6中提供了一個method(`Object.setPrototypeOf(...)`)可以讓我們直接修改現有Object的prototype鏈接。
```javascript
Object.setPrototypeOf( Bar.prototype, Foo.prototype );
```

## Inspecting "Class" Relationships
如果在JavaScript中想找到一個Object他委託到哪一個Object上，我們該怎麼做？在傳統的OOP環境下這個行為稱為`introspection`或`reflection`。
```javascript
function Foo(){
    // ...
}
Foo.prototype.blah = ...;

const a = new Foo();
```
### 方法一：使用instanceof
```javascript
a instanceof Foo; // true
```
它代表著`a在整個[[Prototype]]鏈中，有沒有出現在那個被Foo.prototype所指向的Object中？`，但是如果你有兩個任意Object，你想調查這兩個Object通過[[Prototype]]鏈相互關聯，單靠instanceof是無法做到的。

### 方法二：使用[[Prototype]] reflection
```javascript
Foo.prototype.isPrototypeOf( a ); // true
```
使用`isPrototypeOf(...)`代表`在a的整個[[Prototype]]中，Foo.prototype是否出現過？`，

我們也可以透過使用`isPrototypeOf(...)`來調查兩個任意Object的互相關聯。
```javascript
// b是否是否出現在c的[[Prototype]]鏈中？
b.isPrototypeOf( c );
```

我們我們使用`Object.getPrototypeOf(...)`來取得單一Object的[[Prototype]]。
```javascript
Object.getPrototypeOf(a); 
// {constructor: ƒ, __defineGetter__: ƒ, __defineSetter__: ƒ, hasOwnProperty: ƒ, __lookupGetter__: ƒ, …}
```
而大多數瀏覽器中支援的一種非標準的方法也可以訪問到Object的[[Prototype]]
```javascript
a._proto_;
// {constructor: ƒ, __defineGetter__: ƒ, __defineSetter__: ƒ, hasOwnProperty: ƒ, __lookupGetter__: ƒ, …}
```
這個神奇的`_proto_`屬性跟constructor屬性一樣不存在於本身Object上，而是存在於內建的`Object.prototype`，雖然它看起來像一個屬性，但實際上將它看做是一個getter/setter會更合適。

# 結論
本篇章中介紹了Prototypal Inheritance的方法和JavaScript中的introspection方法，下面我們來做一些整理：
- new關鍵字建立出Object它會鏈接到創建他的函數原型上以獲得函數原型上的property/method，這便是原型繼承的概念。


# Reference
- [You Don't Know JavaScript](https://github.com/getify/You-Dont-Know-JS/blob/1st-ed/this%20%26%20object%20prototypes/ch5.md)

