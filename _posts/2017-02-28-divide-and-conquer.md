---
layout: post
title: ICS260 - Divide and Conquer
date: 2017-02-28 13:20:00 -0800
categories: Study-Notes
tag: algorithms
---

* content
{:toc}



## Divide and Conquer Theorem

In the example of __Merge Sort__:  
During divide-phase, the algorithm splits the problem into half and recursively call itself on the two subproblems.   
During conquer-phase, the algorithm does some operations that take O(n) to merge the sorted sub results into real higher-level results.  

Mathematically:  
$$ T(n) = 2 T(\frac{n}{2}) + O(n) $$
$$ T(n) = n \log{n} $$


Now let constants:  
1. a >= 1 denoting the number of recursive calls we need at each recursion
2. b >= 1 denoting the number of subproblems
3. f(n) denoting the asymptotical cost we need to pay at each recursion.  

The above became:
$$ T(n) = a T(\frac{n}{b}) + f(n) $$

The master theorem says:  

1. Operation cost is __upper bounded__:
$$ if\;f(n) = O(n^{\log_{b}^{a - \epsilon}})\;\;for\;some\;constant\;\epsilon > 0 $$
$$ T(n) = \Theta(n^{\log_{b}^{a}}) $$

2. Operation cost is __sandwiched___:
$$ if\;f(n) = \Theta(n^{\log_{b}^{a}}) $$
$$ T(n) = \Theta(n^{\log_{b}^{a}} \log{n})$$

3. Operation cost is __lower bounded__:
$$ if\;f(n) = \Omega(n^{\log_{b}^{a + \epsilon}})\;\;for\;some\;constant\;\epsilon > 0 $$
$$ T(n) = \Theta(f(n)) $$
