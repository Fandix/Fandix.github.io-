---
title: Sorting Algorithms I
date: 2022-04-07 10:34:59
tags:
  - Data Structure and Algorithm

categories:
  - CS
---

本章將會介紹 `Sorting Algorithms` 中效率比較差但比較簡單的三種 `sorting method`，分別是 `Bubble Sort`, `Insertion Sort` 與 `Seection Sort`。

<img src="https://embed-ssl.wistia.com/deliveries/70d6f4e10e2badb5ef394f00c17ad2bc1c14f6e7.jpg" alt="drawing" width="900"/>

<!-- more -->

# Intro to Sorting Algorithms

- Sorting is one of the most fundamental fo all algorithms in CS.
- Many programming languages have built-in sorting function, it's still a good thing to know how it works.
- Will learning 6 different sorting algorithms in this course, It's good to know to use which algorithm, as they excel in some certain cases.

## List of Algorithms

1. Bubble Sort
2. insertion Sort
3. Selection Sort

---

# Bubble Sort

- Bubble sort compares adjacent element and swaps them if they are in the wrong order.
- This simple algorithm not use in real world only an educational tool.

```javascript
BUBBLE-SORT(A):
	for i from 0 to A.length - 2(inclusive):
		for j from A.length - 1 to i + 1(inclusive):
			if A[j] < A[j - 1]:
				exchenge A[j] with A[j-1]
```

```javascript
function bubbleSort(arr) {
  for (let i = 0; i < arr.length - 2; i++) {
    let swapping = false;
    for (let j = arr.length - 1; j > i; j--) {
      let tmp;
      if (arr[j] < arr[j - 1]) {
        tmp = arr[j];
        arr[j] = arr[j - 1];
        arr[j - 1] = tmp;
        swapping = true;
      }
    }
    if (!swapping) break;
  }
  return arr;
}
```

<img src="https://ithelp.ithome.com.tw/upload/images/20220610/20124767yIPqBdTux5.png" alt="drawing" width="860"/>

## Worst Case Analysis
```
  [n, n-1, n-2, .... , 4, 3, 2, 1] 
  [1, n, n-1, n-2, .... , 4, 3, 2] -> n - 1
  [1, 2, n, n-1, n-2, .... , 4, 3] -> n - 2 
  [1, 2, 3, n, n-1, n-2, .... , 4] -> n - 3
  ...
```
- (n-1) + (n-1) + (n-1) + ... + 1 + 0;
- f(n) = ((n - 1) * n) / 2 = (1/2)n^2 - (1/2)n = O(n^2)

---

# Insertion Sort

- Insertion sort is a little bit more efficient than bubble sort in practice, have the same Big O Value.
- The principle of insertion sort in simple, Keeping inserting a new value into a sorted array. Insert it to the correct spot so the sorted array remains sorted.

## Pseudocode of Insertion Sort
```javascript
INSERTION-SORT(A):
	for j from index th A.length - 1(inclusive):
		key = A[j]
		i = j - 1;
		while i >= 0 and A[i] > key:
			A[i+1] = A[i]
			i = i - 1
		A[i + 1] = key
```

```javascript
function insertionSort(arr) {
    for (let j = 1; j < arr.length; j++) {
        let i = j - 1;
        let key = arr[j];
        while (i >= 0 && arr[i] > key) {
            arr[i + 1] = arr[i];
            i--;
        }
        arr[i + 1] = key;
    }
    return arr;
}
```

<img src="https://ithelp.ithome.com.tw/upload/images/20220610/20124767aH48dd3vE6.png" alt="drawing" width="860"/>

## Worst Case Analysis
```
  [n, n-1, n-2, .... , 4, 3, 2, 1] 
  [1, n, n-1, n-2, .... , 4, 3, 2] -> n - 1
  [1, 2, n, n-1, n-2, .... , 4, 3] -> n - 2 
  [1, 2, 3, n, n-1, n-2, .... , 4] -> n - 3
  ...
```
- f(n) = 1 + 2 + 3 + ... + n = ((n - 1) * n) / 2 = (1/2)n^2 - (1/2)n = O(n^2)

---

# Selection Sort
- The principle of selection sort is - select the smallest value in unsorted array, and then swap it with then left most value in this unsorted array.

## Pseudocode of Selection Sort
```javascript
SELECTION-SORT(A):
	for i from 0 to A.length - 2:
		minIndex = i
		for j from i to A.length - 1:
			if A[j] < A[minIndex]:
				minIndex = i
		swap A[minIndex] and A[i]
```

```javascript
function selectionSort(arr) {
    for (let i = 0; i < arr.length - 2; i++) {
        let minIndex = i;
        for (let j = i; j <= arr.length; j++) {
            if (arr[j] < arr[minIndex]) {
                minIndex = j;
            }
        }
        let tmp = arr[i];
        arr[i] = arr[minIndex];
        arr[minIndex] = tmp;
    }
    return arr;
}
```
<img src="https://ithelp.ithome.com.tw/upload/images/20220610/201247678jBvijoI4D.png" alt="drawing" width="860"/>

## Overview os Selection Sort
- Worst Case Performance: O(n^2)
- Best Cast Performance : O(n^2)
- Average Performance: O(n^2)

---

# Reference
- [資料結構與演算法 (JavaScript)](https://www.udemy.com/course/algorithm-data-structure/)