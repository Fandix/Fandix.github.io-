---
title: 番外篇：What is Dependency Injection?
date: 2021-09-18 14:53:32
tags:
- Angular
- Front-end
- Typescript
- Dependency Injection

categories:
- 2021 鐵人賽
---

在 Angular 中 Dependency Injection 是個非常大的特點，Dependency Injection 是一種設計模式，主要是用於將相關的程式由`外部注入進 component 中`而不是在 component 中創建以實現`解耦`的目的，可以有效地減低維護的複雜度。

![img](https://medicalweightlosscentersofamerica.com/wp-content/uploads/2021/06/HowLongDoVitaminB12InjectionsLast.jpg)

<!-- more -->


# What is Coupling(藕合)
Coupling(藕合) 又可以稱為 Dependency(依賴)，簡單來說就是程式與程式之間互相具有依賴性，在一個專案中如果彼此的依賴性越高則代表`越難維護`。

一般在開發專案時會將功能相關的部分結合起來作為一個 Class 以提供給其他地方使用，當某個地方需要使用到這個 Class 實則需要再對應的地方使用關鍵字 `new` 產生一個新的物件，這樣才可以使用到裡面的 `method` 或 `property`，來舉個例子某一個國小的教室 (ClassRoom) 需要一個老師，老師的功能有 `Teaching`, `Test`, `QA` 這三種功能。
```javascript
class MathTeacher {
  constructor() { }

  Teaching() {
    console.log('Teach Math');
  }

  Test() {
    console.log('Test 1 + 1 = 2');
  }

  QA() {
    console.log('QA Time');
  }
}
```
而現在的情境是在這個教室中有一個數學老師在執行這三種功能並執行它
```javascript
class ClassRoom {
  mathTeach = new MathTeacher();

  ClassTeaching() {
    this.mathTeach.Teaching();
  }

  ClassTest() {
    this.mathTeach.Test();
  }

  ClassQA() {
    this.mathTeach.QA();
  }
}

room = new ClassRoom();
room.ClassTeaching();
room.ClassTest();
room.ClassQA();
```

![https://ithelp.ithome.com.tw/upload/images/20220216/20124767IBMhMMKCLW.png](https://ithelp.ithome.com.tw/upload/images/20220216/20124767IBMhMMKCLW.png)

雖然這種寫法可以完成目標但如果今天需要將數學老師變為英文老師的話，那就需要將 `mathTeach = new MathTeacher();` 這邊更改為 `englishTeacher = new EnglishTeacher();`，這種因為一個程式變動而導致另一個程式也需要隨之修改的行為就稱為`高藕合(Coupling)`，如果專案很大的話這種高耦合會讓專案的維護與開發變得越來越困難。

# Dependency Injection (依賴注入)
為了減輕每個程式中互相的依賴，這時就誕生了 Dependency Injection 這種設計模式，他主要的目的是通過將有關連的程式利用注入的方式由外部注入至需要的程式，而不是像過去一樣在內部創建，而達成 DI 有兩種方法
##  使用建構子 (Contructor) 
首先跟剛剛一樣，新增一個Teacher 的 class (老師類別)，將老師的技能 (Methods) 寫入，可以隨需求更改教學的內容。
```typescript
class Teacher {
  constructor() { }

  Teaching() {
    console.log('Teach Math');
  }

  Test() {
    console.log('Test 1 + 1 = 2');
  }

  QA() {
    console.log('QA Time');
  }
}
```
接著對 ClassRoom Class 中修改 constructor 讓需要授課的老師 class 由外部注入進來
```typescript
class ClassRoom {
  constructor(private teacher: Teacher) {}  // (1)

  ClassTeaching() {
    this.teacher.Teaching();
  }

  ClassTest() {
    this.teacher.Test();
  }

  ClassQA() {
    this.teacher.QA();
  }
}
```
- (1): 利用 constructor 將原先依賴的程式注入進來

接著使用這個新的 Room Class
```typescript
const teach = new Teacher();         // (1)
const room = new ClassRoom(teach);   // (2)

room.ClassTeaching();
room.ClassTest();
room.ClassQA();
```
- (1): 在外部創建需要的 Class
- (2): 將需要的 Class 作為參數傳入 ClassRoom Class 中（注入）

![https://ithelp.ithome.com.tw/upload/images/20220216/20124767IBMhMMKCLW.png](https://ithelp.ithome.com.tw/upload/images/20220216/20124767IBMhMMKCLW.png)

## 使用 Setter
除了在 constructor 中宣告注入的 Class 之外，還可以利用 `setter` 做到相同的功能。
```typescript
class ClassRoom {
    private _teacher!: Teacher
    constructor() {}

    set setTeacher(teacher: Teacher) {   // (1)
        this._teacher = teacher;
    }

    ClassTeaching() {
        this._teacher.Teaching();
    }

    ClassTest() {
        this._teacher.Test();
    }

    ClassQA() {
        this._teacher.QA();
    }
}
```
- (1): 創建一個 `setter` 用於將外部依賴的 Class 注入進來

```typescript
const teach = new Teacher();     // (1)
const room = new ClassRoom();    // (2)
room.setTeacher = teach;         // (3)

room.ClassTeaching();
room.ClassTest();
room.ClassQA();
```
- (1): 在外部創建需要的 Class
- (2): 創建需要的 Class
- (3): 利用 `ClassRoom` 中的 `setter` 將外部依賴的 Class (Teacher) 注入

當使用了 Dependency Injection 這種設計模式後，你想要從數學老師變更為英文老師的話，只需要在 `Teacher` Class 中更改教學方式就好，其他都不用變就可以達到目的，可以打幅度的減低維護的成本

```typescript
class Teacher {
  constructor() {}

  Teaching() {
    console.log("Teach English");
  }

  Test() {
    console.log("Test A, B, C");
  }

  QA() {
    console.log("QA English Time");
  }
}
```
![https://ithelp.ithome.com.tw/upload/images/20220216/20124767hWndZPGwti.png](https://ithelp.ithome.com.tw/upload/images/20220216/20124767hWndZPGwti.png)

# Dependency Inversion Principle (依賴反轉)
> Dependency Inversion Principle：依賴反轉，又稱為依賴反向或依賴倒轉

當專案越來越大的時候，就需要比 Dependency Injection 更有彈性的設計模式的出現，接著我們把上面的例子改一下，現在我們從一間教室變成多間，一位老師也變成多位老師且每個老師都用相同的方法但不同的內容教課，可能想一想就會覺得非常的複雜，不過沒關係我們可以把他整理一下
1. 有一個地方專門培訓老師，所以教導出來的老師教學模式都一樣 （ 每個 Class 中有著相同的 method ）
2. 雖然教學模式一樣，但因為每個老師有不同的專業，所以教學的內容都不一樣
3. 將每位老師分配到不同的教室中
4. 每間教室的老師都用著相同的教學模式

## 有一個地方專門培訓老師，所以教導出來的老師教學模式都一樣 （ 每個 Class 中有著相同的 method ）
要每一個 Class 都有著相同的格式的最好方法就是建立一個 interface 去規定需要哪些教學方式（ method ）
```typescript
interface NormalSchool {
    Teaching: () => void;  // 教學
    Test: () => void;      // 考試
    QA: () => void;        // 提問
}
```

## 雖然教學模式一樣，但因為每個老師有不同的專業，所以教學的內容都不一樣
可以建立多個不同老師 ( Class )，透過 `implements` 將 `interface` 讓每個老師 ( Class ) 有相同的教學模式 ( methods )。
1. 數學老師
```typescript
class MathTeacher implements NormalSchool {
    Teaching() {
        console.log('Teach Math');
    }

    Test() {
        console.log('Test 1 + 1 = 2');
    }

    QA() {
        console.log('Math QA Time!')
    }
}
```
2. 英文老師
```typescript
class EnglishTeacher implements NormalSchool {
    Teaching() {
        console.log('Teach English');
    }

    Test() {
        console.log('Test A, B, C, D');
    }

    QA() {
        console.log('English QA Time!')
    }
}
```
3. Javascript 老師
```typescript
class JavascriptTeacher implements NormalSchool {
    Teaching() {
        console.log('Teach Javascript');
    }

    Test() {
        console.log('Test program');
    }

    QA() {
        console.log('program QA Time!')
    }
}
```

## 將每位老師分配到不同的教室中
修改 ClassRoom 我們把外部注入的對象這定為只接受去過師範大學的老師才可以進入這個教室
```typescript
class ClassRoom {
    constructor(private _teacher: NormalSchool) {}   // 外部注入的對象必須是 NormalSchool

    classTeaching() {
        this._teacher.Teaching();
    }

    classTest() {
        this._teacher.Test();
    }

    classQA() {
        this._teacher.QA();
    }
}
```

## 將每位老師放入教室中並使用著相同的教育方式但教授的是不同的科目
3間教室指定老師使用師範大學的教學方式，去教自己專業科目的內容。

```typescript  
const mathTeacher = new MathTeacher();                 // Create math teacher
const englishTeacher = new EnglishTeacher();           // Create english teacher
const javascriptTeacher = new JavascriptTeacher();     // Create javascript teacher

const room1 = new ClassRoom(mathTeacher);              // 將數學老師放入教室 1
const room2 = new ClassRoom(englishTeacher);           // 將英文老師放入教室 2
const room3 = new ClassRoom(javascriptTeacher);        // 將 Javascript 老師放入教室 3

room1.classTeaching();
room2.classTest();
room3.classQA();
```

![https://ithelp.ithome.com.tw/upload/images/20220216/20124767OuZTlaSPi3.png](https://ithelp.ithome.com.tw/upload/images/20220216/20124767OuZTlaSPi3.png)

上面的結果可以發現每間教室可以接受不同的老師，只要是從師範大學中出來的 ( implements NormalSchool ) 就可已放入，並且每個不同的老師都有著相同的教學方式 ( methods ) 但教授的內容卻是不同的，之後如果某個教室需要更換老師或是需要新增一個新的老師，其他部分就不需要更改也不會受到影響，這就是 `Dependency Inversion Principle`。

# 結論
本章介紹了什麼是 Dependency Injection 的概念與在 Typescript 中如何實現，在我剛接觸 Angular 的時候對於 Dependency Injection 來說可以說是一頭霧水，只知道只要在 Constructor 裡面宣告就可以用了，但根本不知道這麼做的目的是什麼，直到我去看了 Jun 大大對於 Dependency Injection 的一些概念與舉例所以才比較明白是什麼，不過 Jun 大大的文章是用 Java 寫的，所以本章算是把 Jun 大大的文轉換為 Typescript 做個紀錄，大家可以去看 Jun 大大原本的文章或是其他文章，都寫得非常好！

# Reference
- [淺談Coupling 耦合, Dependency Injection 依賴注入及Dependency Inversion Principle 依賴反轉。](https://medium.com/appxtech/%E6%B7%BA%E8%AB%87coupling-%E8%80%A6%E5%90%88-dependency-injection-%E4%BE%9D%E8%B3%B4%E6%B3%A8%E5%85%A5%E5%8F%8Adependency-inversion-principle-%E4%BE%9D%E8%B3%B4%E5%8F%8D%E8%BD%89-b2ae7f746383)
- [Top 5 TypeScript dependency injection containers](https://blog.logrocket.com/top-five-typescript-dependency-injection-containers/)