---
title: Data Structure - Listed-List
date: 2022-04-13 17:14:13
tags:
- CS

categories:
- Data Structure and Algorithm
---

本章開始會介紹基本的 `Data Structure` 比如 `Linked List`, `Stack and Queue`, `Hash Table` 等等，並且會介紹如何利用 `Javascript` 實作這些 Data structure 的功能。

而本章將介紹 `Linked List` 這種資料結構。

<img src="https://media.geeksforgeeks.org/wp-content/cdn-uploads/gq/2013/03/Linkedlist.png" alt="drawing" width="900"/>

<!-- more -->

# What is Linked List?
- In computer science, a linked list is a linear collection of data element whose order is not given by their physical placement in memory. Instead, each element points to the next.
- A data structure that contains only head and length property.
- Link list consists of nodes, and each node has a value (number, string, array, or anything) and a pointer to another node or null.

<img src="https://ithelp.ithome.com.tw/upload/images/20220626/201247674HnuqO93RV.png" alt="drawing" width="860"/>

---

# Advantages of Linked List
- Elements can be inserted into linked lists indefinitely, while an array will eventually either fill up or need to be resized.
- Vert fast insertion and deletion compared to Array.

## Insertion
- 將原先指向 null 的 Node 重新指向新的 item 即可
<img src="https://ithelp.ithome.com.tw/upload/images/20220626/20124767WCSlJgE1it.png" alt="drawing" width="860"/>

- 將原先指向 Hank 的 Node 重新指向到新的 item 並將新 item 的 Node 指向 Hank 完成串連
<img src="https://ithelp.ithome.com.tw/upload/images/20220626/201247679aV7tEznZI.png" alt="drawing" width="860"/>

## Deletion
要移除 `Jimmy` 只需要將原先指向他的 Node 重新指向下一個 item 即可。
<img src="https://ithelp.ithome.com.tw/upload/images/20220626/20124767HztaU95UR0.png" alt="drawing" width="860"/>

---

# Accomplish Linked List
```javascript
class Node {
  constructor(value) {
    this.value = value;
    this.next = null;
  }
}

class LinkedList {
  constructor() {
    this.head = null;
    this.length = 0;
  }
}
```

## Linked List Push
```javascript
class LinkedList {
  constructor() {
    this.head = null;
    this.length = 0;
  }

  push(value) {
    let newNode = new Node(value);
    if (this.head === null) {
      this.head = newNode;
    } else {
      let currentNode = this.head;
      // move head to last one
      while (currentNode.next !== null) {
        currentNode = currentNode.next;
      }
      currentNode.next = newNode;
    }
    this.length++;
  }

  printAll() {
    if (this.length === 0) {
      console.log("Nothing in this linked list.");
      return;
    }
    let currentNode = this.head;
    while (currentNode !== null) {
      console.log(currentNode.value);
      currentNode = currentNode.next;
    }
  }
}
```

## Linked List Pop
```javascript
class LinkedList {
  constructor() {
    this.head = null;
    this.length = 0;
  }

  pop() {
    if (!this.head) return null;
    else if (this.length === 1) {
      let tmp = this.head;
      this.head = null;
      this.length = 0;
      return tmp;
    } else {
      let currentNode = this.head;
      for (let i = 1; i <= this.length - 2; i++) {
        currentNode = currentNode.next;
      }
      let tmp = currentNode.next;
      currentNode.next = null;
      this.length--;
      return tmp;
    }
  }
}
```
<img src="https://ithelp.ithome.com.tw/upload/images/20220626/20124767ORTLcFBOoM.png" alt="drawing" width="860"/>

## Linked List Shift
```javascript
class LinkedList {
  constructor() {
    this.head = null;
    this.length = 0;
  }

  shift() {
    if (!this.head) return null;
    else if (this.length === 1) {
      let tmp = this.head;
      this.head = null;
      this.length = 0;
      return tmp;
    } else {
      let tmp = this.head;
      this.head = this.head.next;
      this.length--;
      return tmp;
    }
  }
}
```

## Linked List Unshift
```javascript
class LinkedList {
  constructor() {
    this.head = null;
    this.length = 0;
  }

  unshift(value) {
    const newNode = new Node(value);
    if (this.head === null) {
      this.head = newNode;
    } else {
      const tmp = this.head;
      this.head = newNode;
      this.head.next = tmp;
    }
    this.length++;
  }
}
```

## Linked List InsertAt
```javascript
class LinkedList {
  constructor() {
    this.head = null;
    this.length = 0;
  }

  insertAt(index, value) {
    if (index > this.length || index < 0) return null;
    else if (index === 0) this.unshift(value);
    else if (index === this.length) this.push(value);
    else {
      const newNode = new Node(value);
      let currentNode = this.head;
      for (let i = 0; i < index - 1; i++) {
        currentNode = currentNode.next;
      }
      const tmp = currentNode.next;
      currentNode.next = newNode;
      newNode.next = tmp;
      this.length++;
    }
  }
}
```

## Linked List RemoveAt
```javascript
class LinkedList {
  constructor() {
    this.head = null;
    this.length = 0;
  }

  removeAt(index) {
    if (index > this.length - 1 || index < 0) return null;
    else if (index === 0) this.shift();
    else if (index === this.length) this.pop();
    else {
      let currentNode = this.head;
      for (let i = 0; i < index - 1; i++) {
        currentNode = currentNode.next;
      }
      currentNode.next = currentNode.next.next;
      this.length--;
    }
  }
}
```

## Linked List Get
```javascript
class LinkedList {
  constructor() {
    this.head = null;
    this.length = 0;
  }

  get(index) {
    if (index > this.length - 1 || index < 0) return null;
    else if (index === 0) return this.head.value;
    else {
      let currentNode = this.head;
      for (let i = 0; i < index; i++) {
        currentNode = currentNode.next;
      }
      return currentNode.value;
    }
  }
}
```

---

# Disadvantages of Linked Lists
1. They use memory then arrays because of the storage used by their pointers.
2. Nodes in a linked list must be read in order from the beginning as linked lists are inherently sequential access.
3. Nodes are stored noncontiguous, greatly increasing the time periods required to access individual elements within the list, especially with a CPU cache.
4. Difficulties arise in linked lists when it comes to reverse traversing. For instance, singly-linked lists are cumbersome to navigate backward and while doubly linked lists are somewhat easier to read, memory is consumed in allocating space for a back pointer.

---

# Difference between Array and Linked List

## Linked List
- No Not Have Indices
- Connection between nodes are a `next` pointer
- Random access is not allowed (Must go through each item before accessing)

## Array
- Items are indexed in order
- Insertion and deletion are expensive
- Elements can quickly be accessed with a specific index

## Overview of Linked List
 '| Array | Linked List
------------- | ------------ | ------------
Accessing Elements | O(1) | O(n)
Insert and Remove from the Beginning | O(n) | O(1)
Insert and Remove from the End | O(1) | O(n)
Insert and Remove from the Middle | O(n) | O(n)

---

# Doubly Linked List
<img src="https://ithelp.ithome.com.tw/upload/images/20220626/20124767Wb6BKGUe9G.jpg" alt="drawing" width="860"/>

相比於 Singly Linked List 而言，Doubly Linked List 會連結頭尾兩個 Node，好處是可以由後方往前移動，壞處是需要使用較多的記憶體空間來儲存前方的 Node。

---

# Reference
- [資料結構與演算法 (JavaScript)](https://www.udemy.com/course/algorithm-data-structure/)