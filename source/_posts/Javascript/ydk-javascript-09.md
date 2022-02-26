---
title: You Don't Know JavaScript [Scope & Closures] - Using Closures?
date: 2020-10-15 17:57:22
tags:
- Javascript
- Front-end
- Scope & Closures

categories:
- You Don't Know JavaScript
---

目前為止我們都專注在解釋辭法範圍，以及他會對程式中的變量與使用產生什麼影響，本章節會將角度轉移到`closure`，回顧一下[Limiting Scope Exposure ?](https://ithelp.ithome.com.tw/articles/10252422)中所提到的POLE原則，我們應該使用塊狀作用域（或函式作用域）來減少變量的洩露，這有助於程式的安全性與可維護性並可以避免命名衝突等等的錯誤，而closure就是建立在這個觀念之上，可以將變量`封裝不讓他們洩露到外部／全域中，並且保留內部函式的訪問權限`，函式會通過closure記住引用的作用域變量。

我們在[Limiting Scope Exposure ?](https://ithelp.ithome.com.tw/articles/10252422)提到的範例中（factorial(...)），我們嘗試了在自身作用域中訪問了外部的callback function這就是closure，closure是對於編成發明中重要的特徵之一，他是主要編成範例的基礎，它包含`function programming(FP)`,`module`,`class-oriented design`

<img src="https://www.engelglobal.com/fileadmin/_processed_/1/5/csm__NG_5336_Large_df8012f928.jpg" alt="drawing" width="860"/>

<!-- more -->

# See the Closure
closure是`function的行為`，object與class都不具有closure只有function才有。
```javascript
// outer/global scope: RED(1)
function lookupStudent(studentID) {
    // function scope: BLUE(2)

    var students = [
        { id: 14, name: "Kyle" },
        { id: 73, name: "Suzy" },
        { id: 112, name: "Frank" },
        { id: 6, name: "Sarah" }
    ];

    return function greetStudent(greeting){
        // function scope: GREEN(3)

        var student = students.find(
            student => student.id == studentID
        );

        return `${ greeting }, ${ student.name }!`;
    };
}

var chosenStudents = [
    lookupStudent(6),
    lookupStudent(112)
];

// accessing the function's name:
chosenStudents[0].name; // greetStudent
chosenStudents[0]("Hello"); // Hello, Sarah!
chosenStudents[1]("Howdy"); // Howdy, Frank!
```
上面的程式中，在最外層定義了一個lookupStudent(...)並且在他的內部return了greetStudent(...);呼叫了lookupStudent(...)兩次並將他的結果都存放在`chosenSrudents`陣列中。

通過使用`.name`發現其實lookupStudent(...)返回的是一個greetStudents(...)的實例，正常來說對於一個function的呼叫結束後，會將此funciton中的內部的變量都丟棄（garbage collected）;但是對於這裡來說有些不同，greetStudent他內部中有使用到了外部（lookupStudent(...)）的變量（studentID,students），對於這個`內部函式引用到外部函式變量`的行為就稱為closure。

closure允許greetStudent(...)繼續訪問外部函式作用域的變量，儘管他已經被完成呼叫（garbage collected）students和studentID的實例不會進到GC'd而是被`保留在記憶體中`，所以在greetStudent(...)在調用這些變量實體的時候他們都仍然存在，所以說如果JS中沒有closure那們在lookupStudet(...)執行完成後會立即會立即清除他的作用域並將`studentID`與`students`收近GC，就因為這個特性才能讓我們在呼叫`choseStudents[1]("Hello")`的時候不會因為`studentID`與`students`不存在而導致ReferenceError。

## Adding Up Closrues
```javascript
function adder(num1) {
    return function addTo(num2){
        return num1 + num2;
    };
}

var add10To = adder(10);
var add42To = adder(42);

add10To(15);    // 25
add42To(9);     // 51
```
內部的addTo(...) closing 外部(adder(...))的num1，所以在adder(...)執行結束後 訪問得到num1的值，所以`add10To`會保留著`num1 = 10;`讓`add10To(15);`可以調用到存在在記憶體中的`num1 = 10;`，讓結果可以輸出10+15 = 25。

對於每一個adder(...)來說他內部都各自定義了自己內部的addTo(...)，因此每一個都是各自獨立的closure，所以上面程式中的add10To(..)和add42To(..)他們其實都有各自獨立的closure，即使closure是基於編譯時期處理的lexcial scope但是他仍被視為函數實例運行的特徵。

## Live Link,Not a Snapshot
很多人認為closure的這個可以讀取到保留變量的行為，是讀取到這個保留值的參照，但這其實是錯誤的，因為closure的這個行為是是確實是`保留對這個變量的訪問`，除此之外不限於只讀取這個值，還可以對這個值進行操作，對於closure的function我們可以任意的使用這個變量(讀/寫)並且在整個函數中都可以使用，這就是closure強大的地方。

```javascript
function makeCounter() {
    var count = 0;

    return function getCurrent() {
        count = count + 1;
        return count;
    };
}

var hits = makeCounter();

hits();     // 1
hits();     // 2
hits();     // 3
```
`const` closed over 內部的getCurrent(...)這樣他可以被保存在記憶體中不會被GC，所以當調用`hits`的時候會不斷返回更新後的count值（遞加）。

雖然closure通常都來自於函式，但是實際上是不一定需要的，只要`外部作用域中有一個內部函式就可以`。
```javascript
var hits;
{   // an outer scope (but not a function)
    let count = 0;
    hits = function getCurrent(){
        count = count + 1;
        return count;
    };
}
hits();     // 1
hits();     // 2
hits();     // 3
```
在一個塊狀作用域中宣告一個function，這樣就可以形成一個closure，但是由於`FiB`的關係，所以使用function expression而不是function declaration。

對於closure還有一個令人誤會的地方，closure他保存的是`變量`而不是`變數`。
```javascript
var studentName = "Frank";

var greeting = function hello() {
    // we are closing over `studentName`,
    // not "Frank"
    console.log(
        `Hello, ${ studentName }!`
    );
}
studentName = "Suzy";

greeting(); // Hello, Suzy!
```
上面的程式中，很多人會以為closure保存的事studentName的值（Frank），但實際是他保留的是`studentName這個變量`，所以當後面重新assignment給studentName的時候值就會發生變化，除了這個之外還有一個經典的錯誤，for loop。
```javascript
var keeps = [];

for (var i = 0; i < 3; i++) {
    keeps[i] = function keepI(){
        // closure over `i`
        return i;
    };
}

keeps[0]();   // 3 -- WHY!?
keeps[1]();   // 3
keeps[2]();   // 3
```
你可能會認為`keeps[0]();`應該要return 0因為他是在i = 0的時候賦予給他的，但是這要是不對的，記住`closure保存的是變量而不是變量裡面的值`，由於for loop的結構會讓我們誤以為他的每次迭代都會宣告一個新的i，但由於i是由var宣告的所以整個迴圈中都只會有一個var（參考[The (Not So) Secret Lifecycle of Variables](https://ithelp.ithome.com.tw/articles/10250873)），雖然三個`keeps(...)`都有各自的closures，但是本質上他們共享一個i，所以當迴圈結束時三個function才都return 3，所以每一個變量都只能存取一個值，所以我們可以嘗試在每個迴圈中都新增一個各自的變量。
```javascript
var keeps = [];

for (var i = 0; i < 3; i++) {
    // new `j` created each iteration, which gets
    // a copy of the value of `i` at this moment
    let j = i;

    // the `i` here isn't being closed over, so
    // it's fine to immediately use its current
    // value in each loop iteration
    keeps[i] = function keepEachJ(){
        // close over `j`, not `i`!
        return j;
    };
}
keeps[0]();   // 0
keeps[1]();   // 1
keeps[2]();   // 2
```
在每個迴圈中都宣告一個屬於自己的變量(let宣告)，然後賦予它當前i的值，因為j是定義在個迴圈中的變量所以所以不會被重新賦值，closure的對象就從i變為各自定義的j，所以輸出的值就會是預期的。

如果我們在迴圈中使用非同步的行為（setTimeout(...),keepEachJ(...)...）也會出現這個情況，在[The (Not So) Secret Lifecycle of Variables](https://ithelp.ithome.com.tw/articles/10250873)中提到如果在loop中使用let宣告，他會在每一次的迭代中都創建一個新的變量，這正是我們希望它發生的。
```javascript
var keeps = [];

for (let i = 0; i < 3; i++) {
    // the `let i` gives us a new `i` for
    // each iteration, automatically!
    keeps[i] = function keepEachI(){
        return i;
    };
}
keeps[0]();   // 0
keeps[1]();   // 1
keeps[2]();   // 2
```

## What If I Can't See It?
如果我們在操作function的時候，我們需要產生一個closure，當我們確實產生了一個closure後他將需要的變量保存下來，但我們卻沒有去訪問過這麼變數，那麼這個closure就不會存在。
```javascript
function lookupStudent(studentID) {
    return function nobody(){
        var msg = "Nobody's here yet.";
        console.log(msg);
    };
}

var student = lookupStudent(112);
student(); // Nobody's here yet.
```
雖然closure將`studentID`保存下來，但是在內部的nodody(...)卻只使用自身宣告的變數`msg`而沒有用到外部的`studentID`，這樣會讓JS引擎知道在lookupStudent(...)執行完之後沒有東西需要使用到`studentID`，便會將它從記憶體中清除。

如果沒有去呼叫內部的函式，那麼我們也無法觀測到closure的存在。
```javascript
function greetStudent(studentName) {
    return function greeting(){
        console.log(
            `Hello, ${ studentName }!`
        );
    };
}

greetStudent("Kyle");
// nothing else happens
```
外部的function他確實被呼叫並且有return一個內部的function，這雖然是一個會產生closure的行為，但是由於內部的function並沒有被呼叫所以就算以技術上來說JS確實為她創造了一個closure，但是無法觀察也沒以意義。

## Observable Definition
> Closure is observed when a function uses variable(s) from outer scope(s) even while running in a scope where those variable(s) wouldn't be accessible.

對於closure定義而言有一些關鍵的部分：
- function必須要被呼叫
- 必須引用一個外部作用域的變量
- Must be invoked in a different branch of the scope chain from the variable(s)


# The Closure Lifecycle and Garbage Collection (GC)
由於closure的本質與function息息相關，所以只要仍有對該function的引用，則closure則會持續存在，換句話說如果有10個function都closure同一個變量，隨著時間流逝其餘九個都不在使用這個變量，就算只剩下一個那麼這個closure依然會繼續存在，等到最後一個funciton都不在使用這個變量，那麼這個closure就會消失。

如果沒有這個機制的話，若在記憶體中無論是否有用到都不斷保存著我們之前所使用過的變量，那麼總有一天會導致記憶體內存不足。

## Per Variable or Per Scope?
對於closure來說，他是保留引用的外部變量還是將整個作用域以及所有變量？以技術上來說closure保存的是變量而不是作用域，但實際的情況會更為複雜。
```javascript
function manageStudentGrades(studentRecords) {
    var grades = studentRecords.map(getGrade);

    return addGrade;

    // ************************

    function getGrade(record){
        return record.grade;
    }

    function sortAndTrimGradesList() {
        // sort by grades, descending
        grades.sort(function desc(g1,g2){
            return g2 - g1;
        });

        // only keep the top 10 grades
        grades = grades.slice(0,10);
    }

    function addGrade(newGrade) {
        grades.push(newGrade);
        sortAndTrimGradesList();
        return grades;
    }
}

var addNextGrade = manageStudentGrades([
    { id: 14, name: "Kyle", grade: 86 },
    { id: 73, name: "Suzy", grade: 87 },
    { id: 112, name: "Frank", grade: 75 },
    // ..many more records..
    { id: 6, name: "Sarah", grade: 91 }
]);

addNextGrade(81);
addNextGrade(68);
// [ .., .., ... ]
```
上面的程式中`grades`通過closure保留在addGrade(...)之中，所以在每次呼叫addNextGrade(...)時，都可以再次訪問到grades並對他從新排列，但是記住，這個closure是對於`grades`這個變量而不是裡面的數值。

除了`grades`被closure之外，由於addGrade(...)中呼叫的sortAndTrimGradesList(...)，這代表就算manageStudentGrades(...)已經結束了但sortAndTrimGradesList(...)還是被closure給保留了下來以便addGrade(...)可以持續調用，

根據closure的定義，由於內部函數沒有使用到`getGrade(...)`與`studentRecords`所以不會對他們進行closure，在manageStudentGrades(...)結束後他們就會被清除。

但是凡事都有例外
```javascript
function storeStudentInfo(id,name,grade) {
    return function getInfo(whichValue){
        // warning:
        //   using `eval(..)` is a bad idea!
        var val = eval(whichValue);
        return val;
    };
}

var info = storeStudentInfo(73,"Suzy",87);

info("name"); // Suzy
info("grade"); // 87
```
上面的程式碼中，我們使用了`eval`做了一些小動作，就算在後面的info(...)中沒有使用到所有的變量，但是JS依然將所有的變量都保存起來。

在與許多現在的JS引擎中，他們都對於`刪除從未明確引用的closure範圍中變量`這件事不斷的進行優化，但是正如我們上面所使用的方法(eval)，在某些情況下JS並不會如預期的那樣操作，因為會有這些意外事件發生，所以作者建議不應該隨便高估他的適用性，如果有一個較大的值被closure給保留在記憶體中但他卻是不被需要的(特殊狀況)，這樣會對整個程式產生記憶體不夠的威脅，所以作者認為手動丟棄這些大量且不被需要的值是比較安全的。
```javascript
function manageStudentGrades(studentRecords) {
    var grades = studentRecords.map(getGrade);

    // unset `studentRecords` to prevent unwanted
    // memory retention in the closure
    studentRecords = null;

    return addGrade;
    // ..
}
```
雖然在我們分析closure的時候，我們知道因為studentRecords沒有被調用到，所以JS`自動`的將它清除（不產生closure），但是作者希望如果開發者知道這個變量不會再被使用，那麼可以手動的將它的值清除以降低錯誤的發生，雖然技術上來說`getGrade(...)`也是不會再被使用到的，如果這個function也會消耗大量內存那麼也建議手動將它清除，但是在上面的例子中`getGrade(...)`並不需要做這樣的處理。

所以了解closure在程式中出現的位置以及他保存了哪些變量是很重要的，應該仔細管理這些保存的值以便讓他使用最低限度的需求而不浪費內存。


# Why Closure?
若我們不使用closure的情況下建立一個按鈕，在點擊他後透過AJAX發送一些數據。
```javascript
var APIendpoints = {
    studentIDs:
        "https://some.api/register-students",
    // ..
};

var data = {
    studentIDs: [ 14, 73, 112, 6 ],
    // ..
};

function makeRequest(evt) {
    var btn = evt.target;
    var recordKind = btn.dataset.kind;
    ajax(
        APIendpoints[recordKind],
        data[recordKind]
    );
}

// <button data-kind="studentIDs">
//    Register Students
// </button>
btn.addEventListener("click",makeRequest);
```
`makeRequest(...)`接收一個來自clich事件的`evt`物件，他需要從目標按鈕中檢索data-kind attribute並使用用這個值去決定AJAX需要請求哪些數據，雖然這麼做事可以的，但是由於每一次都需要去讀取DOM的attrubute，所以會導致效率低下，所以我們可以試著使用closure的方式改寫。
```javascript
var APIendpoints = {
    studentIDs:
        "https://some.api/register-students",
    // ..
};

var data = {
    studentIDs: [ 14, 73, 112, 6 ],
    // ..
};

function setupButtonHandler(btn) {
    var recordKind = btn.dataset.kind;

    btn.addEventListener(
        "click",
        function makeRequest(evt){
            ajax(
                APIendpoints[recordKind],
                data[recordKind]
            );
        }
    );
}

// <button data-kind="studentIDs">
//    Register Students
// </button>

setupButtonHandler(btn);
```
在呼叫setupButtonHandler的時候就檢索了data-kind attraubute並將他賦予給recordKind，然後由內部的makeRequest(...)使用其值來發送將對應的Request，通過closure我們將recordKind儲存讓內部的makeRequest(...)可以隨時使用它，我們也可以將request的值利用closure保存下來。
```javascript
function setupButtonHandler(btn) {
    var recordKind = btn.dataset.kind;
    var requestURL = APIendpoints[recordKind];
    var requestData = data[recordKind];

    btn.addEventListener(
        "click",
        function makeRequest(evt){
            ajax(requestURL,requestData);
        }
    );
}
```
將requestURL與requestData透過closure保存起來，這樣更容易理解而且效率也更好。

# Summary
本章節中介紹了什麼事closure，他的定義與他所帶來的好處。

Closure的精神：
- Observational:closure是一個函數實例，即使將這個函數傳遞給其他作用域並且在其他作用域中調用，他也會記住其外部的變量(儲存至記憶體中)。
- Implementational:closure is a function instance and its scope environment preserved in-place while any references to it are passed around and invoked from other scopes.

Closure的好處：
- 通過保存之前確定的數據而不必每次都重新計算，closure可以提高效率。
- closrue可以通過將變量封裝還提高可讀性與限制作用域的暴露，也同時保障了這個變量可以在將來使用，由於不需要在每次調用都重新傳遞保留的信息，所以更有利於function之間的互動。

參考文獻：
[You Don't Know JavaScript -2nd](https://github.com/getify/You-Dont-Know-JS/blob/2nd-ed/scope-closures/ch7.md)
