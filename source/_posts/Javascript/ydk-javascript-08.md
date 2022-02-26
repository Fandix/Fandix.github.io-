---
title: You Don't Know JavaScript [Scope & Closures] - Limiting Scope Exposure?
date: 2020-10-14 16:50:18
tags:
- Javascript
- Front-end
- Scope & Closures

categories:
- You Don't Know JavaScript
---

目前為止我們都著重在解釋作用域與變量的工作機制，有了這些基礎後將進到下一步，首先我們要先探討不同級別的作用域來組織宣告的變量，特別是`減少作用域的過度暴露`。

<img src="https://media.istockphoto.com/photos/speed-of-light-in-london-city-picture-id961050380?k=20&m=961050380&s=612x612&w=0&h=BDqxdZNgPbce2BCFVWYuCeI6UX66Yh0strtbXK6DXX8=" alt="drawing" width="860"/>

<!-- more -->

# Least Exposure
在軟體工程中定義了`The Princioke of Least Privilege(POLP)`原則，他認為系統的component設計應該以`最小特權`,`最少訪問`,`最小暴露`來運行，若是每個部分都以最小且必要的方式連接則整個系統的安全性會更強大，因為若是一部份發生故障那麼他對其他部分的影響會是最小的。

`PLOE`是針對較低級別的設計，可以將其應用到作用域中，如何做到最大程度地減少作用域的曝光？答案是`在每個作用域中宣告自己的變量`。

在我們設計程式的時後常會避免將變量都在全域中宣告，雖然都這麼避免但是卻不知道這樣會造成什麼困難，當你程式中的變量超過自己的作用域暴露給另一個作用域的時候，通常會出現三個危險：
1.**命名衝突**：當你在一個共同的作用域中使用常用的宣告命名，可能會在不經意間產生兩個一樣名稱的變量或function，這會導致命名的衝突，而這個衝突很可能會造成你程式的bug，因為他呼叫的不一定會是你要的那個變量或function。
2.**意外的行為**：若是你將你的變量或function公開能夠讓其他地方使用到，有可能會被其他開發者`以你不希望的方式使用`你宣告的變量或function。
3.**意外的依賴**：如果公開你的變量或function他可能在不經意間造成其他開發者的依賴或使用，儘管短時間內不會有問題但是未來如果面臨重構則會有大麻煩，因為你的這個變量會影響到除了你自己之外的更多地方。

經過上面這些危險可以知道為什要避免將變量或function宣告在全域或是其他作用域可以訪問到的地方，盡可能地將變數宣告在自身範圍內以減少錯誤的發生。

# Hiding in Plain (Function) Scope
在數學中有個操作叫做`階層`，他是將給定整數與所有連續的較低整數的乘積（6! = 6*5*4*3*2*1），你可以寫一個function幾算每次的值，也可以將之前幾算的值儲存起來這樣就可以不用每次都從頭計算
```javascript
var cache = {};

function factorial(x) {
    if (x < 2) return 1;
    if (!(x in cache)) {
        cache[x] = x * factorial(x - 1);
    }
    return cache[x];
}

factorial(6); // 720

cache;
// {
//     "2": 2,
//     "3": 6,
//     "4": 24,
//     "5": 120,
//     "6": 720
// }

factorial(7); // 5040
```
我們使用`cache`來暫存factorial(...)的結果，但是因為他是在外部作用域的變數，所以有可能會發生上面提到的那三種危險，所以我們必須將`cache`隱藏在作用域中而不是讓他暴露在其他地方可以訪問的位置。
```javascript
// outer/global scope

function hideTheCache() {
    // "middle scope", where we hide `cache`
    var cache = {};

    return factorial;

    // **********************

    function factorial(x) {
        // inner scope
        if (x < 2) return 1;
        if (!(x in cache)) {
            cache[x] = x * factorial(x - 1);
        }
        return cache[x];
    }
}

var factorial = hideTheCache();

factorial(6); // 720
factorial(7); // 5040
```
我們建立一個`hideTheCache(...)`來產生一個作用域，讓我們吧`cache`與`facto[](http://)rial(...)`隱藏在這個function作用域中，因為他們存在於同一個作用域，所以也可以使factorial(...)訪問到cache，最後我們在將factorial的reference return出去賦予給`var factorial`，這樣既可以訪問到cache也可以將它避免暴露在外面。

雖然這樣的確能避免變量洩露到外層，但是每次只要有這個需求的時候就會需要在定義一個函式作用域，這是一件很煩瑣的事情，我們可以使用`函數表達式`來取代每次都需要定義一個新的函示。
```javascript
var factorial = (function hideTheCache() {
    var cache = {};

    function factorial(x) {
        if (x < 2) return 1;
        if (!(x in cache)) {
            cache[x] = x * factorial(x - 1);
        }
        return cache[x];
    }

    return factorial;
})();

factorial(6); // 720
factorial(7); // 5040
```
雖然可能會覺得他依然是建立了一個function來隱藏cache，但是回想一下[The Scope Chain](https://ithelp.ithome.com.tw/articles/10250830)中的"Function Name Scope"，由於factorial(...)是函數表達式所以對於function hideTheCache(...)這個function來說，他在做完後賦予結果給factorail之後他便會消失，舉個簡單的例子
```javascript
let a = (function b(){return false};); //賦予a結果後便會消失

console.log(a()); //false
console.log(b()); // b is undefined
```
這代表著我們可以將函數表達式(function Exoression)的名字取的完全相同而不會發生衝突，意味著我們可以根據我們的想法任意取名而不會影響到其他部分造成衝突，當然你也可以使用匿名函式來達到一樣的作用。

## Invoking Function Expressions Immediately
在上面的程式碼中在function expressions的最後添加了第二個`()`，那實際上是在要用剛剛定義的function expression，這種function expression的情況下第一個(...)不是嚴格需要的，但是為了可讀性還是建議加上它們。

因此當我們定義了一個function expression後就立即調用他，我們稱這個為`Immediately Invoked Function Expression(IIFE)`，IIFE在JS中非常常用到，他可以使有名稱的以可以是匿名的，而且他也可以是獨立的將其返回的值`=`給指定的變量。
```javascript
// outer scope

(function(){
    // inner hidden scope
})();

// more outer scope
```

### Function Bonudaries
不同的定義範圍會影響到IIFE的結果，因為他是一個完整的函數所以他會更改函數邊界的某些語句或構造，舉例來說，對於return而言如果將IIFE包在起中，某些程式的`return`會改變原本的意義，因為他會涉及到IIFE的功能;而非箭頭函數的IIFE還會涉及到this的綁定，`break`和`continue`都不會跨越IIFE的函數邊界以控制外部的循環或block，所以若你有需要使用`return`,`this`,`break`,`continue`的情況，使用IIFE可能不是個好辦法。


# Scoping with Blocks
到目前為止我們介紹了如何使用IIFE來完成作用域的隱藏，現在我們要介紹如何使用塊狀作用域(let)來達到一樣的效果，通常`{...}`都會當作一個block，`但是不一定會是作用域`，而block只有在必要的時候才會成為作用域（其中含有塊狀宣告）。
```javascript
{
    // not necessarily a scope (yet)

    // ..

    // now we know the block needs to be a scope
    let thisIsNowAScope = true;

    for (let i = 0; i < 5; i++) {
        // this is also a scope, activated each
        // iteration
        if (i % 2 == 0) {
            // this is just a block, not a scope
            console.log(i);
        }
    }
}
```
上面的程式碼中可以看到，並不是所有的{...}都可以是作用域
- object使用的{...}用來訂定鍵值列表，這個{...}就不是作用域。
- class所使用的{...}這也不是一個block或是作用域。
- function所使用的{...}他不是一個block但是他是一個作用域。
- switch...case上的{...}也不是block與作用域。

獨立的{...}在過去不能成為作用域的情況下不常見，但是ES6中提供了塊狀作用域宣告let/const於是他們開始流行，獨立的{...}可以在另一個作用域的內部執行。
```javascript
if (somethingHappened) {
    // this is a block, but not a scope
    {
        // this is both a block and an explicit scope
        let msg = somethingHappened.message();
        notifyOthers(msg);
    }
    // ..
    recoverFromSomething();
}
```
在上面的程式中，我們定義了一個內部的塊狀作用域，因為不是整個if中都需要這些變量，大多數的開發者都會選擇讓這些宣告存在於if中，但是若整個程式的開發越來越大，洩露變量的風險也會提告，所以以POLE的規則來說，使用上面的方法比較安全的，回顧[The (Not So) Secret Lifecycle of Variables](https://ithelp.ithome.com.tw/articles/10250873)提到的TDZ，所以建議在這些塊狀作用域的頂部就使用let/const宣告變數以降低TDZ Error的風險。

## var and let
在JS的一開始`var`代表著`屬於這整個函數的變量`，雖然也可以將var定義在block中，但是作者並不建議，他認為var應該宣告在`函數作用域的頂部`會最好。

那為什麼都不使用let就好呢？因為作者覺得var有明確的表達`此變量是屬於函數作用域的`，若都在整個函式中使用let，那麼就不能從視覺上引起注意讓人區分函數作用域中所有宣告的區別，換句話說var比let能更好的在函數作用域中進行溝通，而let則是能夠讓函式作用域與塊狀作用域中的通信，所以若是你的程式中同時需要使用函式作用域和塊狀作用域，那們推薦你同時使用var與let。

## Where To let?
對於POLE的觀點來說，他不干涉你使用哪一種宣告的語法，但是在做這個決定之前要先思考一個問題:`對於我要宣告的這個變量而言，如何讓他指曝光在最小的範圍`，作者推薦`如果個宣告屬於塊狀作用域則使用let，若屬於函式作用域則使用var`。

對於for loop來說，建議一率都使用`let`，因為在loop的過程中i始終都只有在循環的內部做使用，在這種情況下需要使用let將i綁定在loop中減少他的洩露
```javascript
for (let i = 0; i < 5; i++) {
    // do something
}
```

## What's the Catch?
在ES3中提供了`try...catch`功能，而其中的catch提供了一個顯微人知的塊狀作用域。
```javascript
try {
    doesntExist();
}
catch (err) {
    console.log(err);
    // ReferenceError: 'doesntExist' is not defined
    // ^^^^ message printed from the caught exception

    let onlyHere = true;
    var outerVariable = true;
}

console.log(outerVariable);     // true
console.log(err);
// ReferenceError: 'err' is not defined
// ^^^^ this is another thrown (uncaught) exception
```
catch宣告了`err`是屬於他自身的塊狀作用域，而catch的{...}若是使用`let`宣告變數，則他對被定義為塊狀作用域宣告（離開catch{...}）之後就不存在，但是用`var`宣告變數，他會被附加到外部/全域作用域中。

在ES2019更改了catch，他變成可以更改他的聲明（不一定要使用err），若不選擇（不使用err）則catch不再是作用域，但他依然是一個block，因此若你需要對異常做出反應，但卻不不關心錯誤的值，則可以直些忽略掉catch的聲明。
```javascript
try {
    doOptionOne();
}
catch {   // catch-declaration omitted
    doOptionTwoInstead();
}
```

# Function Declarations in Blocks(FiB)
在JS中有個東西叫做`FiB`，他是說如果我們在block內部定義一個function會發生什麼事？
```javascript
if (false) {
    function ask() {
        console.log("Does this run?");
    }
}
ask();
```
對於上面這個程式碼中會發生什麼事？
1.因為ask是定義在if的塊狀作用域中，所以在外部/全域作用域中不可訪問，所以產生ReferenceError。
2.因為if並未被運行，所以ask標示符尚未被定義為function，所以產生TypeError。
3.ask()正常運行。
其實若你在不同環境下運行這個程式會有不同的結果，若是在JS的環境下運行，因為對於JS來說ask是定義在內部作用域的所以不可訪問（錯誤1）;然而若你是在瀏覽器的環境下執行將表現為錯誤2，它意味著ask並不存在於內部作用域，他是在外面只是尚未被定義（undefined）。

在ES6引入塊狀作用域之前瀏覽器就已經接觸到FiB行為了，為了不修改到一些老舊的網站，所以才所以才會造成瀏覽器與JS的結果不一樣。

如果真的需要定義函式在block中，可以嘗試使用`function expression`
```javascript
var isArray = function isArray(a) {
    return Array.isArray(a);
};

// override the definition, if you must
if (typeof Array.isArray == "undefined") {
    isArray = function isArray(a) {
        return Object.prototype.toString.call(a)
            == "[object Array]";
    };
}
```
對於FiB來說是避免在block中`declarations（宣告）`一個函式，上面的程式中是使用`function expression`由於他不是宣告，所以是可以正常動作的。

參考文獻：
[You Don't Know JavaScript -2nd](https://github.com/getify/You-Dont-Know-JS/blob/2nd-ed/scope-closures/ch6.md)