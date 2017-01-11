---
layout: post
title: nested binomial
date: 2016-12-20 11:42:00 -0800
categories: reference
tag: nested-binomial
---

* content
{:toc}



__We now have event A. Its success probability is p, failure probability is (1 - p).__

_Note for the following problem:_  
$$ {\binom {k}{n}} = {\frac {n(n-1)\dotsb (n-k+1)}{k(k-1)\dotsb 1}} $$  

$$ {\binom {n}{n}} = {\binom {0}{n}} = 1 $$

---

Q1: We launched event A for 1000 times. What's the probability of exactly one of them succeeded?

A1: $$ {\binom {1}{1000}} p^{1} (1 - p)^{999} $$

---

Q2: We launched event A for 1000 times. What's the probability of at least one of them succeeded?

A2: $$ \sum_{k=1}^{1000} {\binom {1000}{k}} p^{k} (1 - p)^{1000-k} $$

* Note:  
$$ \sum_{k=1}^{1000} {\binom {10000}{k}} p^{k} (1 - p)^{1000-k}  + {\binom {0}{1000}} p^{0} (1-p)^{1000} = 1 $$

---

Q3: We launched event A for N times. What's the probability of at least two of them succeeded?

A3: $$ \hat p = 1 - \sum_{k=0}^1 {\binom {k}{N}} p^{k} (1-p)^{N-k} $$

---
