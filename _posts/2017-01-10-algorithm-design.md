---
layout: post
title: Algorithm Design
date: 2017-01-10 19:20:00 -0800
categories: Study-Notes
tag: algorithms
---

* content
{:toc}


## Stable Marriage

#### Problem: How to compute a stable perfect matching?

Key Definitions:
* matching?  
_Given M = {m1, m2, ... mn} and W = {w1, w2, ..., wn}, a matching is a subset of M x W (cartesian product), S, such that each element of M and each element of W appears in_ __AT MOST__ _one pair in S._

* perfect match?  
_In a perfect matching S, each element of M and each element of W appear in_ __EXACTLY ONE__ _pair in S._

* instability? stable perfect match?  
_In a matching S, instability means there exists a pair (m, w) that's not in S but each of m and w prefers the other to their current partners in S_

---

#### G-S Algorithm

```
While there’s a free man // indicating he has not proposed to every woman
  m = such man // non-deterministic
  w = the highest ranking women in m’s preference list to whom m has not yet proposed
  if w is free
    (m, w) becomes engaged
  else  // w is currently engaged to some other man m'
    if w prefers m to m'
      (m, w) becomes engaged
      m' becomes free
    else
      (m', w) remain unchanged
      m remain free
    endif
  endif
endwhile
```

#### Claims:  

1. The sequence of partners to whom a man proposes gets worse; the sequence of partners to whom a woman is engaged gets better.

2. The algorithm terminates after at most n^2 proposals.

3. If a man m is free, there is some woman to whom he has not yet proposed.  
> _proof by contradiction:_  
>   _suppose that m is free and he has proposed to all women._  
>   _then all women are engaged to some men._  
>   _since |M| == |W|, these means all men are engaged._  

4. They matching computed by the G-S algorithm is a perfect matching.  
  > _proof by contradiction:_  
  >   _suppose that the algorithm terminates without a perfect matching, meaning there is a free man m._  
  >   _according to the code, m must have proposed to every woman._  
  >   _according to the claim 3, it is impossible._  

5. The matching computed by the G-S algorithm is a stable matching.  
  > _proof by contradiction:_  
  >   _suppose that there is an instability, meaning that the matching contains two pairs (m, w) and (m', w') such that m prefers w' to w and w' prefers m to m'._  
  >   _the last time m proposed, it was to w._  
  >   _if m proposed to w' some earlier time, w' rejected m and she prefers m' to m, contradiction._  
  >   _if m proposed to w' some later time, m doesn't prefer w' to w._  

#### Determinacy Analysis in G-S Algorithm

Q: The non-deterministic G-S algorithm however produces deterministic result, given any particular input. Why?

A: The G-S algorithm always pairs each man with his best valid partner.  
> _proof by contradiction:_  
>   _suppose at least one proposal to a valid partner is rejected(either immediately or as a result of a broken engagement.), consider that the_ __first rejection__ _: m proposes to a valid partner w and w immediately rejects m for m'._  
>   _there exists some woman w' that leaves (m, w) and (m', w') in a stable matching X._  
>   __However__, _X cannot be stable because of the following claims:_  
>   _w prefers m' to m. [w rejected m for m']_  
>   _m' prefers w to w'. [If he preferred w' to w, he would have proposed to w' first in G-S algorithm and end up being rejected before he proposes to w. (Not until then, can w be engaged with m' and w later rejects m, which contradicts our_ __"first rejection"__ _)]_  

#### Complexity and Implementation

Input: two 2D matrices (man, preference_list) and (woman, preference_list)  

Output: a stable perfect matching S.  

The algorithm terminates at most n^2 proposals, looping over n^2 times. If every operation in this loop is O(1), the overall complexity will be O(n^2).  

Specifically:  

1. find a free man - keep only free man on a linked list so it takes O(1) to access the next free man;  

* It takes O(n) to convert an array into a linkedlist;  

2. for a given man m, find his next highest ranking woman w to whom m hasn't proposed - maintain a 1D array h, where h[m] == w;  

3. For a given woman w, find out if she's currently engaged or not - maintain a 1D array c, where c[w] == null if w is not engaged and c[w] == m' if w is engaged with m'.  

4. For a given woman w engaged with m', to whom m proposed, determine whether w prefers m to m' - this is tricky to do in O(1) because normally you'd need to reference to w's preference_list and find how m ranks to m'. __Solution: from woman's preference array, you need to create a "ranking" array r where r[w, m] = m's ranking in w's preference_list. This can be done in O(n^2).__  

---

## Binary Heap Implementation of Priority Queue
