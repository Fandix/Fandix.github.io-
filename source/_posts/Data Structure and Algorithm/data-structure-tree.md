---
title: Data Structure - Tree
date: 2022-04-27 20:55:37
tags:
- CS

categories:
- Data Structure and Algorithm
---

本章開始將會介紹一種非常重要的數據結構 `Tree`，正如同字面上的意思一樣他的數據結構會像樹枝一樣由一個`起始點 (Root)` 往下生長，除了 Root 之外每一個 Node 都會有各自的 `Parent Node` 與 `Child Node`。

<img src="https://www.researchgate.net/profile/Fraser-Morgan-3/publication/252774444/figure/fig1/AS:298040805085186@1448069873829/Basic-tree-data-structure-in-this-case-a-binary-tree-Node-1-is-the-root-node-with.png" alt="drawing" width="800"/>

<!-- more -->

# Graph (Definition)
- In computer science, a graph is an abstract data type.
- A graph data structure consists of a finite set of vertices (also called nodes or points).
- Lines between nodes are know as edges (also called links or lines), and for a directed graph are also known as arrows.

---

# Tree (Definition)
- Tree is a graph without a loop
- Tree must have one and only one root.

## Real Life Application of Tree
- DOM (Document Object Model)
- File System in Operation System
- Artificial Intelligence

---

# Tree Traversal
- Since tree is a commonly used data structure, we need a systematic way to know what nodes are in a tree.
- There are two way to do tree traversal:
    1. Breadth-First Tree Traversal
    2. Depth-First Tree Traversal
        1. Pre Order
        2. In Order
        3. Post Order

## Breadth-First Tree Traversal ( 寬度優先 )
<img src="https://cdn-images-1.medium.com/max/800/1*uJCkOS5_NZfHhc-fIF8T0g.gif" alt="drawing" width="800"/>
<img src="https://ithelp.ithome.com.tw/upload/images/20220626/20124767y1q2LJIDco.png" alt="drawing" width="900"/>


## Depth-First Tree Traversal
### PreOrder
Definition: Process all nodes of a tree by processing the root, then recursively processing all subtree. Also known as prefix traversal.

**Understanding**: `root`, `left`, `right`
<img src="https://ithelp.ithome.com.tw/upload/images/20220626/20124767aYuTYTNSCn.png" alt="drawing" width="900"/>

#### Pseudocode of PreOrder
```javascript
PREORDER(n):
	write(n)
		for i from 0 to n's children count - 1:
			PREORDER(n[i])
```

### InOrder
Definition: Process all nodes of a tree by recursively processing the left subtree. then processing the root, and finally the right subtree.

**Understanding**: `left`, `root`, `right`
**InOrder**: 2, 7, 10, 5, 6, 11, 2, 4, 9, 5

<img src="https://ithelp.ithome.com.tw/upload/images/20220626/20124767sueSD8omqN.png" alt="drawing" width="860"/>

#### Pseudocode of InOrder
```javascript
INORDER(n):
	INORDER(n[0])
	write(n)
	for i from 1 to n's children count-1
		INORDER(n[i])
```

### PostOrder
Definition: Process all nodes of a tree by recursively processing all subtree, then finally processing the root.

**Understanding**: `left` , `right`, `root`
**PostOrder**: 2, 10, 5, 11, 6, 7, 4, 9, 5, 2

<img src="https://ithelp.ithome.com.tw/upload/images/20220626/20124767Y98dFmffmu.png" alt="drawing" width="860"/>

#### Pseudocode of PostOrder
```javascript
POSTORDER(n):
	for i from 0 to n's children count - 1:
		POSTORDER(n[i])
	write(n)
```

--- 
# Reference
- [資料結構與演算法 (JavaScript)](https://www.udemy.com/course/algorithm-data-structure/)


