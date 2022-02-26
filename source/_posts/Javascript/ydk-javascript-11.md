---
title: You Don't Know JavaScript [This & Object Prototypes] - this Or That?
date: 2020-10-22 14:43:09
tags:
- Javascript
- Front-end
- This & Object Prototypes

categories:
- You Don't Know JavaScript
---

`this`是JavaScript中最令人困惑的關鍵字之一，他會自動在每個function作用域中生成，但是`this`實際上是指向什麼對很多資深的JS開發人員來說也是困惑的，對於this這個機制而言並沒有這麼先進，但是開發者常常會將它複雜化或令人困惑的方式引用他，如果沒有對於this的充分了解那麼this這個關鍵字將會變得跟魔術一樣神奇。

<img src="https://pbs.twimg.com/profile_images/950748274914447361/fo36NbaV_400x400.jpg" alt="drawing" width="860"/>

<!-- more -->

# Why this?
如果`this`是個令人困惑的機制，那麼為什麼JavaScript會需要用到它？在了解他之前我們應該知道他的價值。
```javascript
function identify() {
	return this.name.toUpperCase();
}
function speak() {
	var greeting = "Hello, I'm " + identify.call( this );
	console.log( greeting );
}

var me = {
	name: "Kyle"
};
var you = {
	name: "Reader"
};

identify.call( me ); // KYLE
identify.call( you ); // READER
speak.call( me ); // Hello, I'm KYLE
speak.call( you ); // Hello, I'm READER
```
上面的程式中允許`identify(...)`與`speak(...)`重複使用`me`與`you`object，而不是每個object都需要自己的function。

如果不使用`this`，你也可以將object顯性的傳遞給function。
```javascript
function identify(context) {
	return context.name.toUpperCase();
}
function speak(context) {
	var greeting = "Hello, I'm " + identify( context );
	console.log( greeting );
}

var me = {
	name: "Kyle"
};
var you = {
	name: "Reader"
};

identify( you ); // READER
speak( me ); // Hello, I'm KYLE
```
以上面的例子來說，可以看到雖然`this`不是必要的，但是他可以更優雅的傳遞object，從而使API個簡潔並易於重新使用;而隨著模式更加複雜，將上下文使用顯性參數傳遞的方式會比隱式使用this的傳遞更加混亂。

# Confusions
開發者對於`this`如果是使用字面上的意思去解釋他的時候往往都會造成混亂，最常見的有兩種假設，但他們都是錯的。

## itself
第一種假設是`this`代表著函數本身，當你使用`遞歸（在函式內部呼叫function`）或一個在`首次調用可以解除綁定的事件處理`時會在自身函式中呼叫function。

對於剛接觸JS的開發者，他們認為由於function是object(所有function在JS中都是object)所以可以在函數調用之間儲存狀態（function屬性中的值），雖然這是可行的但是他的功能有限。
```javascript
function foo(num) {
	console.log( "foo: " + num );
	this.count++; // keep track of how many times `foo` is called
}

foo.count = 0;
for (var i=0; i<10; i++) {
	if (i > 5) {
		foo( i );
	}
}
// foo: 6
// foo: 7
// foo: 8
// foo: 9

// how many times was `foo` called?
console.log( foo.count ); // 0 -- WTF?
```
上面的程式中，即使我們對於foo(...)調用了四次但是`foo.count`依然是0，雖然我們確實有在foo的屬性中加入了count，但是因為`this`並不是只向該function的object，就算屬性名稱相同但是所指向的object不同所以依然是0。
```javascript
function foo(num) {
	console.log( "foo: " + num );

	// keep track of how many times `foo` is called
	// Note: `this` IS actually `foo` now, based on
	// how `foo` is called (see below)
	this.count++;
}

foo.count = 0;

var i;

for (i=0; i<10; i++) {
	if (i > 5) {
		// using `call(..)`, we ensure the `this`
		// points at the function object (`foo`) itself
		foo.call( foo, i );
	}
}
// foo: 6
// foo: 7
// foo: 8
// foo: 9

console.log( foo.count ); // 4
```
把原本的程式更改為上面就可以順利的訪問到foo中的count了，你可能會感到困惑，但是我們會在後面詳細地做解釋。

## Its Scope
還有一個主要的誤解，`this`是function的作用域，從某種意義上來說他是對的，但另一種意義上來說他是完全錯誤的。

對於scope來說他確實有點像object，在他的範圍中具有每個可用的標示符的屬性，但是JS引擎無法訪問到scope object。
```javascript
function foo() {
	var a = 2;
	this.bar();
}
function bar() {
	console.log( this.a );
}

foo(); //undefined
```
在上面的程式中使用`this.bar()`來調用bar(...)雖然在這裡行得通（之後的章節會解釋）但是其實是錯誤的，對於呼叫`bar(...)`最自然的使用方法就是直接引用他的標示符，但是寫上面這個程式的開發者希望將`bar(...)`與`foo(...)`的lexcial scope連結再一起以便bar(...)可以訪問到a，`這樣的連結是不可能的`，不能夠通過使用this來連結兩個lexcial scope。


# What's this?
對於`this`來說，他`不是取決於開發者時間所綁定而是運行時綁定，他是基於上下文取決於函數調用的條件`，這個綁定`與函式聲明的位置無關而是與函式調用的方式有關。`

當呼叫一個function時會創建一個`啟動紀錄(執行上下文)`，這個紀錄包含有關從何處調用function的信息、呼叫此function的方式、傳遞的參數等等;this會在function執行的期間使用紀錄的屬性之一。

參考文獻：
[You Don't Know JavaScript](https://github.com/getify/You-Dont-Know-JS/blob/1st-ed/this%20%26%20object%20prototypes/ch1.md)