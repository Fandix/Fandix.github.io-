---
title: You Don't Know JavaScript [This & Object Prototypes] - This All Makes Sense Now! [上]
date: 2020-10-28 14:13:03
tags:
- Javascript
- Front-end
- This & Object Prototypes

categories:
- You Don't Know JavaScript
---

在[this Or That?](https://ithelp.ithome.com.tw/articles/10253880)中提到了許多對於`this`的誤解，並且也對於這些誤解做了一些解釋，我們了解到`this`是對每個函式調用的綁定，是基於被調用的位置而不是宣告的位置。

<img src="https://www.apkmirror.com/wp-content/uploads/2022/01/99/61ee5a95c43c4.png" alt="drawing" width="860"/>

<!-- more -->

# Call-site
為了瞭解`this`，我們必須要先了解一個重要的觀念`call-site`，它代表著`函式在程式中被調用的位置`，通常要找到call-site是要`定位從何處調用此函式`，但通常這個行為並不是這麼容易，因為某些模式下會掩蓋掉真正的call-site，這個狀態下我們需要考慮的是`call-stack`，call-stack代表著呼叫function的堆疊，比如說
```javascript
function c(){
    console.log('c');
}

function b(){
    c();
    console.log('b');
}

function a(){
    b();
    console.log('a');
}

a();
```
上面的程式中呼叫a(...)，而a(...)中又呼叫b(...)，最後b(...)中又呼叫c(...)，而call-stack -> a() -> b() -> c()。
![https://ithelp.ithome.com.tw/upload/images/20201027/20124767MJrJJDrECS.png](https://ithelp.ithome.com.tw/upload/images/20201027/20124767MJrJJDrECS.png)
而call-site來說，他是`在他call-stack父層中呼叫自己的位置`，看其來很繞舌不過我們一樣拿上面的程式碼來做舉例，對於`c(...)`而言，他的`call-site`就是在`call-stack父層（b(...)）`所呼叫的位置，以此類推。
```javascript
function c(){
    console.log('c');
}

function b(){
    c(); // function c(...) => call-site
    console.log('b');
}

function a(){
    b(); // function b(...) => call-site
    console.log('a');
}

a();
```

# Nothing But Rules
介紹完call-site與call-stack，接下來我們將重點移到call-site是如何確定函數執行期間`this`的指向，對於這個指向我們有`4條規則`，我們先一一介紹是哪些規則。

## Default Binding
第一種規則是來自函數最常見的情況`函數獨立調用（換句話說就是只呼叫自己而沒有在內部嵌入其他函數）`，若沒有其他規則適用，可以將這個規則是為萬用規則。
```javascript
function foo() {
	console.log( this.a );
}

var a = 2;
foo(); // 2
```
以上面的程式中，`default binding`代表著`this`是綁定`全域物件`，由於我們的`var a = 2`是宣告在全域，所以這裡的this才會指向到全域的a，我們要如何知道default binding規則適用於這個例子？

我們通過call-site來觀察foo(...)是在哪裡被調用的，在我們的程式中foo(...)是一個`直白且無修飾的`函數呼叫，意味著他沒有在內部嵌入其他函數，所以default binding規則在這裡適用。

但是default binding對於使用嚴格模式來說就不適用
```javascript
function  foo ( )  { 
	"use strict" ;
	console . log (  this . a  ) ; 
}

var  a  =  2 ;
foo ( ) ;  // TypeError: `this` is `undefined`
```
但是有一個特別的點，對於嚴格模式來說只要foo(...)的`作用域內`不是嚴格模式，那麼`this`一樣也可以綁定到全域物件。
```javascript
function  foo ( )  { 
	console . log (  this . a  ) ; 
}

var  a  =  2 ;

( function ( ) { 
	"use strict" ;

	foo ( ) ;  // 2 
} ) ( ) ;
```

## Implicit Binding
第二個規則是需要考慮call-site是否有一個環境物件（context object）。
```javascript
function foo() {
	console.log( this.a );
}

var obj = {
	a: 2,
	foo: foo
};
obj.foo(); // 2
```
我們宣告了一個function foo(...)之後將他加入到obj物件中成為他的property，無論foo()是否一開始就在`obj`上被宣告或是後來才加入到`obj`中（上面的例子），這個`函數`都不被obj所真正的`擁有或包含`，但是由於對於call-site來說obj環境來`Reference`foo(...)，所以可以說obj在函數被`調用的時間點`擁有或包含這個`funciton reference`。

當一個context object中有一個function reference則implicit binding規則會將這個fucntion中的this綁定這個object，所以以上面的例子來說foo(...)中的this所指向的就是`obj`。

對於嵌套的物件來說，`只有最後一層/最上層`物件才會對call-site起作用
```javascript
function foo() {
	console.log( this.a );
}

var obj2 = {
	a: 42,
	foo: foo // call-site
};

var obj1 = {
	a: 2,
	obj2: obj2
};

obj1.obj2.foo(); // 42
```

### Implicitly Lost
當一個implicitly bound的函數丟失了綁定，則會`退回default binding`，至於指向的是全域物件還是undefined則取決於是否使用嚴格模式。
```javascript
function foo() {
	console.log( this.a );
}

var obj = {
	a: 2,
	foo: foo
};

var bar = obj.foo; // loses that binding!

var a = "oops, global"; // `a` is property on global object

bar(); // "oops, global"
```
雖然`bar`似乎是obj.foo的reference，但是實際上他只是對foo(...)本體的另一個reference，換句話說雖然bar與obj.foo都是對foo(...)本體的reference，但是實際上是`兩個不一樣的地方`，而且對於call-site而言呼叫bar(...)是一個`直白且無修飾的`函數呼叫，所以他適用於`default binding`。

還有一個更加微妙更常見更出乎意料的方式，當我們傳遞一個callback function時
```javascript
function foo() {
	console.log( this.a );
}

function doFoo(fn) {
	// `fn` is just another reference to `foo`
	fn(); // <-- call-site!
}

var obj = {
	a: 2,
	foo: foo
};

var a = "oops, global"; // `a` also property on global object
doFoo( obj.foo ); // "oops, global"
```
對於參數的傳遞來說他是一個`隱性賦值`，而且如果要傳遞的參數是函數的話則是一個`隱性的reference賦值`，所以結果會與上一個程式碼相同。
```javascript
function doFoo(var fn = obj.foo){
  // 隱性function reference 賦值代表fn與obj.foo的reference是不同的。
}
```
對於傳遞callback function作為參數會丟失binding這件事，除了自己定義的function之外對於原生的funciton也是一樣的情況。
```javascript
function foo() {
	console.log( this.a );
}

var obj = {
	a: 2,
	foo: foo
};

var a = "oops, global"; // `a` also property on global object
setTimeout( obj.foo, 100 ); // "oops, global"
```
可以將他看為
```javascript
function setTimeout(var fn = obj.foo, delay){
    //隱性function reference 賦值
}
```

## Explicit Binding
我們介紹了Implicit Binding如果需要間接地將函數中的this綁定到這個物件上，會需要對這個物件做一些改變（將function reference引入到物件屬性中），但是有沒有方法是可以不更改物件的型態卻又可以使function的this綁定著這個物件的呢？

我們可以使用JS所提供function的prototype（後面會介紹）`call(...)`或`apply(...)`method，他們的第一個參數都是一個物件，他代表著我這個fucntion的this所指向的目標，因為明確的指出this要指向什麼所以我們稱這種方式為Explicit Binding。
```javascript
function foo() {
	console.log( this.a );
}
var obj = {
	a: 2
};

var a = 5; // declaration in global 
foo.call( obj ); // 2
```
通過foo.call(...)的方式將this明確的指向obj，注意的是如果對於call(...)或apply(...)的第一個參數傳遞的不是一個物件（string,boolean,number...）那麼傳遞的這個參數的類性會被包裝在物件（new String(...), new Boolean(...), new Number(...)）這種行為稱為`boxing`。

### Hard Binding
雖然可以對單獨的function進行顯性綁定，但是依然無法解決上面提到的賦值導致綁定丟失的問題，但是可以有一種明確綁定的變種可以解決這個問題。
```javascript
function foo() {
	console.log( this.a );
}

var obj = {
	a: 2
};
var a = 4; // declaration in global

var bar = function() {
	foo.call( obj );
};
bar(); // 2
setTimeout( bar, 100 ); // 2

// `bar` hard binds `foo`'s `this` to `obj`
// so that it cannot be overriden
bar.call( window ); // 2
```
我們在bar內部強制綁定了foo(...)的this指向obj，所以無論之後怎麼調用bar(...)在他的內部都會自動的強制綁定obj，這種行為我們稱為`hard binding`。

對於hard binding來說，ES5中提供了`funciton.prototype.bind`可以將物件強制綁定給函數。
```javascript
function foo(something) {
	console.log( this.a, something );
	return this.a + something;
}

var obj = {
	a: 2
};

var bar = foo.bind( obj );

var b = bar( 3 ); // 2 3
console.log( b ); // 5
```

### API Call "Contexts
在許多現在JS的內建函數中都有提供一個可選的參數通常稱為context，這種設計可以讓你直接填入你需要綁定的object而不必一定要使用`bind(...)`。
```javascript
function foo(el) {
	console.log( el, this.id );
}

var obj = {
	id: "awesome"
};

// use `obj` as `this` for `foo(..)` calls
[1, 2, 3].forEach( foo, obj ); // 1 awesome  2 awesome  3 awesome
```
```javascript
arr.forEach(function callback(currentValue[, index[, array]]) {
    //your iterator
}[, thisArg]);
/* 
    callback : 把 Array 中的每一個元素作為參數，帶進本 callback function中
         currentValue : 當前被處理的Array元素
         index(可選)：當前被處理的Array元素的index
         array(可選)：forEach()本身的Array -> arr
    thisArg(context)(可選)：callback function的this (需要綁定的物件)
*/
```

## New Binding
在傳統擁有class的語言中，constructor是一個特殊的method，當一個class被`new`實體化後這個constructor就會被調用以用來初始化這個class。
```javascript
something = new MyClass(...);
```
雖然JavaScript中也有`new`但是他與其他語言的new是沒有關係的，對於JavaScript來說`constructor就只是個函數`他們偶然的與new一起被調用，但他卻不依附於也不會初始化一個class。

當一個函數前面加上new調用，也就是constructor調用時，會自動完成以下的事情：
1. 憑空創造一個全新的物件。
2. 被創建的物件會接入原形鍊（[[prototype]]-Link）。
3. 被調用funciton中的this被設定為指向新的物件。
4. 除非function return屬於自身的物件，否則這個new調用的function會`自動return`這個新創建的物件。

```javascript
function foo(a) {
	this.a = a;
}

var bar = new foo( 2 );
console.log( bar.a ); // 2
```
使用new來調用foo(...)等於我們建立了一個新的物件並將function中的this指向這個新創出來的物件，這種綁定新建出物件的方法稱為new binding。

參考文獻：[You Don't Know JavaScript](https://github.com/getify/You-Dont-Know-JS/blob/1st-ed/this%20%26%20object%20prototypes/ch2.md)