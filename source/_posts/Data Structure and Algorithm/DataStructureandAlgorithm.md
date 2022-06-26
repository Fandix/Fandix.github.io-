---
title: Introduction to Algorithm Design
date: 2022-04-06 10:35:28
tags:
- CS

categories:
- Data Structure and Algorithm
---

本章中將會介紹一些簡單的 Search 方法，比如 `Linear Search`, `Binary Search`，也會介紹一些在寫 Algorithm 時常會用到的一些技巧。
<img src="https://ithelp.ithome.com.tw/upload/images/20220608/20124767TrjI5W40xE.png" alt="drawing" width="860"/>

<!-- more -->

---

# Linear Search (Sequential Search)
- 是一種會按造順序檢查列表中每一個元素直到找到匹配項目或搜索了整個列表的演算法
- 可能是最簡單的演算法
- 在實踐中將嘗試在 array 中找到一個數字，如果有找到則返回他的 index，反之回傳 -1

## Pseudocode for Linear Search
```javascript
LINEAR-SEARCH(array, n);
	for i from 0 to array.length - 1:
		if (array[i] === n):
				return i
return -1
```

```javascript
function linearSearch(arr, n) {
    for (let i = 0; i < arr.length; i++) {
        if (arr[i] === n)
            return i;
    }
    return -1;
}
```

## Overview of Linear Search
- Worst Cast Performance: O( n ) → 要找的位於arr的最後一項
- Best Case Performance: O( 1 )  → 要找的位於arr第一項
- Average performance: O( n/2 )

---

# Binary Search
- 是一種搜索演算法，他再以排序過後的arr中找到目標值的位置
- 比 linear search 更有效率，但僅適用於以排序過數據集
- 在實踐中我們將有一個排序的 array 以及一個我們洗要在這個排序數組中找到的數字，如果找到了則回傳 index，反之回傳-1

## Pseudocode for Binary Search
```javascript
BINARY-SEARCH(array, num)
	min = 0
	max = array.length - 1
	while min <= max:
		middle = (min + max ) / 2
		if num > array[middle]:
			num = middle + 1
		else if num < array[middle]:
			max = middle - 1
		else
			return middle
return -1
```
```javascript
function binarySearch(arr, num) {
    let min = 0;
    let max = arr.length - 1;
    while (min <= max) {
        const middle = (min + max) / 2;
        if (num > arr[middle]) {
            min = middle + 1;
        } else if (num < arr[middle]) {
            max = middle - 1;
        } else {
            return middle
        }
    }
    return -1
}
```

## Overview of Binary Search
- Worst Cast Performance: O( logn )
- Best Case Performance: O( 1 )  → 要找的正好在arr正中間
- Average performance O(logn)

---

# Intersection Problem
Write a function that take two arrays as parameters, and then return an array that is the intersection of these two arrays. For example, Intersection([1, 2, 3], [5, 16, 1, 3]) should return [1, 3].

```javascript
function intersection(arr1, arr2) {
    const arr = [];
    for (let i = 0; i < arr1.length; i++) {
        for (let j = 0; j < arr2.length; j++) {
            if (arr2[j] === arr1[i]){
                arr.push(arr1[i])
            }
        }
    }
    return arr;
}
```
由於有兩層 for...loop 所以為 `O( n^2 )`
<img src="https://ithelp.ithome.com.tw/upload/images/20220608/20124767QzUDcBbgWn.jpg" alt="drawing" width="800"/>

---

# Counter
- 是一個在設計演算法常用的技巧，counter 並不是一個正式的名字，雖然不同地方有不同名字，但功能是一樣的
- 使用 counter 可以有效的降低演算法的複雜度

使用 Intersection Problem 來做範例，Intersection([1, 2, 3], [5, 16, 1, 3])，可以計算在**陣列中的數字總共出現幾次**，若出現次數 `大於 1` 代表有交集。

```javascript
function Counter(arr1, arr2) {
    let result = [];
    const arr = arr1.concat(arr2);
    let counter = {};
    
    for (let i = 0; i < arr.length; i++) {
        if (!counter[arr[i]]) {
            counter[arr[i]] = 1;
        } else {
            counter[arr[i]]++;
        }
        
    }
    
    for (const property in counter) {
        if (counter[property] > 1) {
            result.push(property)
        }
    }
	  return result;
}
```
<img src="https://ithelp.ithome.com.tw/upload/images/20220609/20124767t3A34ZElgE.jpg" alt="drawing" width="800"/>

## Coding Practice - Frequency Problem
Write a function that takes two strings and check if they have the same letters.
Order doesn't matter.

Ex.
sameFrequency('abbc', 'aabc') false
sameFrequency('abba', 'abab') true
sameFrequency('aasdebasdf', 'adfeebed') false

```javascript
function sameFrequency(str1, str2) {
  const str1Obj = {};
  const str2Obj = {};

  str1.split('').forEach((s, i) => {
    if (!str1Obj[s]) {
      str1Obj[s] = 1;
    } else {
      str1Obj[s]++;
    }
  })

  str2.split('').forEach((s, i) => {
    if (!str2Obj[s]) {
      str2Obj[s] = 1;
    } else {
      str2Obj[s]++;
    }
  })

  if (Object.keys(str1Obj).length !== Object.keys(str2Obj).length) {
    return false;
  }

  for (const key in str1Obj) {
    if (str1Obj[key] !== str2Obj[key]) {
      return false;
    }
  }
  return true;
}
```

---

# Pointer
- 是一個設計演算法的技巧，也一樣不是正規的名字，雖然名字不一樣但是思考是相同的
- 目的是為了降低演算法的複雜度


input = [ 11, 0, 1, 2, 3, 9, 14, 17, 21 ];
- 先拿第一個與最後一個做平均，若結果不為 average 的話則將 right point 往左邊移動一格，再繼續和 left point 做平均檢查是否為 average
- 如果平均是 average 的話，則將 left 指標往右移動一格以找尋下一個符合的數

```javascript
function averagePair(arr, average) {
  let left = 0, right = arr.length - 1;
  const res = [];
  while(left <= right) {
    if ((arr[left] + arr[right]) / 2 !== average) {
      right--;
    } else {
      res.push([arr[left], arr[right]]);
      left++;
	  right--;
    }
  }
  return res;
}

console.log(averagePair([-11, 0, 1, 2, 3, 9, 14, 17, 21], 1.5));  // O(n)
```
<img src="https://ithelp.ithome.com.tw/upload/images/20220609/20124767DsAiriNGSc.jpg" alt="drawing" width="900"/>

## Coding Practice - Palindrome
Write a function that checks if the input string is a palindrome. Palindrome is a word that can read forwards and backwards.

Ex.
isPalindrome('awesome') false
isPalindrome('foobar') false
isPalindrome('tacocat') true
isPalindrome('amanaplanacanalpanama') true

```javascript
function isPilindrome(str) {
  const strArr = str.split('');
  let left = 0, right = strArr.length - 1;
  while(left !== right && left < right) {
    if (strArr[left] !== strArr[right]) {
      return false;
    }
    left++;
    right--;
  }
  return true;
}

console.log(isPilindrome('awesome'));
console.log(isPilindrome('foobar'));
console.log(isPilindrome('tacocat'));
console.log(isPilindrome('amanaplanacanalpanama'))
```

## Coding Practice - Subsequence Problem
A subsequence of a string is a new string that is formed from the original string by deleting some (can be none) of the characters without disturbing the relative positions of the remaining characters.
Write a function that checks if is strings is a subsequence of the  other string.

### 想法
- 拿兩個字串做比較，先拿第一個字串的第一個字與第二個字串的第一個字比較
    - 如果相同就兩個的point都往前一格
    - 如果不同就第二個字串的 point 持續往前直到找到為止
    - 當第二個字串中的字沒辦法找到與第一個字串一樣的字時變回傳 false

```javascript
function isSubsequence(str1, str2) {
  const str1Arr = str1.split('');
  const str2Arr = str2.split('');
  let str1Point = 0, str2Point = 0;

  let res = false;
  while (str1Point <= str1.length - 1 && str2Point <= str2.length - 1) {
    if (str1Arr[str1Point] !== str2Arr[str2Point]) {
      str2Point++;
    } else {
      str1Point++;
    }

    if (str1Point > str1Arr.length - 1) {
      res = true;
    }
  }
  return res;
}

console.log(isSubsequence('hello', 'heerllsao Dear'));
console.log(isSubsequence('book', 'brooklyn'));
console.log(isSubsequence('abc', 'bac'));
```

<img src="https://ithelp.ithome.com.tw/upload/images/20220609/201247672itHoGyqnr.jpg" alt="drawing" width="900"/>

---

# Sliding Window
- Generally speaking, a sliding window is a sub-list that runs over an underlying collection. For example, if you have an array like:
    [ a, b, c, b, e, f, g, h]
    Then, a sliding window of size 3 would run over it like
    [a, b, c]
    ---[b, c, d]
    ------[c, d, e]
    ---------[e, f, g]
    ...
    
## Coding practice - Max and Min Sum
當 input = [2,7,3,0,6,1,-5,-12,-11] 時，
Sliding window:

[2,7,3,]
--[7,3,0]
----[3,0,6]

從上面的規律可以看到，每一次的三個數相加都是將原本的數加上下一個 index 的數字後再減去第一個數字

```javascript
function maxSum(arr, target) {
  let max = 0, left = 0, right = target;
  for (let i = 0; i < target; i++) {
    max += arr[i];
  }

  while (right <= arr.length) {
    const tmp = (max - arr[left]) + arr[right];
    max = tmp > max ? tmp : max;
    left++;
    right++;
  }
  return max;
}

maxSum([2, 7, 3, 25, 0, 6, 1, -5, -12, -11], 3);
```

## Coding Practice - Min Subarray Length
Write a function called minSubLength which accepts two parameters, an array of positive integers and positive integer. This function should return the minimal length of a continue subarray - the sum of element inside this subarray has to be greater than or equal to the positive integer parameter. If subarray not found, then return 0.

Ex.
minSubLength([8,1,6,15,2,16,5,7,14,30,12], 60); //2

```javascript
function minSubLength(arr, target) {
  let left = 0, right = 1;
  let val = arr[0];
  let length = 999;
  while (right <= arr.length) {
    val += arr[right];

    if (val > target) {
      length = (right - left) < length ? right - left : length;
      left++;
      right = left + 1;
      val = arr[left];
    } else {
      right++;
    }
  }

  return length + 1;
}

console.log(minSubLength([8, 1, 6, 15, 3, 16, 5, 7, 14, 30, 12], 60));
```

## Coding Practice - Unique Letters String
Write a function called uniqueLetterString, which accepts a string and returns the length of the longest substring with all distinct characters.

Ex.
uniqueLetterString('thisishowwedoit') // 6

```javascript
function uniqueLetterString(str) {
  let left = 0, right = 0;
  let count = {};
  let max = -Infinity;
  while (right < str.length) {
    if (!count[str[right]]) {
      count[str[right]] = 1;
      right++;
    } else {
      max = (right - left) > max ? right - left : max;
      count = {};
      left++;
      right = left;
    }
  }
  return max = (right - left) > max ? right - left : max;
}

console.log(uniqueLetterString('thisishowwedoit'));
```

---

# Recursion
- A function that calls itself.
- Recursion is using a data structure called 'stack'. When we are calling a function inside another function, we are on the call stack.
- recursion is a also a mathematical relation in sequences
    - T(1) = 10
    - T(n) = T(n-1) +15
    - T(2) = T(2-1) + 15 = 10 + 15 = 25
    - T(3)=T(3-1)+15=T(2)+15=10 + 15 + 15 = 40

## Pseudocode of Recursive Sequence
```javascript
RECURSION-SEQUENCE(n):
	if n equals to 1:
		return 10;
	else:
		return RECURSION-SEQUENCE(n-1) + 15
```
```javascript
function rs(n) {
	if (n === 1) {
		return 10;
	} else {
		return rs(n-1) + 15;
	}
}
```

<img src="https://ithelp.ithome.com.tw/upload/images/20220609/20124767QYZ056daDr.jpg" alt="drawing" width="860"/>

## Coding Practice - Fibonacci Sequence
Write a function that takes an integer N as an input and returns the Nth number in Fibonacci Sequence.
Fibonacci Sequence is defined by:

F(0) = 0
F(1) = 1
F(n) = F(n-1) + F(n - 2)
```javascript
function fn(n) {
  if (n === 0) {
    return 0;
  } else if (n === 1) {
    return 1;
  } else {
    return fn(n - 1) + fn(n - 2);
  }
}
```
<img src="https://ithelp.ithome.com.tw/upload/images/20220609/20124767HwOATilJpA.jpg" alt="drawing" width="860"/>

## Coding Practice - Array of Arrays
Write a function that collects all value in an array of arrays and return an array or values collected.

Ex.
```javascript
let arrs = [[[['a', [['b', ['c']]], [['e']], [[['f', 'g', 'h']]]]]]];
collector(arrs); // [a, b, c, e, f, g, h]
```
```javascript
function fn(arr) {
  let res = [];
  for (let i = 0; i < arr.length; i++) {
    if (Array.isArray(arr[i])) {
      res = res.concat(fn(arr[i]));
    } else {
      res.push(arr[i]);
    }
  }
  return res;
}
```
<img src="https://ithelp.ithome.com.tw/upload/images/20220609/20124767GMRncvHDk0.jpg" alt="drawing" width="860"/>

---

# Reference
- [資料結構與演算法 (JavaScript)](https://www.udemy.com/course/algorithm-data-structure/)