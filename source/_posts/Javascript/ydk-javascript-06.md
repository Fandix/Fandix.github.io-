---
title: You Don't Know JavaScript [Scope & Closures] - Around the Global Scope?
date: 2020-10-11 00:25:11 
tags:
- Javascript
- Front-end

categories:
- You Don't Know JavaScript
---


我們在前面的章節中不斷地提到`全域作用域`，可能會疑問為什麼最外層的作用域是全域作用域？而為什麼對於JS來說會是最重要的？僅`避免使用全域範圍`就足夠了嗎？

對於JS來說，全域是一個很複雜的環節，他非常的實用但細節也非常多，本章節將會對這些問題做一些解釋。

<img src="https://hbr.org/resources/images/article_assets/2019/06/Jul19_05-931636724.jpg" alt="drawing" width="860"/>

<!-- more -->

# Why Global Scope?
對於我們在開發一個專案的時候，我們並不會把所有的程式都寫在同一個文件中，那麼 JS 是通過什麼方式讓不同文件中的程式在運行的時候上下組合再一起的？？

對於瀏覽器來說主要有三種方式：
1. 如果使用的是 `ES Modules`，那麼這些文件會被JS單獨加載，然後使用 `import` 將需要的 module 引入到需要的環境中，單獨的 module 通過這個方法互相引入與合作且不需要共享作用域。
2. 如果在建構的時期將程式捆綁在一起，則通常會將所有文件連接在一起然後再交給 JS 引擎和瀏覽器，然後他們都只處理連接在一起的這個大文件，雖然所有的程式都在一個文件中，但每個部分都需要有能夠讓其他部分呼叫的名字與功能;在某些設置中，文件中內容都包裝在一個封閉的作用域中（wrapper function..），每個功能都可以透過共享的作用域來讓其他功能呼叫。
```javascript
(function wrappingOuterScope(){
    var moduleOne = (function one(){
        // ..
    })();

    var moduleTwo = (function two(){
        // ..

        function callModuleOne() {
            moduleOne.someMethod();
        }

        // ..
    })();
})();
```
以上面的例子來說，由於 `moduleOne` 與 `moduleTwo` 都存在於 `wrappingOuterScope` 作用域中，所以可以所以可以互相訪問，而 wrappingOuterScope 只是一個 function 而不是一個真正的全域作用域，但是他儲存了所有的功能，所以他可以稱為是全域作用域的替身。
3. 如果都不是上面的兩種方式，那麼全域作用域就是聯繫不同功能的唯一方法。
```javascript
var moduleOne = (function one(){
    // ..
})();
var moduleTwo = (function two(){
    // ..

    function callModuleOne() {
        moduleOne.someMethod();
    }

    // ..
})();
```
由於沒有共同的 function scope，所以將 `moduleOne` 與 `moduleTwo` 宣告在全域中，而事實上JS是分別對兩個文件進行加載
```javascript
// module1.js
var moduleOne = (function one(){
    // ..
})();
```

---
```javascript
// module2.js
var moduleTwo = (function two(){
    // ..

    function callModuleOne() {
        moduleOne.someMethod();
    }

    // ..
})();
```
這兩個檔案在瀏覽器中分別被加載，每個文件中的頂部變量宣告都會成為全域變量，而全域作用域是這兩個文件溝通的唯一橋樑，所以對JS引擎來說這他們都是獨立的程式。

全域作用域還包括：
- JS exposes its built-ins：
    - **primitives**: `undefined`,`null`,`Infinity`,`NaN`...
    - **natives**: `Date()`,`Object()`,`String()`...
    - **global functions**: `eval()`,`parseInt()`...
    - **namesoaces**: `Math`.`Atinucs`.`JSON`
    - **friends of JS**: Intl, WebAssembly
- The environment hosting the JS engine exposes its own built-ins:
    - `console`
    - `DOM`(window,document...)
    - `timer`(setTimeout(...)...)
    - `web platform APIs`(history,navigatot...)


# Where Exactly in this Gloval Scope
雖然說全域作用域就是文件的最外層，也就是說他不存在於任 function 或 block 中，但他的定義也不是這麼間單的，不同的JS環境對全域作用域的定義與處理方式都不一樣，如果沒有分辨的能力可能會有不能預期的錯誤出現。

## Browser "Window"
```javascript
var studentName = "Kyle";

function hello() {
    console.log(`Hello, ${ studentName }!`);
}
hello(); //Hello, Kyle!
```
我們可以使用 `<script>` 標籤加入到瀏覽器的環境中，他會動態的建立一個 DOM 元素，上面的例子中我們將 `stundntName` 和 `hello(...)` 都宣告在全域作用中，這意味著我們可以在全域的物件（在瀏覽器中全域的物件是 `window` ）中找到與他們同名的屬性。
```javascript
var studentName = "Kyle";

function hello() {
    console.log(`Hello, ${ window.studentName }!`);
}
window.hello(); // Hello, kyle!
```

### Globals Shadowing Globals
我們有在[The Scope Chain](https://ithelp.ithome.com.tw/articles/10250830)介紹到什麼是 shadowing，內部作用域的宣告可以覆蓋外部作用域的宣告，讓其無法訪問到外部作用域的同名變量。

而對於全域作用域來說，全域物件的屬性會被在全域宣告的變量shadowing。
```javascript
window.something = 42; // property of object in global

let something = "Kyle"; // declarate blobal variable

console.log(something); // Kyle -> shadowing
console.log(window.something); //42
```
上面的程式碼中，對於全域的物件宣告一個someting的屬性，但是也使用 `let` 宣告一個同名的變量，這樣的結果是 something 的詞法宣告會 shadowing 全域物件的屬性。

### DOM Globals
對於DOM來說有一個特別的現象，當你處於瀏覽器的環境下，若你有一個 DOM 元素它具有 `id Attribute`，那麼他就會自動創建一個引用他的全域變量。
```html
<ul id="my-todo-list">
   <li id="first">Write a book</li>
   ..
</ul>
```
對於上面的html會轉換為
```javascript
first; 
// <li id="first">..</li>

window["my-todo-list"]; 
// <ul id="my-todo-list">..</ul>
```
如果id的值對於 lexcal 來說是合法的那麼這個 id 的值便會被建立，若不是則會在全域物件中建立（window[...]），雖然這種將所有id的值都建立為全域是舊版瀏覽器的行為，但是為了迎合一些舊版的網站只能暫時將他們保留，作為開發者能做的就是盡量不要去使用這些被自動創建出來的全域變量。

### what's in a(window) Name?
```javascript
var name = 42;
console.log(name, typeof name); // "42", string
```
`window.name` 他是在瀏覽器中定義的全域，他是全域物件中的一個屬性（property），我們使用 var 宣告了 name，但是他卻沒有 shadowing 全域物件的 porperty，這代表著當全域物件中已經有這個名字的 property，那麼使用 var 宣告一個一樣名字的全域變數時`var 的宣告會被忽略`，這與我們上面看得的使用let宣告的結果並不一樣。

但是奇怪的是我們將這個全域的 `number`name console 出來後發現他雖然也是 42 但是他的資料結構變為 `string`，這是因為對於 window 物件來說，要取得這個 name 屬性他所使用的是`getting/setting` 方法，這個方法要求他的值必須是字串。

### Developer Tools Console/REPL
對於開發者工具來說，雖然他們也確實的再處理JS程式，但是為了給使用者良好的使用體驗所以會額外做一些處理，在某些情況下開發者工具並不會處理JS程序的所有步驟，這個可能會造成程式與開發者工具之間的差異，舉例來說，
開發者工具在對於一些JS的錯誤相對放寬，所以當輸入一個程式到開發者工具中，有可能不會報錯。

其中的差異有以下幾種：
- 全域作用域行為
- Hoisting
- 在全域作用域中使用塊狀作用域宣告關鍵字（let/const）

所以雖然使用 console/REPL 在確實有在全域作用域中處理程序，但這並不是很準確，雖然這些開發者工具會在一定程度上`模仿`全域作用域的行為，但是他僅僅是模仿並不是嚴格遵守，開發者工具優先考量開發者使用的便利性，所以行為會與真正的JS規範有一定的差距。

### ES Modules(ESM)
在ES6中引入了對模塊的概念（ES Modules），他能夠修改文件中的全域作用域是他最為明顯的影響之一。
```javascript
var studentName = "Kyle";

function hello() {
    console.log(`Hello, ${ studentName }!`);
}

hello();  // Hello, Kyle!
export hello;
```
若是使用 ES Modules 來加載那麼他的運行方式和一般程式運行的方式一樣，但是從應用程式的角度來說卻是不一樣的，儘管在模塊的頂層宣告，但是在全域作用域中 `stydentName` 和 `hello(...)` 任然不是全域變數，他們是屬於模塊作用域中（模塊中的全域）的。

所以說他並不是被加入到全域物件中，但是這並不代表他不能夠在全域中被訪問到，只是不能通過在 module 中的全域來建立真正意義上的全域變數。

module 中的全域作用域是真正全域作用域的下一級，這代表全域作用域中的宣告可以提供給 module 使用。

ESM的出現就是為了讓開發者減少對全域作用域的依賴，在全域作用域中可以 import 需要的 module，這樣可以減少對於全域或全域物件的用法。

### Node
Node會處理他加載的每一個.js文件包括啟動Node的主要 module，而這樣的效果是 Node 程式的頂層並不是全域作用域。
```javascript
var studentName = "Kyle";

function hello() {
    console.log(`Hello, ${ studentName }!`);
}

hello(); // Hello, Kyle!
```
在處理之前，Node 會將程式碼包裝近一個 function 中，所以 `var` 和 `function` 會在一個 function 中被宣告而不是視為在全域作用中宣告，就像下面的程式碼。
```javascript
function Module(module,require,__dirname,...) {
    var studentName = "Kyle";

    function hello() {
        console.log(`Hello, ${ studentName }!`);
    }

    hello(); // Hello, Kyle!
    module.exports.hello = hello; //export variables
}
```
Node 實際上會運行你調用的 module，所以上面的程式碼中可以了解到為什麼 `studentName` 和 `hello(...)` 不是全域宣告而是 module 範圍的宣告。

Node 定義了一些如 `require(...)` 的全域變量，但他們實際上並不是真正定義在全域作用域中，他們只是被`注入`到每一個 module 中，所以要在Node中定義實際的全域變量需要將他加入到一個Node 自動提供的 `globals` 屬性中，當然他也不是真正的 `globals`，他只是實際`全域作用域的一個 reference`，類似於瀏覽器中的 `window`。


# Global This
在 ES2020 中，JS 終於定義了對全域物件標準化的引用 `flobalThis`，所以理論上可以使用 globalThis 代替上面介紹的所有方法。

參考文獻：
[You Don't Know JavaScript 2nd](https://github.com/getify/You-Dont-Know-JS/blob/2nd-ed/scope-closures/ch4.md)