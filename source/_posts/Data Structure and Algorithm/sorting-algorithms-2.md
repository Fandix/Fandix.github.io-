---
title: Sorting Algorithms II
date: 2022-04-08 17:13:45
tags:
  - Data Structure and Algorithm

categories:
  - CS
---

本章將會介紹 `Sorting Algorithms` 中效率比較比較好 `O(nlogn)` 但比較複雜的三種 `sorting method`，分別是 `Merge Sort`, `Heap Sort` 與 `Quick Sort`。

<img src="https://res.cloudinary.com/codecrucks/images/f_webp,q_auto/v1631197926/sorting-algorithms/sorting-algorithms.jpg?_i=AA" alt="drawing" width="900"/>

<!-- more -->

# Merge Sort

- The Principle of merge sort is quite simple. Take advantage of the fact that combining two sorted array has O(n) time complexity, using the pointer skill.
- The sorting algorithm is a classic example of “divide and conquer” (將問題分割為小部分進行處理).

## Diagram

<img src="https://ithelp.ithome.com.tw/upload/images/20220622/201247672Hh5FuVNf5.png" alt="drawing" width="860"/>

## Pseudocode I of Merge Sort

```javascript
MARGE(A1, A2):
	result = [], i = 0, j = 0
	while i < A1.length and j < A2.length:
		if A1[i] > A2[i]:
			add A2[j] to result
			j++
		else:
			add A1[i] ro result
			i++

// either arr1 or arr2 wll have something left
// use loop to put all remaining things into the result
```

```javascript
function merge(arr1, arr2) {
  const result = [];
  let i = 0,
    j = 0;
  while (i < arr1.length && j < arr2.length) {
    if (arr1[i] < arr2[j]) {
      result.push(arr1[i]);
      i++;
    } else {
      result.push(arr2[j]);
      j++;
    }
  }
  while (i < arr1.length) {
    result.push(arr1[i]);
    i++;
  }
  while (j < arr2.length) {
    result.push(arr2[j]);
    j++;
  }
  return result;
}
```

## ## Pseudocode II of Merge Sort

```javascript
MERGE-SORT(A):
	if A.length equals to 1:
		return A
	else
		middle = A.length / 2
		left = A.slice(0, middle)
		right = A.slice(middle, A.length)
		return MERGE(MERGE-SORT(right), MERGE-SORT(right))
```

```javascript
function mergeSort(a) {
  if (a.length === 1) {
    return a;
  }

  const middle = Math.floor(a.length / 2);
  const left = a.slice(0, middle);
  const right = a.slice(middle, a.length);
  return merge(mergeSort(left), mergeSort(right));
}
```

## BigO of Merge Sort

- Worse Case Performance: O(nlogn)
- Best Case Performance: O(nlogn)
- Average Performance: O(nlogn)

---

# Tree (Data Structure)

- In computer science, a tree is a widely used abstract data type that simulates a hierarchical tree structure, with a root value and subtrees of children with a parent node, represented as a set of linked nodes.
  - A tree should have only one root.
  - Another definition of tree is 'tree is acyclic graph'

## Binary Tree

- **Binary Tree** means each node has at most two children, which are referred th as the left child and the right
- **Complete Binary** Tree mean an 'almost-full' binary tree, the right most nodes of the bottom might be missing.
  <img src="https://static.javatpoint.com/ds/images/full-binary-tree-vs-complete-binary-tree3.png" alt="drawing" width="500"/>
- **Full Binary** Tree means a binary tree in which all leaf nodes have the same depth
- **Max Heap** means a complete binary tree where the `largest node` is always the at `root` for any sub-tree
  <img src="https://www.tutorialspoint.com/data_structures_algorithms/images/max_heap_example.jpg" alt="drawing" width="500"/>

---

# Heap Sort Algorithm

- Heap sort uses **Max heap** to sort.
- In order to understand our next sorting algorithm, Heap Sort, you must first understand what a 'max heap' is. Also, we need to know how max heap algorithm works.
  <img src="https://ithelp.ithome.com.tw/upload/images/20220622/20124767lwObu6iVNB.png" alt="drawing" width="500"/>
  <img src="https://ithelp.ithome.com.tw/upload/images/20220622/20124767aK7p49oywC.png" alt="drawing" width="900"/>

## How to create max heap?

When we swap a node N down, just keep swapping if the node N has a child node that is bigger than the node N

```javascript
arr = [6,13,10,4,1,5,2,8,14,9,11,7,3,15,12];
Max Heap = [15,14,12,13,11,7,10,8,4,9,1,5,3,2,6];
```

<img src="https://ithelp.ithome.com.tw/upload/images/20220622/20124767k6VHlXv68N.png" alt="drawing" width="900"/>

## Pseudocode I of Build Max Heap

```javascript
BUILD-MAX-HEAP():
	heapSize = A.length - 1
	for i from (A.length / 2) to 0(inclusive):
	  MAX-HEAPIFY(i)
```

```javascript
function buildMaxHeap() {
  heapSize = arr.length - 1;
  for (let i = Math.floor(heapSize / 2); i >= 0; i--) {
    maxHeapify(i);
  }
}
```

## Pseudocode II of Build Max Heap

```javascript
MAX-HEAPIFY(i):
	// arr =   [6,13,10,4,1,5,2,8,14,9,11, 7, 3,15,12];
	// index = [0, 1, 2,3,4,5,6,7, 8,9,10,11,12,13,14];
	// if index = 6(value = 2); -> l = 13(value = 15), r = 14(value = 12)
	l = 2 * i + 1
	r = 2 * i + 2
	if l < heapSize and A[l] > A[i]:
			largest = l
	else:
			largest = i
	if r <= heapSize and A[r] > A[i]:
			largest = r
	if largest is not i:
			swap A[i] with A[largest]
			MAX_HEAPIFY(largest)
```

```javascript
function maxHeapify(index) {
  let largest = index;
  let left = 2 * index + 1;
  let right = 2 * index + 2;

  if (left <= heapSize && arr[left] > arr[index]) largest = left;
  if (right <= heapSize && arr[right] > arr[largest]) largest = right;
  if (largest !== index) {
    let tmp = arr[index];
    arr[index] = arr[largest];
    arr[largest] = tmp;
    maxHeapify(largest);
  }
}
```

## Pseudocode III of Build Max Heap

```javascript
HEAP-SORT():
	.BUILD-MAX-HEAP()
	for i from A.length - 1 to 0:
		exchange A[0] with A[i]
		heapSize--
		MAX-HEAPIFY(0)
```

```javascript
function heapSort() {
  buildMaxHeap();
  for (let i = arr.length - 1; i >= 0; i--) {
    let tmp = arr[i];
    arr[i] = arr[0];
    arr[0] = tmp;
    heapSize--;
    maxHeapify(0);
  }
}
```

## BigO of Heap Sort

- Worse Case Performance: O(nlogn)
- Best Case Performance: O(nlogn)
- Average Performance: O(nlogn)

<img src="https://ithelp.ithome.com.tw/upload/images/20220622/20124767g5BkDsKYe7.png" alt="drawing" width="900"/>

---

# Partition

- By itself, Partition is not a sorting algorithm, but it is an important subroutine of the QuickSort algorithm. So, to understand QuickSort, you must first understand Partition.
- The idea of partition algorithm is to divide the array into 2 parts. Either part is a sorted array, but the element in the middle is sorted.

<img src="https://ithelp.ithome.com.tw/upload/images/20220622/20124767LctBg2KRtg.png" alt="drawing" width="900"/>

# Quick Sort

- As you know from its name, quick sort is quick
- Developed by British computer scientist Tony Hoare in 1959 and published in 1961, it is still a **commonly used** algorithm for sorting.

## Pseudocode of Quick Sort
<img src="https://ithelp.ithome.com.tw/upload/images/20220622/20124767gHSDNDNxuz.png" alt="drawing" width="900"/>

```javascript
function quickSort(p, r) {
  if (p < r) {
    const q = partitilon(p, r);
    quickSort(p, q - 1);
    quickSort(q + 1, r);
  }
}
```

## Pseudocode of Partition
<img src="https://ithelp.ithome.com.tw/upload/images/20220622/20124767bMbTAyo0L9.png" alt="drawing" width="900"/>

```javascript
function partitilon(p, r) {
  const prvot = arr[r];
  let i = p - 1;

  for (let j = p; j < r; j++) {
    if (arr[j] <= prvot) {
      i++;

      const tmp = arr[i];
      arr[i] = arr[j];
      arr[j] = tmp;
    }
  }

  const tmp = arr[i + 1];
  arr[i + 1] = arr[r];
  arr[r] = tmp;

  return i + 1;
}
```

## Overview of Quick Sort
- Worse Case Performance: O(n^2)
- Best Case Performance: O(nlogn)
- Average Performance: O(nlogn)

---

# Reference
- [資料結構與演算法 (JavaScript)](https://www.udemy.com/course/algorithm-data-structure/)