---
layout: post
title: DataStructure - Binary Tree
date: 2017-05-06 16:00:00 -0800
categories: Study-Notes
tag: dataStructure
---

* content
{:toc}




## Binary Tree

__Review:__
1. The __depth__ of a node is the number of edges from the node to the tree's root node.
2. The __height__ of a node is the number of edges from the node to the farthest leave.

For a perfect binary tree with L levels:  

1. Number of nodes in total: $$ 2^{L} - 1 $$  
2. Depth: $$ 0 \leq d < L $$  
3. Number of nodes in at each depth: $$ 2^{d} $$  
4. Sum of depth of each node: $$ \sum_{0 \leq d < L} d \cdot 2^{d} $$  
5. Height: $$ 0 \leq h < L $$  
6. Sum of height of each node: $$ \sum_{0 \leq d < L} h \cdot 2^{L-1-h} $$  
