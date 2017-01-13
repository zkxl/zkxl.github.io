---
layout: post
title: Deep Learning in Math
date: 2016-12-15 15:19:00 -0800
categories: deeplearning-basics
tag: undecided
---

* content
{:toc}



From "Multilayer feedforward networks are universal approximators", it is shown that a neural net with a single hidden layer already has __universal approximation property__, meaning: it can approximate any continuous function from a compact domain to the reals, to arbitrary precision.  

Mathematically, for any $$ \epsilon > 0 $$ there exists an $$ \alpha > 0 $$, so that:  

$$ \abs{x_1 - x_2} < \alpha => \abs{f(x_1) - f(x_2)} < \epsilon %%  
