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

$$ |x_1 - x_2| < \alpha => |f(x_1) - f(x_2)| < \epsilon $$  

Here, with $$ \epsilon $$ being the arbitrary precision, for every small interval $$ [\alpha_{n-1}, \alpha_{n}) $$ in input space, you can allocate a neural with a threshold activation function that produce a value of $$ f(\alpha_{n}) - f(\alpha_{n-1}) $$ when activated.  
