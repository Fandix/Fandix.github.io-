---
title: You Don't Know JavaScript [This & Object Prototypes] - Prototypes [上]
date: 2020-12-17 10:47:54
tags:
- Javascript
- Front-end
- This & Object Prototypes

categories:
- You Don't Know JavaScript
---

我們在[Object [上]](https://ithelp.ithome.com.tw/articles/10254000)與[Object [下]](https://ithelp.ithome.com.tw/articles/10254011)中多次提到[[Prototype]]，但是都沒有提到他是什麼，現在我們將在這個章節中介紹什麼是prototype。

<img src="https://www.datocms-assets.com/16499/1595350861-prototyping.png?auto=format&dpr=0.42&fit=max&w=3840
" alt="drawing" width="860"/>

<!-- more -->

# [[Prototype]]
JavaScript中的物件都會擁有一個內部屬性，在語言規範中稱為`[[Prototype]]`，它用於被另一個物件引用，在這些大部分物件剛被建立的時候，Prototype會被設定為`非null`。
```javascript
const myObject = {
    a: 2
};
myObject.a; // 2
```
我們在[Object [下]](https://ithelp.ithome.com.tw/articles/10254011)中提到的
> 他會先在Object上執行一個[[Get]]操作，他會檢查物件並在物件中尋找有沒有這個被請求的屬性名稱，如果找到就返回相對應的值，如果沒能在物件中找到屬性則會做另一個重要的事情（遍歷[prototype]鏈）

以上面的程式來說，`myObject.a`他會調用myObject中的[[Get]]方法來檢查本身物件中是否有一個`a`屬性，如果有就會直接使用它，而`本身物件中沒有這個屬性的話才會將注意力轉向物件的[[Prototype]]`，會順著prototype鏈不斷的向上尋找，直到找到這個屬性或回傳`undefined(沒找到)`。

```javascript
const anotherObject = {
    a: 2
};

const myObject = Object.create(anotherObject);
myObject.a; // 2
```
> Object.create() 指定其原型物件與屬性，創建一個新物件。

上面的程式碼中，我們先建立一個anotherObject物件並新增了a屬性，並且myObject的prototype連接到anotherObject上，所以當我們在找myObject中的a屬性時，他會先檢查自身有沒有這個屬性，沒有的話則會向上一層prototype（anotherObject）尋找，所以就算自身物件中沒有a這個屬性但可以利用prototype這個機制找到更上層的物件。

## Object.prototype
每一個[[Prototype]]的終點是內建的`Object.prototype`，這個物件包含各種在Javascript中被使用的供通工具，因為`Javascript中所有普通的物件都是Object.prototype的衍伸`。

## Setting & Shadowing Properties
物件的prototype尋找屬性的概念其實跟scope尋找變數的概念相似，他們都會因為已經找到需要找的屬性（變數）而停止繼續向上尋找，這就是`Shadowing`的概念。
```javascript
myObject.foo = 'bar';
```
如果foo同時出現在`myObject本身`與`myObject的更高層[[Prototype]]`，那麼Javascript就只會訪問自身的foo屬性，因為自身的foo屬性shadowing了上層所有的同名屬性。

但如果foo不存在於自身物件而是存在於myObject[[Prototype]]的更高層時，可能會有三種情況：
1. 如果foo在[[Prototype]]的高層某處被找到，而這個foo屬性`沒有被設定為唯讀(writable: false)`，則這個foo會被添加到myObject上並將這個層級以上的所有同名屬性shadowing。
2. 如果foo在[[Prototype]]的高層某處被找到，但這個foo`被設定為唯讀(writable: true)`，那麼這個屬性`不能添加到myObject上和不能改變這個屬性的狀況`，如果在嚴格模式下運行則會直出錯誤否則會這個屬性的添加會被忽視，繼續往上層[[Prototype]]尋找。
3. 如果foo在[[Prototype]]的高層某處被找到而且他是一個`setter`，那麼這個被找到的屬性便會一直被調用，沒有foo會被添加到myObject（不會shadowing）。


# Class
由於Javascript沒有class的概念，所以繼承需要透過將[[Prototype]]鏈到另一個Object上，由於Javascript中沒有Class的概念，所以會有一些`模仿`class的奇異行為，讓我們來介紹一下這個奇怪的方式。

在默認情況下，所有函數都會擁有一個名為`prototype`的`公開且不可枚舉`的屬性
```javascript
function Foo() {
    // ...
}
Foo.prototype; //{constructor: f}
```
這個Object通常稱為`Foo的原型`，通過調用`new Foo()`創建的每個Object都將把他們（創建出的Object）的[[Prototype]]鏈接到Foo的原型。
```javascript
function Foo(){
    // ...
}

const a = new Foo();
Object.getPrototypeOf(a) === Foo.prototype; // true
```
當a透過new關鍵字創建時，就會將a這個Object的[[Prototype]]鏈接到Foo的原形上，在Javascript中沒有像OOP語言中可以instances出多個物件，而是創建了一個新的Object並將他們內部的[[Prototype]]鏈接在一起，`沒有初始化一個Object也沒有對他進行拷貝而是將他們鏈接在一起`。

## Constructors
```javascript
function Foo(){
    // ...
}
const a = new Foo();
```
在上面的例子中，因為我們使用了`new`關鍵字使我們的Foo看起來很像一個class，除了點之外因為對於Foo()這個操作方法來說，看起來很像在調用Contructors函數，綜合這兩點讓我們的這個行為看起來很像在使用一個class。

```javascript
function Foo(){
    // ...
}
Foo.prototype.constructor === Foo; // true

const a = new Foo();
a.constructor === Foo; // true
```
Foo.prototype Object在默認情況下獲得了一個`公開且不可枚舉的屬性constructor`，這個屬性是對於這個函數的引用，此外我們也可以看到透過new關鍵字創建出的a Object也擁有一個constructor屬性，因為a是透過new創建的所以他的屬性會指向`創建他的函數(Foo)`。

### Constructor Or Call?
在上面的程式碼中，因為我們透過new關鍵字`構建`一個Object（a）所以很容易將Foo歸類為是個`constructor`，但實際上Foo`並不是一個constructor`他只是一個普通的函數，但是這邊會有一個很奇怪的事情發生，當在一個常規函數前加上new關鍵字時，這個函數的調用就會變為`constructor call (not constructor)`。
```javascript
function NothingSpecial() {
	console.log( "Don't mind me!" );
}

var a = new NothingSpecial(); // "Don't mind me!"

a; // NothingSpecial {}
```
上面的例子中NothingSpecial只是一個普通的函數，但是當他前面加上new關鍵字時他就會constructor一個Object，這個調用的行為是`constructor call（構造函數調用）`但函數本身並不是constructor。

不過也可以說，`在JavaScript中，所有在函數前加上new的函數都是constructor`。
> Functions aren't constructors, but function calls are "constructor calls" if and only if new is used.

## Mechanics
```javascript
function Foo(name) {
	this.name = name;
}

Foo.prototype.myName = function() {
	return this.name;
};

var a = new Foo( "a" );
var b = new Foo( "b" );

a.myName(); // "a"
b.myName(); // "b"
```
在上面的範例中展現了兩個class-orientation的技巧:
1. 我們在[this All Makes Sense Now! [上]](https://ithelp.ithome.com.tw/articles/10253946)中的`New Binding`中提到，透過new關鍵字會先建立一個新的object再將這個新object的[[Prototype]]鏈接到創建他的函數上，最後這個`創建他的函數的this會指向這個新建立出的物件`，所以Foo中的this會指向新創建出的a、b。
2. 在Foo.prototype的鏈上新增一個method，由於new關鍵字創建出的object都會將[[Prototype]]鏈接上創建他的函數，所以a、b的都會擁有Foo.prototype上的myName這個method。

第一次看到第二點的時候可能會覺得在創建a,b的時候將Foo.prototype中的method`複製`給了a與b，但想一想我們一開始介紹的prototype的找尋方法，當我們呼叫了a物件中的myName method時，Javascript引擎會先在a Object中找是否有這個method，若沒有的話則會`往prototype上層尋找`，由於a與Foo的prototype鏈接再一起，所以便會向上找Foo的prototype，在這裡找到這個method便拿來使用。

## Constructor Redux
我們在一開始提到constructor屬性的話題，當時我們提到`a.constructor === Foo; //ture`這件事，讓我們以為a擁有一個constructor屬性並且這個它指向Foo，但這是不正確的。

在默認情況下使用new Foo()創建出的物件上才會存在Foo.prototype的.constructor屬性，但是如果我們將Foo的prototype更改為其他Object時，這樣這個被更改的prototype便不會出現在被創建出的Object上。
```javascript
function Foo(){
    // ...
}
// change prptotype object
Foo.prototype = {
    a: 1 
}

const a = new Foo();
a.constructor === Foo; // false
a.constructor === Object; // true
```
由於a這個Object本身沒有.constructor這個屬性，所以沿著[[Prototype]]向上找到了Foo.prototype，但這個物件中也`沒有.constructor(被我們手動更改)`，所以他會繼續向上找，找到`最上層的Object.prototype`，在這裡找到了.constructor，於是將a中的屬性指向Object。

所以可以發現其實.constructor並不是一個不可變得屬性，`他只是不可枚舉但依然是可更改的(writable: true)`，可以在[[Prototype]]上用任何值添加或覆蓋掉原本產生的constructor屬性。

根據[[Get]]的算法只要在任何一個地方找到.constructor就會把拿來使用，這可能會跟你的預期大不相同，所以對於.constructor來說它是極度不安全的引用，`應該盡量避免使用`。


# 結論
本篇章中介紹了JavaScript中的[[Prototype]]以及Class的概念，讓我們總結一下：
- JavaScript中的物件都會擁有一個內部屬性，在語言規範中稱為`[[Prototype]]`。
- 每一個[[Prototype]]的終點是內建的Object.prototype，`Javascript中所有普通的物件都是Object.prototype的衍伸`。
- 物件的prototype尋找屬性的概念其實跟scope尋找變數的概念相似，他們都會因為已經找到需要找的屬性（變數）而停止繼續向上尋找。
- Javascript沒有class的概念，所以繼承需要透過將[[Prototype]]鏈到另一個Object上。
- 使用new關鍵字建立新的Object時，這個新的Object的Prototype會`鏈接到建立他的函數原型上`。
- 使用new關鍵字建立新的Object，`沒有初始化一個Object也沒有對他進行拷貝而是將他們鏈接在一起`。
- 在JavaScript中，在普通函數前面加上new關鍵字，此函數的調用行為會變成`constructor call（構造函數調用）`。
- .constructor並不是一個不可變得屬性，`他只是不可枚舉但依然可更改的(writable: true)`。

# Reference
- [You Don't Know JavaScript](https://github.com/getify/You-Dont-Know-JS/blob/1st-ed/this%20&%20object%20prototypes/ch5.md)