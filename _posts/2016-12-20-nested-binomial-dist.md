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

_Notes for the following problem:_  

$$ {\binom {k}{n}} = {\frac {n(n-1)\dotsb (n-k+1)}{k(k-1)\dotsb 1}} $$  


$$ {\binom {n}{n}} = {\binom {0}{n}} = 1 $$

---

Q1: We launched event A for 1000 times. What's the probability of exactly one of them succeeded?  

A1:  

$$ {\binom {1}{1000}} p^{1} (1 - p)^{999} $$

---

Q2: We launched event A for 1000 times. What's the probability of at least one of them succeeded?  

A2:  

$$ \sum_{k=1}^{1000} {\binom {k}{1000}} p^{k} (1 - p)^{1000-k} $$

equivalently:  

$$ 1 - {\binom {0}{1000}} p^{0} (1-p)^{1000} $$

---

Q3: We launched event A for N times. What's the probability of at least two of them succeeded?  

A3:  

$$ \hat p = 1 - \sum_{k=0}^1 {\binom {k}{N}} p^{k} (1-p)^{N-k} $$

---

__Now we have another event B. It has the same success/failure probability as A. We define event A is overall successful if at least two out of all the trials we launched succeeded.__


Q4: We launched event A and event B for N times respectively. What's the probability of at least one of them is overall successful?  

A4:  

$$ 1 - {\binom {0}{2}} {\hat p}^{0} (1 - {\hat p})^{2} $$

---

__Now we have m such type of events, A, B, C, ......__

Q5: We launched all of such events, each of them for N times. What is the probability of at least three (m >= 3) of such events are overall successful?  

A5:  

$$ 1 - \sum_{j=0}^{2} {\binom {j}{m}} {\hat p}^{j} (1 - {\hat p})^{m-j} $$
