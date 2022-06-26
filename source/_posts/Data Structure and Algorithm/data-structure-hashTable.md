---
title: Data Structure - Hash Table
date: 2022-04-15 17:30:56
tags:
- CS

categories:
- Data Structure and Algorithm
---

本章會介紹 `Hash Table` 這種 Data structure，他主要是將大量的資料利用一種算法，將他們歸類於不同的 `Group`，所以當需要查詢某一個 Item 時，可以先找到該 Item 所屬的 Group 之後再進到這個 Group 中尋找，可以有效的減少 Search 所花費的時間。

<img src="https://hughewilliams.files.wordpress.com/2012/09/450px-hash_table_5_0_1_1_1_1_1_ll-svg.png" alt="drawing" width="900"/>

<!-- more -->

# Hash Function
- `Hash` is ... not a very commonly used word in English. Mostly people know it with `hash brown`.
- In computer science, hash function is a commonly used idea. There’s tons of different hash functions. Ex. Passwords are hashed before storing into database.
JavaScript objects and arrays are hashed.
- The principle of a hash function is to convert one value to another.
- From the problem we observed before, let’s think about how to solve it... why not hashed all players ID into a small integer?

## Hash Function I - Division Method
- Advantage of using division method is FAST.
- Disadvantage:
    - Ideally, Integer m has to be a number that it `far` enough from 2^p, where P is a positive integer. (Because 10^P is divisible by 2^P, taught in Number Theory).
    - If naming convention of objects are similar, then it has a chance to get MANY MANY MANY collisions.

<img src="https://ithelp.ithome.com.tw/upload/images/20220626/20124767jfp4UurSFk.png" alt="drawing" width="860"/>

### Collision and Load Factor
- Collision: When two or more objects happen to be hashed into the same index in hashTable.
- Load factor: the ratio of n / m
    - Usually, the load factor will be a number between 0 and 1
    - The smaller the load factor is, the hashTable is likely to have empty spots, but not too many collisions.
    - The bigger the load factor is, the hashTable is likely to be full and have lots of collisions.

## Hash Function II - Multiplication Method
<img src="https://ithelp.ithome.com.tw/upload/images/20220626/201247678HcPtIpSoN.png" alt="drawing" width="860"/>

---

# Handing Collisions
- No matter what hash function method we use, we will have collision.
- When coming across collision, we just store elements into an array. Therefore, our hashTable is actually a `array of array`

---

# Coding HashTable

## Based Class

```javascript
class HashTable {
  // m = hashTable size
  constructor(size) {
    this.size = size;
    this.table = [];
    for (let i = 0; i < this.size; i++) {
      this.table.push([]);
    }
  }

  // division method
  hashFunction1(key) {
    return key % this.size;
  }

  // multiplication method
  hashFucntion2(key) {
    const A = (Math.sqrt(5) - 1) / 2;
    return Math.floor(this.size * ((key * A) % 1));
  }
}
```

## Set and Get
```javascript
class HashTable {
  set(key, value) {
    let index = this.hashFucntion2(key);
    this.table[index].push({ key, value });
  }

  get(key) {
    let index = this.hashFucntion2(key);
    for (let i = 0; i < this.table[index].length; i++) {
      if (this.table[index][i] === key) {
        return this.table[index][i];
      }
    }
  }
}
```

---

# Understanding HashTable
- Assuming the following things are true (which might not be true in real life but could be close enough):
    1. Hash function has O(1) when hashing any keys.
    2. We are doing simple uniform hashing, which means that each key that we are hashing is equally likely to be hashed into any slot of hashTable, independent of other keys hashing.
- Then.
    1. Let’s called the load factor n/m = α, If m = θ(n), then α = O(1).
    2. The running time for hashTable is O(1 + α) .

---

# Reference
- [資料結構與演算法 (JavaScript)](https://www.udemy.com/course/algorithm-data-structure/)
