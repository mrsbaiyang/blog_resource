---
title: Treap  
date: 2018-08-21 18:47:11  
tags: 数据结构  
categories: 技术

---
### Treap是什么
Treap是Tree+Heap，也是一种二叉查找树。它相对于二叉树多了priority的概念。  
它具有二叉树和堆的特性：  

1. Treap是关于key的二叉排序树（也就是规定的排序方式）。
2. Treap是关于priority的堆（按照随机出的优先级作为小根/大根堆)。（非二叉堆，因为不是完全二叉树）
3. key和priority确定时，treap唯一。

Treap依赖于随机数，随机生成的优先级属性，通过简单的左右旋可以将长链旋转成近似完全二叉树结构。

### Treap是如何做到近似完全二叉树的呢？
拿插入来说，先按照key的大小添加到二叉树上，再按照priority旋转二叉树。因为priority是随机分配的，所以在一定的概率上不容易导致单链条的二叉树，这就是Treap最根本的特点。