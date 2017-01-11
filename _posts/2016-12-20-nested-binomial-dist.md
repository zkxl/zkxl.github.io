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
$$ {\binom {n}{k}} = {\frac {n(n-1)\dotsb (n-k+1)}{k(k-1)\dotsb 1}} $$

Q1: We launched event A for 1000 times. What's the probability of exactly one of them succeeded?

A1: $$ {\binom {1000}{1}} p^{1} (1 - p)^{999} $$

---

Q2: We launched event A for 1000 times. WHat's the probability of at least one of them succeeded?

A2: $$ \sigma_{k=1}^{n} {\binom {n}{k}} p^{k} (1 - p)^{n-k} $$
