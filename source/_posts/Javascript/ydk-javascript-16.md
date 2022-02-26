---
title: You Don't Know JavaScript [This & Object Prototypes] - Object [番外 - getter/setter]
date: 2020-11-02 21:34:53
tags:
- Javascript
- Front-end
- This & Object Prototypes

categories:
- You Don't Know JavaScript
---

我們在[Object [下]](https://ithelp.ithome.com.tw/articles/10254011)中有提到 `getter / setter`，由於這個部分在書中的解釋是比較舊的版本，所以找過新版的資料後整理在此篇章中。

getter是一種獲取特定屬性的方法，setter是設置特定屬性的方法，getter和setter常被用在：
1. 對物件初始化
2. 可以隨時添加屬性到任何物件中

<img src="https://kommunikation.uni-freiburg.de/pm-en/online-magazine/experience-and-get-involved/Neue_StartseiteMelpomene_Fotolia540.jpg" alt="drawing" width="860"/>

<!-- more -->

# Getter(the get Keyword)
有時候物件屬性會需要`回傳動態的數據`，或要在`不使用明確的方法呼叫（不直接訪問物件屬性）下反映出內部變數`的狀態，這樣可以使用getter來達到這個目的，
```javascript
const obj = {
  log: ['a', 'b', 'c'],
  count: 0,
  get latest() {
    if (this.log.length === 0) {
      return undefined;
    }
    obj.count ++; // 將count綁定給get
    return this.log[this.log.length - obj.count]; // 將log綁定給get
  }
};

for(let i = 0; i< 4; i++){
	console.log(obj.latest); // 呼叫get method以訪問這些被綁定給getter的屬性
}
/*
    c
    b
    a
    undefined
*/
```

## 移除getter
getter可以使用`Delete`操作符移除。
```javascript
const obj = {
  log: ['a', 'b', 'c'],
  count: 0,
  get latest() {
    if (this.log.length === 0) {
      return undefined;
    }
    obj.count ++; // 將count綁定給get
    return this.log[this.log.length - obj.count]; // 將log綁定給get
  }
};

console.log(obj.latest); // c
delete obj.latest;
console.log(obj.latest); // undefined
```

## 新增get到現有物件中
我們除了在一開始建立物件的時候定義getter之外，可以使用`defineProperty`添加getter給`現有物件中`。
```
const obj = {
    a: 0
}

Object.defineProperty(obj, 'b', {
    get: function(){
        return this.a + 1;
    }
})
console.log(obj.b); // 1
```

# Setter
setter能在嘗試`修改指定屬性時執行給定的函式`，Setter最常與Getter一同建立虛擬屬性（pseudo-property）。
```javascript
const language = {
  set current(name) {
    this.log.push(name);
  },
  log: []
};

language.current = 'EN';
language.current = 'FA';

console.log(language.log); // ['EN', 'FA']
```

## 移除setter
setter可以使用`Delete`操作符移除。
```javascript
const language = {
  set current(name) {
    this.log.push(name);
  },
  log: []
};

language.current = 'EN';
console.log(language.log); // 'EN'

delete language.current;
console.log(language.current); // undefined
```

## 新增set到現有物件中
setter一樣也可以通過`defineProperty`添加setter給`現有物件中`。
```javascript
const obj = {
    a: 0
};

Object.defineProperty(obj, 'b', { set: function(x) { this.a = x / 2; } });

o.b = 10; 
console.log(o.a) // 5
```


# 結論
可以使用 getter / setter 對物件中的屬性進行一些操作，特別是如果想在返回屬性或設置它們之前對值做一些特殊的事情，兩個方法都可以使用`defineProperty`方法添加 getter/setter 到現有的物件中，也可以使用`delete`將他們移除。


# Reference
- [Getter 和 Setter 的定義](https://developer.mozilla.org/zh-TW/docs/Web/JavaScript/Obsolete_Pages/Obsolete_Pages/Obsolete_Pages/%E6%96%B0%E7%89%A9%E4%BB%B6%E7%9A%84%E5%BB%BA%E7%AB%8B/Getter_%E5%92%8C_Setter_%E7%9A%84%E5%AE%9A%E7%BE%A9)
- [getter](https://developer.mozilla.org/zh-TW/docs/Web/JavaScript/Reference/Functions/get)
- [setter](https://developer.mozilla.org/zh-TW/docs/Web/JavaScript/Reference/Functions/set)